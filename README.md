# Cart.com

Cart.com is a unified commerce and logistics provider for B2C and B2B brands, combining an ecommerce storefront platform (the former AmeriCommerce Online Store), marketplace and channel management, order management, warehouse management, and a nationwide fulfillment network.

- Website: https://cart.com/
- Developer portal: https://developers.cart.com/
- GitHub: https://github.com/AmeriCommerce
- Status: https://status.cart.com/
- Release notes: https://cart.canny.io/changelog

## APIs

**Cart.com Online Store API** — REST/JSON, OpenAPI 3.1, 136 paths / 262 operations / 73 schemas, served from each merchant's own storefront domain at `https://{storeDomain}/api/v1`. Auth is an `X-AC-Auth-Token` header, issued either from the admin console or through an OAuth 2 authorization-code flow with a coarse read/write scope vocabulary. Thirty webhook event types, ten of which are synchronous and shape storefront behaviour (pricing, tax, discount, validation, payment authorization).

## Artifacts

| Artifact | Method |
|---|---|
| `openapi/` | searched — harvested verbatim from `AmeriCommerce/rest-api` |
| `overlays/` | generated |
| `authentication/`, `scopes/` | derived / searched |
| `conventions/`, `rate-limits/`, `lifecycle/`, `changelog/` | searched |
| `asyncapi/` (webhook catalog) | searched — no AsyncAPI published |
| `errors/`, `data-model/`, `agentic-access/`, `mcp/` | derived |
| `packages/`, `examples/`, `components/`, `sandbox/` | searched |
| `conformance/`, `well-known/`, `security/` | derived / probed |
| `skills/`, `llms/` | generated |

## Not published by Cart.com

No `/.well-known/` documents (no security.txt, OpenID/OAuth discovery, or api-catalog), no A2A agent card, no MCP server, no AsyncAPI, no first-party SDK in any package registry, no CLI, no trust center or named compliance certifications, no public Postman workspace, no idempotency contract, and no test/sandbox credentials.
