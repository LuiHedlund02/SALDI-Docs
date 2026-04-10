# SOAP operations

Status: draft
Audience: integrators, maintainers

This page provides a more detailed reference for the legacy SOAP layer in `soapserver/` and `soapklient/`.

## Important warning
Although this integration uses SOAP/WSDL, the effective contract is not strongly typed. Most operations take a single string argument with tab-separated fields.

Treat this as a legacy compatibility API, not a modern service contract.

## Common auth/session model
### `logon`
Purpose:
- authenticate user against SALDI
- create PHP session / `online` row
- return the `session_id`

Input pattern:
- `regnskab<TAB>brugernavn<TAB>password`

Output pattern:
- success appears to return `0<TAB><session_id>`

### `logoff`
Purpose:
- invalidate the SOAP session by removing the related `online` row

Input pattern:
- `session_id`

All later SOAP operations assume a valid session ID created by `logon`.

## Data/query operations
### `singleselect`
Purpose:
- execute a single-row style select query

Input pattern:
- `session_id<TAB>SQL`

Notes:
- accepts raw SQL text
- this is one of the riskiest operations because it depends on caller-supplied query text

### `multiselect`
Purpose:
- execute a multi-row select/export style query

Input pattern:
- `session_id<TAB>SQL`

Notes:
- may write CSV-like output into `../temp/$db/`
- depends on writable temp paths

## Write operations
### `singleinsert`
Purpose:
- insert a row through a SQL-like request

Notes:
- parsing appears fragile and formatting-sensitive
- new ID lookup uses legacy `max(id)`-style inference patterns

### `singleupdate`
Purpose:
- update a row through a SQL-like request

Notes:
- also accepts raw SQL-like payloads
- return semantics appear weak/inconsistent in the current code

### `addorderline`
Purpose:
- add an order line to an order

Notes:
- likely used by webshop/order sync flows
- should be validated carefully against current order expectations before use in new integrations

### `invoice`
Purpose:
- invoice/order-related SOAP action

Notes:
- one of the business-specific legacy operations layered on top of the same session model

## Client-side wrapper behavior
`soapklient/soapfunc.php` wraps these operations and:
- downloads/rewrites WSDL files dynamically
- writes local WSDL copies
- depends on base URL assumptions
- performs encoding conversion between legacy encodings and UTF-8

## Main risks
1. raw-SQL style payloads in select/update/insert operations
2. weakly typed request contract despite SOAP/WSDL packaging
3. fragile parsing based on tab separators and specific SQL formatting
4. dependency on writable temp paths and live WSDL fetches
5. legacy auth/session model based on PHP session IDs

## Safe usage guidance
- validate all input strictly before calling SOAP operations
- avoid assuming full SOAP schema guarantees
- confirm base URLs and writable WSDL/temp paths in each environment
- test non-ASCII data explicitly
- prefer newer API surfaces for new integrations where possible

## Suggested next expansion
This draft should later be extended with:
- per-operation exact field format examples
- success/error response examples
- WSDL-to-runtime mismatch notes
- deployment prerequisites checklist
