---
name: Subscribe to Cart.com Online Store webhooks
description: Register webhook subscriptions on a Cart.com Online Store, choose failure and caching behaviour, and handle the synchronous events that the storefront waits on.
api: openapi/cart-com-online-store-openapi-original.yml
generated: '2026-07-31'
method: generated
operations:
  - get-stores
  - get-order_statuses
  - get-orders
---

# Subscribe to Online Store webhooks

## Before you start

- **Base URL.** `https://{storeDomain}/api/v1`, `X-AC-Auth-Token` on every request.
- **Scopes.** Managing subscriptions needs both `orders` (view and change order data) and `system` (perform system tasks). See `scopes/cart-com-scopes.yml`.
- **Contract gap — read this first.** The webhook subscription resource `/api/v1/webhooks` is documented in the provider's Webhooks guide but is **not present in the published OpenAPI**, so there is no `operationId` for it and no machine-readable request schema. Treat the field list below as the contract; it is transcribed verbatim from the provider's documentation into `asyncapi/cart-com-online-store-webhooks.yml`.
- From platform version 2020.3 subscriptions can also be managed in the admin console at `/store/admin/settings/addons/webhooksubscriptionslist.aspx`.

## Steps

1. **Resolve the store.** `get-stores` — `GET /stores`. Every subscription is scoped to one `store_id`; only events on that store fire it.
2. **Pick the event type.** Choose from the 30 documented types in `asyncapi/cart-com-online-store-webhooks.yml`. Decide first whether you need an *observer* (asynchronous, no reply: `OrderPlaced`, `OrderShipped`, `CustomerCreated`, `ProductUpdated`, …) or a *participant* (synchronous, the store waits on your reply: `GetPrice`, `GetTax`, `GetDiscount`, `GetProductStatus`, `ValidateCart`, `ValidateCheckout`, `ValidateCustomer`, `ValidateProduct`, `AuthorizingPayment`, `CapturingPayment`).
3. **Create the subscription.** `POST /api/v1/webhooks` with:
   - `event_type` — the name from the catalog
   - `url` — your HTTPS endpoint
   - `store_id` — from step 1
   - `failure_type` — `Ignore` (default), `Error` (shows a message to the shopper), or `Fallback`
   - `fallback_url` — required when `failure_type` is `Fallback`
   - `cache_length` — `Short` (5 min), `Long` (30 min) or `NoCache`
4. **List and manage.** `/api/v1/webhooks` behaves like any other resource, so the standard `page`, `count`, `fields` and query-syntax parameters apply for reading, and `PUT`/`DELETE` on `/api/v1/webhooks/{id}` for updates.
5. **Correlate deliveries back to the API.** Order events carry order data; re-read the authoritative record with `get-orders` — `GET /orders?id=<n>&expand=items,payments,shipments`. Status names are merchant-defined, so resolve them with `get-order_statuses` rather than hard-coding strings.

## Rules

- **Three seconds.** Synchronous events time out at 3 seconds by design because the delay is visible to a shopper. Do all real work asynchronously and return the minimum payload.
- **Responses are cached.** With `cache_length: Short` or `Long`, the store reuses your reply for 5 or 30 minutes. A pricing or tax subscriber that must be exact per request has to set `NoCache`.
- **No signature and no retries are documented.** There is no HMAC header, no shared secret and no documented redelivery policy — authenticate the caller yourself (an unguessable path, mutual TLS, or an allowlist) and treat delivery as at-most-once.
- **`failure_type: Error` is shopper-visible.** Use it only where a failed callback genuinely should block checkout.
- **Payment events are authoritative.** `AuthorizingPayment` and `CapturingPayment` responses (`approved`, `declined`, `authorization_code`, `transaction_id`, `avs_code`, `reject_reason`, `outstanding_balance`) decide whether money moves. Never return a default-approve.
