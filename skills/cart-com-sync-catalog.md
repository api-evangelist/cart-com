---
name: Sync a product catalog into a Cart.com Online Store
description: Create and update products, variants, pricing, pictures and inventory on a Cart.com Online Store, and reconcile stock levels from an external system of record.
api: openapi/cart-com-online-store-openapi-original.yml
generated: '2026-07-31'
method: generated
operations:
  - get-categories
  - post-categories
  - get-manufacturers
  - get-product_statuses
  - get-products
  - post-products
  - put-products-id
  - post-product_variants
  - post-product_pricing
  - post-product_pictures
  - get-variant_inventory
  - put-variant_inventory-id
  - get-inventory
---

# Sync a product catalog

## Before you start

- **Base URL.** `https://{storeDomain}/api/v1`, `X-AC-Auth-Token` on every request.
- **Scopes.** `catalog` (write) — plus `settings` if you also touch warehouses or shipping. See `scopes/cart-com-scopes.yml`.
- **Rate limit.** 5–50 calls per 10-second window by plan. A full catalog sync will hit this; batch reads with `count=100` and back off on `429` using `Retry-After`.
- **No bulk write.** Every create and update is a single-resource call. Plan the sync around the call budget, not around the catalog size.

## Steps

1. **Resolve the taxonomy first.** `get-categories` — `GET /categories`; create anything missing with `post-categories`. Do the same for `get-manufacturers` and `get-product_statuses`; products reference all three by integer id.
2. **Detect existing products.** `get-products` — `GET /products?item_number=<sku>&fields=id,item_number,item_name`. The API has no `GET /products/{id}`; filter the collection with the query syntax instead.
3. **Create or update.** New SKU → `post-products` (`POST /products` with `item_name`, `item_number`, `product_status_id`, `primary_category_id`). Existing SKU → `put-products-id` (`PUT /products/{id}`).
4. **Add variants.** `post-product_variants` — `POST /product_variants` per option value, referencing `product_id`. Group them with `post-variant_groups` when the store uses variant groups.
5. **Set pricing.** `post-product_pricing` — `POST /product_pricing` for customer-type or quantity-tier pricing beyond the base `price`/`retail`/`cost` fields on the product itself.
6. **Attach images.** `post-product_pictures` — `POST /product_pictures` with `product_id`. Binary assets go through the documented `POST /api/v1/upload` endpoint (`system` scope); note that upload is documented but is **not** present in the published OpenAPI.
7. **Reconcile stock.** `get-variant_inventory` — `GET /variant_inventory?item_number=<sku>` then `put-variant_inventory-id` — `PUT /variant_inventory/{id}` with the new quantity. For a read-only, store-wide view use `get-inventory` — `GET /inventory`, the one special inventory endpoint.
8. **Verify.** `GET /products?id=<n>&expand=variants,pricing,pictures,categories` returns the whole assembled product in one call.

## Rules

- **Filtering.** Every field is a query parameter: `?price=gt:5.00`, `?item_name=like:widget`, `?item_name=like:doge+OR+like:wow`. Operators are `eq` (default), `not`, `like`, `startwith`, `gt`, `gte`, `lt`, `lte`. Parenthesised boolean grouping is not supported.
- **Paging.** `page` + `count`; read `total_count` and follow `next_page` from the list metadata rather than incrementing blindly — an out-of-range page is a `404`.
- **Caching.** `GET` responses are server-cached with `Last-Modified` / `Expires` / `Cache-Control`. Honour them; send `Cache-Control: no-cache` only when you truly need up-to-the-second data.
- **No idempotency contract.** If a `POST /products` times out, re-query by `item_number` before retrying or you will create a duplicate.
- **Catalog writes fire webhooks** — `ProductCreated`, `ProductUpdated`, `ProductStatusChanged`. If the store subscribes a synchronous `GetPrice` or `GetProductStatus` webhook, your writes are not the only source of truth for displayed price and availability. See `asyncapi/cart-com-online-store-webhooks.yml`.
