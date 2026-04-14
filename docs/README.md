# SALDI documentation

`docs/` is the canonical technical documentation hub for the SALDI repository.

SALDI is a large legacy PHP monolith with broad shared runtime behavior, business modules with hidden cross-coupling, and several generations of integration surfaces. Use these docs to find the current technical source of truth, and use `doc/` and `guides/` as historical or user-facing supplements.

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
- [`modules/troubleshooting-by-area.md`](modules/troubleshooting-by-area.md)

### Ops / deployers
- Root [`INSTALLATION.txt`](../INSTALLATION.txt)
- Root [`LAESMIG.txt`](../LAESMIG.txt)
- [`getting-started/installation.md`](getting-started/installation.md)
- [`operations/deployment-and-troubleshooting.md`](operations/deployment-and-troubleshooting.md)
- [`operations/configuration-runbook.md`](operations/configuration-runbook.md)
- [`operations/backup-restore-runbook.md`](operations/backup-restore-runbook.md)
- [`operations/upgrades-and-releases.md`](operations/upgrades-and-releases.md)
- [`operations/release-smoke-sheet.md`](operations/release-smoke-sheet.md)
- [`operations/verification-checklists.md`](operations/verification-checklists.md)

### Integrators
Start with the docs in `docs/integrations/` first, then consult code-adjacent specs/examples as supplementary material.

Primary docs:
- [`integrations/restapi-overview.md`](integrations/restapi-overview.md)
- [`integrations/api-v2-endpoints.md`](integrations/api-v2-endpoints.md)
- [`integrations/legacy-webshop-booking.md`](integrations/legacy-webshop-booking.md)
- [`integrations/remote-booking.md`](integrations/remote-booking.md)
- [`integrations/soap-and-projectmanager.md`](integrations/soap-and-projectmanager.md)
- [`integrations/soap-operations.md`](integrations/soap-operations.md)
- [`integrations/projectmanager.md`](integrations/projectmanager.md)

Supplementary code-adjacent references:
- `restapi/swagger.yaml`
- `restapi/swagger-ui.html`
- `restapi/IMPLEMENTATION_STATUS.md`
- `api/v2/README.md`

### Product/support/power users
- Root `guides/`
- Root `doc/brugervejledning.pdf`
- release/history docs in `doc/`

## Core system areas
Current docs:
- [`platform/includes.md`](platform/includes.md) — shared runtime/platform layer, globals, helpers, external tool dependencies
- [`platform/admin.md`](platform/admin.md) — admin console, backups, restore, licensing, settings, destructive workflows
- [`platform/index.md`](platform/index.md) — login shell, iframe navigation, cookies, install redirect behavior
- [`platform/report-and-docs.md`](platform/report-and-docs.md) — report and document workflows, side effects, preview/rendering paths
- [`platform/frontend-assets.md`](platform/frontend-assets.md) — shell/include matrix, asset stacks, CSS/JS ownership map
- [`modules/debitor.md`](modules/debitor.md) — customer and accounts-receivable flows
- [`modules/finans.md`](modules/finans.md) — journal, posting, finance, VAT, reporting
- [`modules/lager.md`](modules/lager.md) — inventory, receiving, stock, pricing, labels
- [`modules/kreditor.md`](modules/kreditor.md) — supplier/AP, purchase, receipt, payment, intake
- [`modules/systemdata.md`](modules/systemdata.md) — configuration, users, fiscal years, forms, POS, import/export
- [`modules/produktion.md`](modules/produktion.md) — production-order lifecycle and production reporting
- [`modules/rental.md`](modules/rental.md) — rental bookings, items, reservations, remote settings, linked orders
- [`modules/booking.md`](modules/booking.md) — small legacy booking gateway
- [`modules/sager.md`](modules/sager.md) — case management, planning, payroll/time, templates, attachments
- [`modules/troubleshooting-by-area.md`](modules/troubleshooting-by-area.md) — cross-module triage map for common failure types

## Integrations
Important integration areas in the repository:
- `api/` — legacy API/integration entry points
- `restapi/` — newer REST API
- `soapserver/` and `soapklient/` — SOAP integrations
- `sync_shop/` — webshop sync
- `dandomain/` — Dandomain integration
- `remoteBooking/` — booking/payment integration
- `projectManager/` — embedded project-management subsystem

Current integration docs:
- [`integrations/restapi-overview.md`](integrations/restapi-overview.md)
- [`integrations/api-v2-endpoints.md`](integrations/api-v2-endpoints.md)
- [`integrations/legacy-webshop-booking.md`](integrations/legacy-webshop-booking.md)
- [`integrations/remote-booking.md`](integrations/remote-booking.md)
- [`integrations/soap-and-projectmanager.md`](integrations/soap-and-projectmanager.md)
- [`integrations/soap-operations.md`](integrations/soap-operations.md)
- [`integrations/projectmanager.md`](integrations/projectmanager.md)

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

## Documentation confidence labels
Use these lightweight labels when a section benefits from provenance clarification:
- **Verified locally** — confirmed by running the flow in the audit environment
- **Code-inferred** — derived from code reading and repository inspection only
- **Legacy/reference** — historical or supporting material that may not describe the active runtime exactly

Use the label once per section or note, not on every sentence.
