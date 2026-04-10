# API v2 endpoints

Audience: integrators, backend developers

This page documents the API surface under `api/v2/`, which is separate from the newer JWT-based `restapi/` tree.

## Purpose
`api/v2/` is a lightweight JSON/HTTP API layer using API-key authentication. It currently exposes CRUD-style access to selected SALDI entities and includes basic API-key management helpers.

## Authentication model
Authentication is handled by:
- `api/v2/includes/AuthMiddleware.php`
- `api/v2/includes/ApiKeyManager.php`

### Current auth behavior
- clients send `X-API-Key`
- middleware accepts both `X-API-Key` and lowercase header variants
- if no key is found, the endpoint returns `401` with `{"error":true,"message":"API key is required"}`
- key validation is performed against an `api_keys` table in a master database connection
- only active keys are accepted
- after validation, the middleware switches DB context to the tenant database named in the API-key record

### Important caution
The current code hardcodes the master validation connection to database `develop` in `ApiKeyManager.php`. Treat this as deployment-sensitive and confirm it before rolling out API v2 in another environment.

## Request/response style
Common runtime traits:
- JSON-first API
- HTTP verbs: `GET`, `POST`, `PUT`, `DELETE`
- `POST`/`PUT` parse JSON from `php://input`
- responses are JSON objects or arrays with HTTP status codes
- CORS is permissive (`*`) in the current implementation

## Endpoints

### `addresses.php`
Underlying table/domain:
- `adresser`

Supported operations:
- `GET /api/v2/addresses.php` — list all addresses
- `GET /api/v2/addresses.php?id=123` — fetch one address
- `POST /api/v2/addresses.php` — create address
- `PUT /api/v2/addresses.php?id=123` — update address
- `DELETE /api/v2/addresses.php?id=123` — delete address

Example create request:
```http
POST /api/v2/addresses.php
X-API-Key: <key>
Content-Type: application/json

{
  "firmanavn": "Example ApS",
  "addr1": "Examplevej 1",
  "postnr": "2100",
  "bynavn": "København Ø",
  "email": "info@example.test",
  "art": "D"
}
```

Example create response:
```json
{
  "id": 123,
  "message": "Address created successfully"
}
```

Notes:
- `id` is ignored on create/update payloads
- payload fields are inserted largely as sent, so callers must match the actual `adresser` schema for their installation

### `ordrer.php`
Underlying domain:
- `ordrer`

Supported operations:
- `GET /api/v2/ordrer.php` — list orders, default limit `100`
- `GET /api/v2/ordrer.php?id=123` — fetch one order
- `GET /api/v2/ordrer.php?konto_id=10&status=0&limit=50` — filtered list
- `GET /api/v2/ordrer.php?id=123&include_lines=true` — fetch one order with embedded order lines
- `POST /api/v2/ordrer.php` — create order
- `PUT /api/v2/ordrer.php?id=123` — update order

Required create fields:
- `konto_id`
- `art`

Defaulting behavior on create:
- `status = 0`
- `ordredate = current date`
- `valuta = 'DKK'`
- `valutakurs = 100`
- `momssats = 25`
- `ordrenr` is generated as `MAX(ordrenr)+1` within the same `art`

Example create request:
```http
POST /api/v2/ordrer.php
X-API-Key: <key>
Content-Type: application/json

{
  "konto_id": 10,
  "art": "DO",
  "ordredate": "2026-04-10",
  "betalingsbet": "Netto",
  "adresse1": "Examplevej 1",
  "postnr": "2100",
  "bynavn": "København Ø"
}
```

Example create response:
```json
{
  "id": 456,
  "ordrenr": 1042,
  "message": "Order created successfully"
}
```

Important runtime notes:
- null/empty handling is field-sensitive; some numeric/business fields are forced to `NULL`, others to empty string
- `include_lines=true` enriches the response with joined `ordrelinjer` plus product/variant data where available
- numeric and boolean fields are cast in response formatting, so output types can differ from raw table output

### `ordrelinjer.php`
Underlying domain:
- `ordrelinjer`

Supported operations:
- `GET /api/v2/ordrelinjer.php` — list all lines
- `GET /api/v2/ordrelinjer.php?id=55` — fetch one line
- `GET /api/v2/ordrelinjer.php?ordre_id=456` — list lines for one order
- `POST /api/v2/ordrelinjer.php` — create line
- `PUT /api/v2/ordrelinjer.php?id=55` — update line

Example create request:
```http
POST /api/v2/ordrelinjer.php
X-API-Key: <key>
Content-Type: application/json

{
  "ordre_id": 456,
  "posnr": 1,
  "vare_id": 77,
  "beskrivelse": "Rental line",
  "antal": 1,
  "pris": 1000
}
```

Example create response:
```json
{
  "id": 55,
  "message": "Order line created successfully"
}
```

## API-key support/admin endpoints
These files exist in `api/v2/` and should be treated as admin/support tooling, not public business endpoints:
- `manage_api_keys.php`
- `manage_api_keys.html`
- `setup_api_key.php`
- `verify_api_key.php`
- `test.php`

### `manage_api_keys.php`
Actions:
- `GET ?action=list` — list masked API keys
- `POST ?action=create` — create a key using form fields `database`, `description`, `created_by`
- `PUT ?action=deactivate` — deactivate a key using JSON body with `api_key`

### `setup_api_key.php`
Purpose:
- bootstrap helper that creates the `api_keys` table if missing and inserts/reactivates a hardcoded test key

Treat this as environment setup/testing code, not a production-safe key provisioning workflow.

### `verify_api_key.php`
Purpose:
- diagnostic helper that checks whether the hardcoded test key exists and is active

## Error shapes and common status codes
Typical responses:
```json
{ "error": true, "message": "Invalid API key" }
```

Common status codes:
- `200` — success
- `201` — created
- `400` — bad input or missing ID/required field
- `401` — missing/invalid API key
- `404` — entity not found
- `405` — unsupported method
- `500` — DB or connection failure

## Choosing API v2 vs `restapi/`
Prefer `api/v2/` when:
- you need the existing API-key CRUD surface already implemented there
- you are integrating with older tooling that expects simple PHP endpoints

Prefer `restapi/` when:
- you want the newer JWT/Bearer auth model
- you need broader endpoint coverage and the newer endpoint structure
- you want tenant selection tied to login/token flow rather than static API keys

## Implementation cautions
- SQL is built dynamically with escaping, not prepared statements.
- auth middleware logs header/API-key presence to `error_log`.
- permissive CORS may not be appropriate for every deployment.
- API-key admin helpers are lightly protected and should not be exposed carelessly.
- master DB selection in `ApiKeyManager.php` must be reviewed during deployment.
