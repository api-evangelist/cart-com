---
name: Create and fulfill an order on a Cart.com Online Store
description: Create an order for an existing customer, add line items and addresses, record a payment, then ship it — using the Cart.com Online Store API.
api: openapi/cart-com-online-store-openapi-original.yml
generated: '2026-07-31'
method: generated
operations:
  - get-customers
  - post-orders
  - post-order_items
  - post-order_addresses
  - post-order_payments
  - get-order_statuses
  - post-order_shipments
  - get-orders
---

# Create and fulfill an order

## Before you start

- **Base URL.** `https://{storeDomain}/api/v1` — the merchant's own storefront domain. There is no shared Cart.com API host.
- **Auth.** Send `X-AC-Auth-Token: <token>` on every request. Tokens come from the admin console (non-expiring) or the OAuth 2 flow at `/api/oauth`. See `authentication/cart-com-authentication.yml`.
- **Scopes.** This flow needs `orders` (write) and `read_people`. See `scopes/cart-com-scopes.yml`.
- **Content type.** `application/json`, `snake_case` properties, ISO 8601 datetimes.
- **Rate limit.** 5–50 calls per rolling 10-second window depending on the merchant's plan. Read `X-AC-Call-Limit` on every response; on `429`, wait for `Retry-After`. See `rate-limits/cart-com-rate-limits.yml`.
- **There is no idempotency key.** Writes are not retry-safe. Before retrying any `POST`, re-read with a filtered `GET` and confirm the record was not already created.

## Steps

1. **Find the customer.** `get-customers` — `GET /customers?email=<address>`. The API has no `GET /customers/{id}`; use the query syntax on the collection instead (`?id=<n>` works the same way). If nothing comes back, create one with `post-customers` first.
2. **Create the order.** `post-orders` — `POST /orders` with `customer_id` and `store_id`. Keep the returned `id`; every later step references it.
3. **Attach the addresses.** `post-order_addresses` — `POST /order_addresses` once for the billing address and once for the shipping address, each carrying the `order_id`.
4. **Add the line items.** `post-order_items` — `POST /order_items` per line, with `order_id`, `product_id` (or `item_number`), `quantity` and `price`. Send them one at a time; there is no bulk endpoint.
5. **Record the payment.** `post-order_payments` — `POST /order_payments` with `order_id`, amount and payment method. If the store routes authorization through a `AuthorizingPayment` webhook, that subscriber decides `approved`/`declined` — see `asyncapi/cart-com-online-store-webhooks.yml`.
6. **Look up the target status.** `get-order_statuses` — `GET /order_statuses` and pick the id for the status you intend to move the order to (statuses are merchant-defined, not fixed constants).
7. **Ship it.** `post-order_shipments` — `POST /order_shipments` with `order_id`, warehouse, carrier and tracking number, then `put-orders-id` — `PUT /orders/{id}` to move the order onto the shipped status. Setting a Shipped status fires the `OrderShipped` webhook.
8. **Verify.** `get-orders` — `GET /orders?id=<order_id>&expand=items,payments,shipments` to read back the assembled order in one call.

## Rules

- **Errors are not RFC 9457.** Every failure is `{"status_code": n, "message": "...", "details": "..."}`. Only `200` and `404` are declared in the contract; `401`, `429` and `500` are real but undeclared. See `errors/cart-com-problem-types.yml`.
- **A `500` on a write usually means a validation or referential-integrity failure**, not a transient outage — the `details` string names the failing column or stored procedure. Do not blind-retry it.
- **Paging.** `page` (default 1) and `count` (default 100); an out-of-range `page` returns `404`, not an empty list.
- **Expansion.** `expand=items,payments,shipments` populates nested collections; `fields=` restricts the payload and implicitly expands any nested resource it names.
- **Never send `decrypt`-scoped calls** (`/credit_cards/{id}/decrypted`, `/order_payments/{id}/decrypted`) unless the task explicitly requires cardholder data and the token was minted with the `decrypt` scope.
