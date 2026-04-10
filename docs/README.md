# SALDI documentation

Status: draft

This `docs/` folder is intended to become the canonical technical documentation hub for the SALDI repository.

SALDI is a large legacy PHP monolith with many business modules, shared platform code, and several integration surfaces. Because documentation already exists in multiple places, this folder should be used to point readers to the current source of truth and clearly separate active docs from historical/reference material.

## Start here
If you are new to the repository, start with:

1. Root [`README.md`](../README.md)
2. Root [`INSTALLATION.txt`](../INSTALLATION.txt)
3. [`getting-started/installation.md`](getting-started/installation.md)
4. [`architecture/module-map.md`](architecture/module-map.md)
5. Platform doc: [`platform/includes.md`](platform/includes.md)

## By audience

### New developers / maintainers
- [`architecture/module-map.md`](architecture/module-map.md)
- [`platform/includes.md`](platform/includes.md)
- [`platform/admin.md`](platform/admin.md)
- [`platform/index.md`](platform/index.md)
- [`platform/report-and-docs.md`](platform/report-and-docs.md)
- [`platform/frontend-assets.md`](platform/frontend-assets.md)
- [`modules/debitor.md`](modules/debitor.md)
- [`modules/finans.md`](modules/finans.md)
- [`modules/lager.md`](modules/lager.md)
- [`modules/kreditor.md`](modules/kreditor.md)
- [`modules/systemdata.md`](modules/systemdata.md)
- [`modules/produktion.md`](modules/produktion.md)
- [`modules/rental.md`](modules/rental.md)
- [`modules/booking.md`](modules/booking.md)
- [`modules/sager.md`](modules/sager.md)

### Ops / deployers
- Root [`INSTALLATION.txt`](../INSTALLATION.txt)
- Root [`LAESMIG.txt`](../LAESMIG.txt)
- [`getting-started/installation.md`](getting-started/installation.md)
- [`operations/deployment-and-troubleshooting.md`](operations/deployment-and-troubleshooting.md)
- [`operations/configuration-runbook.md`](operations/configuration-runbook.md)
- [`operations/backup-restore-runbook.md`](operations/backup-restore-runbook.md)
- [`operations/upgrades-and-releases.md`](operations/upgrades-and-releases.md)
- [`operations/verification-checklists.md`](operations/verification-checklists.md)

### Integrators
- `restapi/swagger.yaml`
- `restapi/swagger-ui.html`
- `restapi/IMPLEMENTATION_STATUS.md`
- `api/v2/README.md`
- [`integrations/restapi-overview.md`](integrations/restapi-overview.md)
- [`integrations/api-v2-endpoints.md`](integrations/api-v2-endpoints.md)
- [`integrations/legacy-webshop-booking.md`](integrations/legacy-webshop-booking.md)
- [`integrations/remote-booking.md`](integrations/remote-booking.md)
- [`integrations/soap-and-projectmanager.md`](integrations/soap-and-projectmanager.md)
- [`integrations/soap-operations.md`](integrations/soap-operations.md)
- [`integrations/projectmanager.md`](integrations/projectmanager.md)

### Product/support/power users
- Root `guides/`
- Root `doc/brugervejledning.pdf`
- release/history docs in `doc/`

## Core system areas
Current docs:
- [`platform/includes.md`](platform/includes.md) — shared runtime/platform layer
- [`platform/admin.md`](platform/admin.md) — admin console, backups, restore, licensing, settings
- [`platform/index.md`](platform/index.md) — login shell, iframe navigation, install redirect behavior
- [`platform/report-and-docs.md`](platform/report-and-docs.md) — report and document workflow helpers with side effects
- [`platform/frontend-assets.md`](platform/frontend-assets.md) — frontend shells, asset stacks, CSS/JS risk map
- [`modules/debitor.md`](modules/debitor.md) — customer and accounts-receivable flows
- [`modules/finans.md`](modules/finans.md) — journal, posting, finance, VAT, reporting
- [`modules/lager.md`](modules/lager.md) — inventory, receiving, stock, pricing, labels
- [`modules/kreditor.md`](modules/kreditor.md) — supplier/AP, purchase, receipt, payment, intake
- [`modules/systemdata.md`](modules/systemdata.md) — configuration, users, fiscal years, forms, POS, import/export
- [`modules/produktion.md`](modules/produktion.md) — production-order lifecycle and production reporting
- [`modules/rental.md`](modules/rental.md) — rental bookings, items, reservations, remote settings, linked orders
- [`modules/booking.md`](modules/booking.md) — small legacy booking gateway
- [`modules/sager.md`](modules/sager.md) — case management, planning, payroll/time, templates, attachments

Planned next areas to document:
- deeper field-level API payload examples
- more detailed SOAP request/response examples
- richer `projectManager/` schema and endpoint reference
- `remoteBooking/` endpoint-by-endpoint details
- more detailed include/global dependency maps and troubleshooting notes

## Integrations
Important integration areas in the repository:
- `api/` — legacy API/integration entry points
- `restapi/` — newer REST API
- `soapserver/` and `soapklient/` — SOAP integrations
- `sync_shop/` — webshop sync
- `dandomain/` — Dandomain integration
- `remoteBooking/` — booking/payment integration
- `projectManager/` — embedded project-management subsystem

Current drafted integration docs:
- [`integrations/restapi-overview.md`](integrations/restapi-overview.md)
- [`integrations/api-v2-endpoints.md`](integrations/api-v2-endpoints.md)
- [`integrations/legacy-webshop-booking.md`](integrations/legacy-webshop-booking.md)
- [`integrations/remote-booking.md`](integrations/remote-booking.md)
- [`integrations/soap-and-projectmanager.md`](integrations/soap-and-projectmanager.md)
- [`integrations/soap-operations.md`](integrations/soap-operations.md)
- [`integrations/projectmanager.md`](integrations/projectmanager.md)

These can later be expanded into deeper per-endpoint or per-operation pages where needed.

## Historical and reference docs
These existing repo docs are still valuable, but they do not all represent canonical current technical documentation:
- [`../README.md`](../README.md) — repo landing page
- [`../INSTALLATION.txt`](../INSTALLATION.txt) — primary install reference today
- [`../LAESMIG.txt`](../LAESMIG.txt) — Danish readme/entry document
- `../doc/` — release notes, change logs, user/reference materials
- `../guides/` — user-facing PDF guides

## Documentation conventions for this repo
- Prefer short module/platform docs over one giant manual.
- Document current behavior, not idealized architecture.
- Mark legacy vs current UI paths when both exist.
- Link to code-adjacent specs where they already exist.
- When code changes affect setup, schema, integrations, or workflows, update docs in the same PR.
