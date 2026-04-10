# Legacy webshop and booking integrations

Audience: integrators, maintainers, support

This page covers the older integration surfaces used by SALDI for webshop sync, Dandomain imports, and booking/payment flows. These scripts are still active, but they are legacy entry points. For new work, prefer newer API surfaces when feasible.

## 1. Legacy API dispatcher: `api/rest_api.php`
`api/rest_api.php` is the main query-string based legacy action dispatcher.

### Current responsibilities
- table/data helpers such as `fetch_from_table`, `update_table`, `insert_into_table`
- webshop order insertion flows such as `insert_shop_order` and `insert_shop_orderline`
- supporting actions such as `getNextAccountNo`, `createDebitor`, `deleteOrder`, invoice/order handling, and related helpers

### Important traits
- uses request/query parameters for access control and routing
- validates account/API access against SALDI data/config
- writes logs under `temp/`
- restricts generic CRUD helpers to selected tables
- contains significant mapping logic from external shop data into SALDI orders/order lines

### Auth and request contract warnings
This dispatcher is not self-describing. In practice, callers need the correct combination of values such as:
- `db`
- `saldiuser`
- `key`
- action/query-string flags selecting the operation
- client IP matching any configured allowlist logic

Observed failure strings in this area include variants such as:
- `Missing saldiuser`
- `Access denied (key)`
- `Wrong IP`

Treat the query-string contract as code-defined, not spec-defined.

## 2. Newer API layer in `api/v2/`
`api/v2/` is a more modern API surface than `rest_api.php`.

Current behavior:
- uses API-key authentication via `X-API-Key`
- validates keys through `AuthMiddleware` and `ApiKeyManager`
- supports HTTP-verb/JSON style requests
- currently documents address/order/order-line CRUD

For new integrations that still need the `api/` tree rather than `restapi/`, prefer `api/v2/` over the older dispatcher.

## 3. Bulk webshop sync: `sync_shop/`
Main file:
- `sync_all_products.php`

Current behavior:
- bulk-syncs products, stock, and pricing to configured webshop endpoints
- reads target API URLs from `grupper`
- reads product/inventory data from SALDI tables
- handles assemblies/part items in some paths
- launches background `curl` calls and writes logs to `temp/`

### Operational concerns
- depends on server-side `curl`
- is batch/job oriented rather than request/response oriented
- can affect many products in one run
- should be tested with logging enabled because partial failures can be hard to spot from the UI alone

## 4. Dandomain integrations: `dandomain/`
This folder contains customer-/shop-specific Dandomain integration scripts.

### Common pattern
- fetch orders via Dandomain SOAP/WSDL
- normalize customer/order/line data
- insert data into SALDI through `api/rest_api.php`
- sometimes push stock/price changes back to Dandomain

### Important files
- `customAudio.php`
- `havemoebelland.php`
- `index.php`
- `service.wsdl`
- `INTEGRATION_OVERSIGT_CUSTOM_AUDIO.md` / `.html`

This is one of the better-documented legacy integration areas, but it is still customer/script specific.

## 5. Booking/payment integration: `remoteBooking/`
`remoteBooking/` is a standalone booking frontend plus backend/payment/mail helpers.

### Main pieces
- `index.php` — booking UI shell
- `index.js` — client flow
- `api.php` — backend JSON entry point
- `createCust.php` — customer creation
- `vibrantPaymentLink.php` / `vibrantPaymentIntent.php` — payment link + status flow
- `sendMail.php` and related mail helpers — booking confirmation flow

### Current booking flow
1. load products and dates
2. create/reuse customer
3. create booking/order in SALDI
4. open payment flow
5. poll payment status
6. update booking and send confirmation mail

This area is tightly coupled to rental/order creation and booking-related tables.

## Key risks
1. **Legacy auth/routing risk** — older dispatcher endpoints depend on query-string conventions and legacy access checks.
2. **Batch sync blast radius** — `sync_all_products.php` can update many products and prices in one run.
3. **Customer-specific script drift** — Dandomain scripts may diverge from one another over time.
4. **Booking/payment coupling risk** — booking success depends on both order creation and payment/mail flows.
5. **Limited standalone docs** — some behaviors still live primarily in code, especially in `remoteBooking/` and sync scripts.

## Troubleshooting and verification
- If a shop import fails, inspect both the Dandomain/customer-specific script and the matching `api/rest_api.php` action path.
- If product sync behaves oddly, check `sync_shop/sync_all_products.php`, the configured target URL in `grupper`, and temp logs together.
- If public booking fails, test the full flow: products → customer → booking → payment → mail.
