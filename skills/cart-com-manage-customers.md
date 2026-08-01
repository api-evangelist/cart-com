---
name: Manage customers and B2B profiles on a Cart.com Online Store
description: Find or create customers, attach addresses and custom fields, assign customer types for B2B pricing, and link customers to company profiles.
api: openapi/cart-com-online-store-openapi-original.yml
generated: '2026-07-31'
method: generated
operations:
  - get-customers
  - post-customers
  - put-customers-id
  - get-addresses
  - post-addresses
  - get-customer_types
  - get-profiles
  - post-customer_association
  - get-custom_fields
  - post-custom_field_values
---

# Manage customers and B2B profiles

## Before you start

- **Base URL.** `https://{storeDomain}/api/v1`, `X-AC-Auth-Token` on every request.
- **Scopes.** `people` (write) for customers/addresses/profiles, plus `custom_fields` if you write metadata. See `scopes/cart-com-scopes.yml`.
- **PII.** Customer records are personal data. Never request the `decrypt` scope for this flow — payment-instrument decryption lives on `/credit_cards/{id}/decrypted` and `/order_payments/{id}/decrypted` and is out of scope here.

## Steps

1. **Look the customer up by email.** `get-customers` — `GET /customers?email=<address>&fields=id,email,first_name,last_name`. There is no `GET /customers/{id}`; the collection plus query syntax is the only read path.
2. **Create if absent.** `post-customers` — `POST /customers`. Creating a customer fires the `CustomerCreated` webhook; a storefront self-registration fires `CustomerRegistered` instead.
3. **Update in place.** `put-customers-id` — `PUT /customers/{id}`. Changing the email address fires `CustomerEmailUpdated` with both `old_email` and `new_email`; any other change fires `CustomerUpdated`.
4. **Attach addresses.** `get-addresses` / `post-addresses` — `GET|POST /addresses` with `customer_id`. Order-time addresses are a separate resource (`/order_addresses`); do not conflate them.
5. **Assign the customer type.** `get-customer_types` — `GET /customer_types`, then set `customer_type_id` on the customer. Customer type is what drives B2B tiered pricing, payment-method availability, and the `customer_type_id` passed to the `GetPrice` and `GetDiscount` webhooks.
6. **Link to a company profile.** `get-profiles` — `GET /profiles` to find the company record, then `post-customer_association` — `POST /customer_association` to bind the customer to it. This is the B2B account hierarchy.
7. **Write metadata.** `get-custom_fields` — `GET /custom_fields` to resolve the field definition id, then `post-custom_field_values` — `POST /custom_field_values`. Cart.com has no free-form `metadata` map; custom fields are the supported extension point.
8. **Verify.** `GET /customers?id=<n>&expand=addresses,custom_fields,additional_emails`.

## Rules

- **Errors.** `{"status_code": n, "message": "...", "details": "..."}` — not RFC 9457. A `401` means the token is missing or invalid; a `404` on a list call usually means an out-of-range `page`, not an empty result set.
- **No idempotency key.** Always search by email before `POST /customers`, and re-search after any timed-out write, or you will create duplicate customers.
- **Rate limit.** `X-AC-Call-Limit` on every response; `429` + `Retry-After` when the 10-second window is exhausted. Bulk customer imports should use the `import` resource and the `import` scope rather than looping `POST /customers`.
- **Synchronous validation webhooks.** If the store subscribes `ValidateCustomer`, a subscriber has 3 seconds to approve or reject your write; a rejection surfaces as an error on your call.
