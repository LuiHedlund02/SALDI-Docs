# REST API Overview

Audience: integrators, backend developers, maintainers

The `restapi/` tree contains the current SALDI REST API implementation. It is structured as a PHP API with shared infrastructure, endpoint handlers, business models/services, and local test/spec artifacts.

## Structure
Main areas:
- `core/` — auth, JWT handling, CORS, logging, exceptions, base endpoint behavior
- `models/` — DB-backed domain models
- `services/` — higher-level business logic used by endpoints
- `endpoints/v1/` — runtime v1 handlers and router
- `tests/` — API-focused test scripts
- `swagger.yaml` / `swagger-ui.html` — API spec and browser UI
- `IMPLEMENTATION_STATUS.md` — practical implementation summary

## Request flow
Most v1 requests route through `endpoints/v1/index.php`, which dispatches to endpoint files under `endpoints/v1/`.

Typical shared responsibilities handled by base code include:
1. request logging
2. auth checking
3. tenant database resolution
4. JSON response formatting
5. shared error handling

In practice, many endpoints are thin wrappers around model/service code.

## Authentication model
### Current runtime model
The active model is JWT/Bearer based:
- clients send `Authorization: Bearer <access_token>`
- `POST /auth/login` issues access + refresh tokens
- `POST /auth/refresh` issues a new access token
- protected routes require a valid access token with token type `access`

### Login request
Runtime login expects:
```json
{
  "username": "brugernavn",
  "password": "password",
  "account_name": "regnskab-navn"
}
```

Example success response shape:
```json
{
  "success": true,
  "data": {
    "access_token": "...",
    "refresh_token": "...",
    "token_type": "Bearer",
    "expires_in": 3600,
    "user": {
      "id": 1,
      "username": "brugernavn",
      "is_admin": false
    },
    "tenant": {
      "id": 1,
      "name": "Demo",
      "db": "demo_db"
    }
  },
  "message": "Login successful"
}
```

Important runtime facts:
- users are stored in each tenant DB, not only in the master DB
- `account_name` is required because login first resolves the tenant from `regnskab`
- the login path rejects closed tenants
- passwords are checked against both legacy MD5 and `saldikrypt()`-style hashes

### Refresh request
`POST /auth/refresh` expects:
```json
{ "refresh_token": "..." }
```

Example success shape:
```json
{
  "success": true,
  "data": {
    "access_token": "...",
    "token_type": "Bearer",
    "expires_in": 3600
  },
  "message": "Token refreshed successfully"
}
```

## Tenant resolution
Tenant DB selection is handled primarily through:
- `tenant_id` embedded in the JWT during login
- fallback support for `X-Tenant-ID`

Runtime behavior from `JWTAuth.php`:
1. validate bearer token
2. read `tenant_id` from token if present
3. otherwise try `X-Tenant-ID`
4. resolve the real tenant DB from the master `regnskab` table
5. connect to that DB before protected endpoint logic runs

This means tenant context is partly in the token and partly compatible with header-based callers.

## Response shape
Most endpoints use the `BaseEndpoint` wrapper response shape:
```json
{
  "success": true,
  "data": { ... },
  "message": "..."
}
```

Error cases typically keep the same top-level structure with `success: false` and an HTTP status matching the failure.

## Endpoint organization
Observed endpoint groups include:
- `auth/`
- `user/`
- `dashboard/`
- `notifications/`
- `accounts/`
- `accountingYear/`
- `attachment/`
- `currencies/`
- `labels/`
- `products/`
- `inventory/`
- `debitor/...`
- `creditor/...`
- `vat/` and `vat-codes/`

Patterns observed:
- many resources expose `index.php` handlers inside their resource folders
- some endpoints are CRUD-like
- some endpoints are action-oriented, such as send/PDF/register flows

## Documentation and spec status
Key documentation artifacts:
- `restapi/swagger.yaml`
- `restapi/swagger-ui.html`
- `restapi/IMPLEMENTATION_STATUS.md`
- `restapi/Krav Spec for API.pdf`
- endpoint-specific notes under subfolders
- `tests/` for executable examples/verification

### Important warning
`swagger.yaml` is partially stale relative to runtime behavior.

Most importantly:
- Swagger still documents older header/API-key auth patterns.
- Runtime code now centers on JWT Bearer auth plus tenant resolution.

If API behavior changes, update both code and spec artifacts together.

## Known implementation gaps
From `IMPLEMENTATION_STATUS.md`, areas still needing work include:
- `GET /invoices/{id}/pdf` placeholder behavior
- fuller email/send integration for invoice send flows
- partially implemented invoice update behavior

## Safe maintenance guidance
- Keep `BaseEndpoint`, `JWTAuth`, endpoint docs, and `swagger.yaml` in sync.
- When changing auth, verify both JWT-based callers and any required legacy compatibility paths.
- Treat tenant resolution as a first-class behavior and test it explicitly.
- Use `tests/` and runtime logging together when validating endpoint changes.
