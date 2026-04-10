# REST API Overview

Status: draft
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

Typical responsibilities handled by shared base code include:
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
- protected routes require a valid access token with the correct token type

### Tenant resolution
Tenant DB selection is handled primarily through:
- `tenant_id` embedded in the JWT
- fallback support for `X-Tenant-ID`

This means tenant context is partly in the token and partly compatible with header-based callers.

### Legacy compatibility
Older auth code still exists in `core/auth.php` and related paths, including header styles such as:
- `X-DB`
- `X-SaldiUser`
- API-key style `Authorization`

Treat this as backward-compatibility infrastructure, not the main model for new integrations.

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
`swagger.yaml` appears partially stale relative to runtime behavior.

Specifically:
- Swagger still documents older header/API-key auth patterns.
- Runtime code now centers on JWT Bearer auth plus tenant resolution.

If API behavior changes, update both code and spec artifacts together.

## Known implementation gaps
From `IMPLEMENTATION_STATUS.md`, areas still needing work include:
- `GET /invoices/{id}/pdf` placeholder behavior
- fuller email/send integration for invoice send flows
- partially implemented invoice update behavior

## Safe maintenance guidance
- Keep `BaseEndpoint`, `JWTAuth`, and endpoint docs in sync.
- When changing auth, verify both JWT-based callers and any required legacy compatibility paths.
- Treat tenant resolution as a first-class behavior and test it explicitly.
- Update `swagger.yaml`, `swagger-ui.html`, and `IMPLEMENTATION_STATUS.md` in the same change when endpoint behavior changes.

## Suggested future expansion for this doc
This draft should later be extended with:
- auth flow diagram
- tenant resolution notes
- endpoint catalog by business domain
- spec/runtime drift checklist
- testing conventions for API changes
