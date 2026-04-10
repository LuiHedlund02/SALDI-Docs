# Module Map

Status: draft

This document gives a high-level map of the SALDI codebase. It is intended as a navigation aid for a large PHP monolith where functionality is spread across business modules plus a shared platform layer.

## Platform
These directories provide the application shell, shared runtime behavior, and reusable UI/backend helpers.

- `index/` — login, entry points, and main application shell/navigation.
- `admin/` — administration, backup/restore, licensing, and maintenance workflows.  
  See: `docs/platform/admin.md`
- `includes/` — shared runtime core: DB access, auth/session bootstrap, common helpers, document handling, order/report support.  
  See: `docs/platform/includes.md`
- `stdFunc/` — reusable utility helpers used across modules.
- `reportFunc/` — shared report logic and reporting-related helpers.  
  See also: `docs/platform/report-and-docs.md`
- `docsIncludes/` — document/attachment-related include logic and supporting routines.  
  See also: `docs/platform/report-and-docs.md`
- `javascript/` — shared client-side scripts and legacy frontend logic.  
  See also: `docs/platform/frontend-assets.md`
- `css/` — shared stylesheets and UI presentation assets.  
  See also: `docs/platform/frontend-assets.md`
- `img/` / `ikoner/` — static assets and icons.

## Core business modules
These directories contain the main ERP/business logic.

- `debitor/` — customer/accounts receivable, orders, invoices, payments, dunning, CRM, and related output flows.  
  See: `docs/modules/debitor.md`
- `finans/` — cash journal, posting, reconciliation, budgets, VAT, financial reporting, SAF-T.  
  See: `docs/modules/finans.md`
- `lager/` — product master, stock, receiving, counting, transfers, pricing, labels, inventory reports.  
  See: `docs/modules/lager.md`
- `produktion/` — production/manufacturing workflows and production-order related logic.  
  See: `docs/modules/produktion.md`
- `kreditor/` — supplier/accounts payable, vendor-side order/payment flows.  
  See: `docs/modules/kreditor.md`
- `booking/` — booking-oriented entry points and reservation-related UI.  
  See: `docs/modules/booking.md`
- `rental/` — rental workflows and reservation/calendar behavior.  
  See: `docs/modules/rental.md`
- `sager/` — case/job/task-like workflows and related operational tracking.  
  See: `docs/modules/sager.md`
- `systemdata/` — system-level configuration, users, master/reference data, forms, menu/POS settings, GDPR-related settings.  
  See: `docs/modules/systemdata.md`

## Integrations
These areas connect SALDI to external systems, APIs, partner tools, or specialized subsystems.

- `api/` — legacy integration/API entry points.  
  See also: `docs/integrations/legacy-webshop-booking.md`
- `restapi/` — newer REST API with OpenAPI/Swagger artifacts.  
  See: `docs/integrations/restapi-overview.md`
- `soapserver/` — SOAP services exposed by SALDI.  
  See also: `docs/integrations/soap-and-projectmanager.md`
- `soapklient/` — SOAP client wrappers and external-sync helpers.  
  See also: `docs/integrations/soap-and-projectmanager.md`
- `sync_shop/` — webshop synchronization routines.  
  See also: `docs/integrations/legacy-webshop-booking.md`
- `dandomain/` — Dandomain-specific integration scripts.  
  See also: `docs/integrations/legacy-webshop-booking.md`
- `remoteBooking/` — booking/payment/mail integration surface.  
  See also: `docs/integrations/legacy-webshop-booking.md`
- `projectManager/` — embedded project-management subsystem with its own schema/docs.  
  See also: `docs/integrations/soap-and-projectmanager.md`

## Supporting and historical areas
These areas are important, but are often reference, historical, or support-oriented rather than core module boundaries.

- `doc/` — release notes, change logs, reference material, PDFs.
- `guides/` — user-facing guide PDFs.
- `test/`, `testing/`, `tests/` — test or verification-related areas.
- `oldDesign/` — legacy design assets/templates.
- import/conversion utilities such as `ConvertTables/`, `converter/`, `createJson/`, `importfiler/`, `concense-kontoplaner/`.

## Legacy and newer UI paths coexist
SALDI contains both older and newer UI implementations.

In practice this means:
- some modules have a modernized list or header flow alongside older entry pages
- many screens still depend on shared legacy platform code in `includes/`, `javascript/`, and `css`
- module boundaries help navigation, but cross-module coupling is common

When changing code, do not assume one visible screen is the only active path. Check for older entry pages, include-based flows, and integration-specific variants.

## Current documentation coverage
Current documentation in this repository includes:
- `docs/README.md`
- `docs/getting-started/installation.md`
- `docs/operations/deployment-and-troubleshooting.md`
- `docs/operations/configuration-runbook.md`
- `docs/operations/backup-restore-runbook.md`
- `docs/operations/upgrades-and-releases.md`
- `docs/operations/verification-checklists.md`
- `docs/architecture/module-map.md`
- `docs/platform/includes.md`
- `docs/platform/admin.md`
- `docs/platform/index.md`
- `docs/platform/report-and-docs.md`
- `docs/platform/frontend-assets.md`
- `docs/modules/debitor.md`
- `docs/modules/finans.md`
- `docs/modules/lager.md`
- `docs/modules/kreditor.md`
- `docs/modules/systemdata.md`
- `docs/modules/produktion.md`
- `docs/modules/rental.md`
- `docs/modules/booking.md`
- `docs/modules/sager.md`
- `docs/integrations/restapi-overview.md`
- `docs/integrations/api-v2-endpoints.md`
- `docs/integrations/legacy-webshop-booking.md`
- `docs/integrations/remote-booking.md`
- `docs/integrations/soap-and-projectmanager.md`
- `docs/integrations/soap-operations.md`
- `docs/integrations/projectmanager.md`

Recommended next additions:
- deeper field-level API payload examples
- more detailed include/global dependency maps
- deeper report action and document lifecycle detail
- module-specific troubleshooting guides where needed
