# `includes/` platform layer

Status: draft
Audience: maintainers, backend developers, reviewers

## Purpose
`includes/` is the shared runtime layer for SALDI. It is not a small helper folder; it acts as the platform core used by many modules across the monolith.

This area provides:
- database access and query wrappers
- session and user/account bootstrap
- shared utility functions
- document and attachment handling
- order/report support code
- common UI fragments and legacy presentation helpers

## Why this folder matters
Changes here can affect multiple business modules at once. A small change in `includes/` can alter login behavior, DB access, reporting, document flows, translations, or UI rendering across the application.

## Main entry files

### `online.php`
Primary runtime/bootstrap include used by many pages.

Responsibilities observed in code:
- reads runtime context from the `online` table using session ID
- loads account/user state such as database, rights, language, and current account
- updates `online.logtime`
- reads and sometimes initializes values in `settings`
- sets timezone and tool-path variables like `ps2pdf`, `pdftk`, `ftp`, `dbdump`, `tar`, `zip`, `weasyprint`
- checks for expired sessions and redirects/exits when needed
- can trigger upgrade/version checks when account DB version lags application version

Operational note:
This file has significant side effects on include/load. It should be treated as part of the request lifecycle, not as a pure helper.

### `db_query.php`
Shared database abstraction and error-handling layer.

Responsibilities observed in code:
- connects to PostgreSQL or MySQLi depending on configured DB type
- exposes core query helpers such as `db_connect()`, `db_select()`, `db_modify()`, `db_close()`
- applies query checks via `injecttjek()` before modification queries
- logs SQL activity into per-database temp files
- sends error mail and can emit UI alerts and terminate execution on failure

Operational note:
This is not a thin DB wrapper. It contains logging, alerting, and failure-handling behavior that affects application flow.

### `std_func.php`
Large shared utility hub used broadly across the system.

Typical responsibilities:
- formatting and conversion helpers
- language/text lookup helpers
- date and decimal handling
- shared business utilities used by multiple modules
- UI helper functions such as alert/notification-style output

## Important subareas

### `stdFunc/`
A more split-out helper library inside `includes/`.

Examples of responsibilities inferred from filenames:
- translation lookup (`findTxt.php`)
- fiscal-year logic (`fiscalYear.php`, `checkOpenFiscalYear.php`)
- schema/table safety helpers (`ensureTableAndColumns.php`)
- navigation helpers (`navStack.php`)
- numeric/date conversions and small utility functions

### Document subsystem
Main files and directories:
- `documents.php`
- `docsIncludes/`

Responsibilities inferred:
- upload, move, delete, and show documents
- document-pool style storage and linking
- OCR/extraction and metadata handling in some flows
- synchronization between DB records and filesystem artifacts

This subsystem likely has both database and filesystem side effects and should be documented together with storage layout and required external tools.

### Order and report support
Main files/directories:
- `ordrefunc.php`
- `orderFuncIncludes/`
- `kreditorOrderFuncIncludes/`
- `reportFunc/`
- `rapportfunc.php`
- `rapportfuncMSC.php`
- `grid.php`

Responsibilities inferred:
- shared order-processing helpers
- list/grid rendering
- report generation and common report logic
- helpers reused by debtor, creditor, finance, and related modules

## Runtime coupling and hidden dependencies
`includes/` appears heavily coupled to:
- globals set by request entry files
- session state
- the `online`, `settings`, `regnskab`, and `grupper` tables
- filesystem layout such as `temp/`
- deployment-time external binaries configured in settings
- include order within legacy request flows

Because of this, code in `includes/` should be treated as stateful and context-sensitive unless proven otherwise.

## Known documentation warnings
1. **`online.php` has side effects on include.** It reads and writes runtime state, updates DB values, and may exit or redirect.
2. **`db_query.php` changes control flow.** Query failures can log, mail, alert, and terminate a request.
3. **Document flows may require filesystem + shell tooling.** Document code appears to interact with both DB schema and OS-level tools.
4. **Translation helpers may be stateful.** Some fallback logic may read from CSV and write translations into the DB at runtime.
5. **Include order matters.** Many helpers depend on globals and prior bootstrap state rather than explicit function contracts.

## Safe maintenance guidance
- Prefer changes in module-local helpers before expanding central files unless the behavior is truly cross-cutting.
- Test changes in at least one login flow and one business module flow after editing `online.php`, `db_query.php`, or `std_func.php`.
- When changing document logic, verify both DB behavior and filesystem side effects.
- When changing query helpers, verify both PostgreSQL and MySQLi code paths if both are supported.
- Document any newly introduced globals, settings keys, or external binary requirements.

## Suggested future expansion for this doc
This draft should later be extended with:
- include dependency map
- key global variables table
- main helper function index
- document storage/layout description
- error/logging behavior reference
- external tool dependency matrix
