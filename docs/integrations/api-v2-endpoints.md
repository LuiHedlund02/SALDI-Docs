# API v2 endpoints

Status: draft
Audience: integrators, backend developers

This page documents the API surface under `api/v2/`, which is separate from the newer `restapi/` tree.

## Purpose
`api/v2/` is a lightweight JSON/HTTP API layer using API-key authentication. It currently exposes CRUD-style access to selected SALDI entities and includes basic API-key management helpers.

## Authentication model
Authentication is handled by:
- `api/v2/includes/AuthMiddleware.php`
- `api/v2/includes/ApiKeyManager.php`

### Current auth behavior
- clients send `X-API-Key`
- middleware reads it from headers / `HTTP_X_API_KEY`
- key validation happens against the `api_keys` table in a master DB
- only active keys are accepted
- after validation, the middleware switches DB context to the database named by the API-key record

### Important note
This is different from the JWT-based `restapi/` authentication model.

## Request/response style
Observed characteristics:
- JSON-first API
- HTTP verbs: `GET`, `POST`, `PUT`, `DELETE`
- `POST`/`PUT` parse JSON from `php://input`
- responses are JSON objects/arrays with HTTP status codes
- CORS is permissive (`*`) in the current implementation

## Documented endpoints
### `addresses.php`
This is the only endpoint clearly documented in `api/v2/README.md`.

Observed operations:
- `GET /api/v2/addresses.php` — list addresses
- `GET /api/v2/addresses.php?id=123` — fetch one address
- `POST /api/v2/addresses.php` — create address
- `PUT /api/v2/addresses.php?id=123` — update address
- `DELETE /api/v2/addresses.php?id=123` — delete address

Underlying table/domain:
- `adresser`

## Underdocumented but present endpoints
### `ordrer.php`
Observed operations:
- `GET` all or single order (`?id=`)
- list/filter by `konto_id`, `status`, `limit`
- optional `include_lines=true`
- `POST`, `PUT`, `DELETE` support for orders

Underlying domain:
- `ordrer`

Notable behavior:
- special handling for null/default values
- order-number generation logic
- optional embedding of order lines in the response

### `ordrelinjer.php`
Observed operations:
- `GET` one line (`?id=`) or lines by order (`?ordre_id=`)
- `POST`, `PUT`, `DELETE` support for order lines

Underlying domain:
- `ordrelinjer`

## API-key admin/helper endpoints
Other files present in `api/v2/`:
- `manage_api_keys.php`
- `manage_api_keys.html`
- `setup_api_key.php`
- `verify_api_key.php`
- `test.php`

These appear to support API-key creation/testing/verification, but they are not covered in the README and should be treated as admin/support tooling until documented more fully.

## Documentation gaps
The written docs currently lag behind the code.

Currently documented:
- address CRUD
- generic auth header
- JSON response format
- error codes

Currently not documented in detail:
- order CRUD
- order-line CRUD
- API-key management endpoints
- field-by-field payload expectations
- tenant/database switching behavior after key validation

## Implementation cautions
Observed technical cautions:
- SQL is built dynamically with escaping, not prepared statements
- auth middleware logs header/API-key presence to `error_log`
- permissive CORS may not be appropriate for every deployment

## Suggested next expansion
This draft should later be extended with:
- payload field reference for each endpoint
- example requests/responses
- API-key lifecycle guide
- DB/tenant switching notes
- comparison with `restapi/` so integrators know which surface to use
