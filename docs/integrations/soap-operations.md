# SOAP operations

Audience: integrators, maintainers

This page provides a practical reference for the legacy SOAP layer in `soapserver/` and `soapklient/`.

## Important warning
Although this integration uses SOAP/WSDL, the effective contract is not strongly typed. Most operations take a single string argument with tab-separated fields.

Treat this as a legacy compatibility API, not a modern service contract.

## Runtime prerequisites
Before using SOAP in a real environment, confirm:
- base URLs for the generated/fetched WSDL are correct
- `soapklient/soapfunc.php` can write local WSDL copies
- temp paths under `temp/` or `temp/$db/` are writable
- character-encoding behavior has been tested with Danish and other non-ASCII data
- the caller can keep and reuse the returned PHP session ID

## Common auth/session model
### `logon`
Purpose:
- authenticate user against SALDI
- create PHP session / `online` row
- return the `session_id`

Input grammar:
- `regnskab<TAB>brugernavn<TAB>password`

Typical success response:
- `0<TAB><session_id>`

Failure behavior:
- caller should expect a non-zero/error-prefixed string rather than a strongly typed SOAP fault contract

### `logoff`
Purpose:
- invalidate the SOAP session by removing the related `online` row

Input grammar:
- `session_id`

All later SOAP operations assume a valid session ID created by `logon`.

## Data/query operations
### `singleselect`
Purpose:
- execute a single-row style select query

Input grammar:
- `session_id<TAB>SQL`

Notes:
- accepts raw SQL text
- one of the riskiest operations because it depends on caller-supplied query text
- output shape is string-based and caller-parsed, not a typed SOAP object graph

### `multiselect`
Purpose:
- execute a multi-row select/export style query

Input grammar:
- `session_id<TAB>SQL`

Notes:
- may write CSV-like output into `../temp/$db/`
- depends on writable temp paths
- large result sets should be treated as export-style operations, not lightweight reads

## Write operations
### `singleinsert`
Purpose:
- insert a row through a SQL-like request

Notes:
- parsing is formatting-sensitive
- ID lookup uses legacy `max(id)`-style inference patterns in some paths
- caller should not assume transactional safety across multiple inserts

### `singleupdate`
Purpose:
- update a row through a SQL-like request

Notes:
- also accepts raw SQL-like payloads
- return semantics are weaker/inconsistent compared with modern JSON APIs

### `addorderline`
Purpose:
- add an order line to an order

Notes:
- likely used by webshop/order sync flows
- validate carefully against current order expectations before using it in new integrations

### `invoice`
Purpose:
- invoice/order-related SOAP action

Notes:
- one of the business-specific legacy operations layered on top of the same session model
- should be treated as a high-risk action because it touches real ERP state, not only export data

## Example request patterns
These are representative string shapes, not full WSDL objects.

### Logon example
```text
DemoRegnskab<TAB>apiuser<TAB>secret
```

### Single select example
```text
<session_id><TAB>select id,firmanavn from adresser where art='D' order by id limit 1
```

### Multi select example
```text
<session_id><TAB>select id,ordrenr,sum,moms from ordrer order by id desc
```

## Success/error patterns to expect
Common practical patterns:
- success often starts with `0<TAB>...`
- failures are string-based and operation-specific
- parsing is fragile because tabs and exact formatting matter
- malformed SQL-like payloads can fail before the caller gets a clean business error

## Client-side wrapper behavior
`soapklient/soapfunc.php` wraps these operations and:
- downloads/rewrites WSDL files dynamically
- writes local WSDL copies
- depends on base URL assumptions
- performs encoding conversion between legacy encodings and UTF-8

This means environment configuration is part of the effective contract.

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
