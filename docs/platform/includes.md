# `includes/` platform layer

Audience: maintainers, backend developers, reviewers

## Purpose
`includes/` is the shared runtime layer for SALDI. It is not a small helper folder; it acts as the platform core used by many modules across the monolith.

Provenance:
- **Verified locally**: `online.php` participates directly in login/session bootstrap, reads/writes `online` state, and affects whether module entry pages load correctly in the audit runtime
- **Code-inferred**: most helper-surface descriptions, error-side-effect claims, and external-tool coupling details below still come from static inspection of the repository

This area provides:
- database access and query wrappers
- session and user/account bootstrap
- shared utility functions
- document and attachment handling
- order/report support code
- common UI fragments and legacy presentation helpers

## Why this folder matters
Changes here can affect multiple business modules at once. A small change in `includes/` can alter login behavior, DB access, reporting, document flows, translations, PDF generation, or UI rendering across the application.

## Request/bootstrap chain
A common runtime path is:
1. entry page includes `../includes/connect.php`
2. entry page includes `../includes/db_query.php` and/or `../includes/online.php`
3. `online.php` reloads runtime state from the `online` table using the active session ID
4. tool-path and localization settings are loaded from `settings`
5. large helper libraries such as `std_func.php`, `ordrefunc.php`, `grid.php`, or `documents.php` are layered on top

The result is a stateful bootstrap model where include order matters.

## Main entry files

### `online.php`
Primary runtime/bootstrap include used by many pages.

**Verified locally**: this include participates directly in request bootstrap, reads session-linked rows from `online`, updates `online.logtime`, and can block page loading when runtime context is wrong.

Current runtime responsibilities:
- reads runtime context from the `online` table using the active session ID
- loads account/user state such as database, rights, language, and current accounting year
- updates `online.logtime`
- reads settings-driven tool paths such as `ps2pdf`, `pdftk`, `ftp`, `dbdump`, `tar`, `zip`, and `weasyprint`
- participates in timeout/session-expiry handling
- can trigger upgrade/version checks when account DB version lags the application version

Operational note:
Treat `online.php` as part of the request lifecycle, not as a pure helper include.

### `db_query.php`
Shared database abstraction and error-handling layer.

**Code-inferred with partial local confirmation**: temp/log path behavior and query-side logging were exercised indirectly during local runtime debugging, but the broader failure/alert/mail paths remain code-inferred.

Current runtime responsibilities:
- connects to PostgreSQL or MySQLi depending on configured DB type
- exposes `db_connect()`, `db_select()`, `db_modify()`, `db_close()`, and related helpers
- applies `injecttjek()` before some modification queries
- logs SQL activity into temp files
- can alert, mail, log, and terminate execution on failure

Operational note:
This is not a thin DB wrapper. Query failures can change request flow.

### `std_func.php`
Large shared utility hub used broadly across the system.

**Code-inferred**: this section is based mainly on static inspection; local verification only confirmed that path/bootstrap behavior around `std_func.php` can affect many entry pages at once.

Common responsibilities:
- formatting and conversion helpers
- language/text lookup helpers
- date and decimal handling
- barcode and file helpers
- shared business utilities reused by multiple modules
- UI helper functions such as alert/notification-style output

## Key globals and settings loaded here
These variables are relied on broadly enough that they should be treated as part of the platform contract.

| Variable / setting | Typical source | Why it matters |
|---|---|---|
| `$db` | `online` / `connect.php` | selects the active tenant database |
| `$db_type` | `connect.php` | controls PostgreSQL vs MySQLi paths |
| `$s_id` | PHP session / `online` row | request/session identity |
| `$regnaar` | `online` / `grupper art='RA'` | active financial year |
| `$sprog_id` / `language_id` | `online`, cookies, users, settings | affects `findtekst()` and UI language |
| `$customAlertText` | `connect.php` / runtime | can change alert/error wording |
| `$ps2pdf`, `$pdftk`, `$weasyprint` | `settings` | PDF and document rendering |
| `$ftp`, `$dbdump`, `$tar`, `$zip`, `$unzip` | `settings` | backup/export/import flows |
| `$exec_path` | runtime config | shell-command prefix for tools like `convert`, `rm`, `mv` |

When introducing new globals or settings keys, document them immediately.

## Important helper surfaces

### Database helpers
- `db_connect()` — opens a DB connection for the configured engine
- `db_select()` — shared read/query wrapper
- `db_modify()` — shared write/query wrapper with error-path side effects
- `db_error()` — centralized DB error path

### Localization and UI helpers
- `findtekst()` / `findTxt.php` — translation lookup with runtime fallback behavior
- `activeLanguage()` — resolves and may persist the active language
- `alert()` — emits JS alert output
- `tekstboks()` — shared UI message-box helper

### Business helpers with broad reach
- `create_debtor()` / `createDebitor.php` — creates debtor records and is reused outside `debitor/`
- `ensureTableAndColumns.php` — runtime schema guard used in mutable legacy flows
- `fiscalYear.php` / `checkOpenFiscalYear.php` — financial-year guardrails used across finance-sensitive code

## Important subareas

### `stdFunc/`
A more split-out helper library inside `includes/`.

Notable pieces:
- `findTxt.php` — translation lookup and persistence behavior
- `fiscalYear.php` and `checkOpenFiscalYear.php` — financial-year validation
- `ensureTableAndColumns.php` — runtime schema creation/repair
- `createDebitor.php` — reusable debtor creation helper
- `navStack.php` — navigation-state support

### Document subsystem
Main files and directories:
- `documents.php`
- `docsIncludes/`

Responsibilities:
- upload, move, delete, and show documents
- document-pool style storage and linking
- XML/PDF/image preview handling
- synchronization between DB records and filesystem artifacts

This subsystem has both database and filesystem side effects and depends on shell tools plus optional external conversion services.

### Order and report support
Main files/directories:
- `ordrefunc.php`
- `orderFuncIncludes/`
- `kreditorOrderFuncIncludes/`
- `reportFunc/`
- `rapportfunc.php`
- `rapportfuncMSC.php`
- `grid.php`

Responsibilities:
- shared order-processing helpers
- list/grid rendering
- report generation and report-side mutators
- helpers reused by debtor, creditor, finance, and related modules

## External tool matrix
These tools are referenced from shared includes or settings and should be treated as runtime dependencies.

| Tool | Typical use |
|---|---|
| `ps2pdf` | PostScript to PDF conversion |
| `weasyprint` | HTML to PDF rendering |
| `pdftk` | PDF merging/background operations |
| `convert` / `mogrify` | image/PDF preview conversion |
| `mv`, `rm`, `cp` | document move/delete/copy flows |
| `pg_dump`, `mysqldump`, `psql`, `mysql` | backup and restore |
| `gzip`, `gunzip`, `tar` | archive creation/extraction |
| `ncftp` | FTP-style file delivery |

## Runtime coupling and hidden dependencies
`includes/` is heavily coupled to:
- globals set by request entry files
- session state and the `online` table
- `settings`, `regnskab`, and `grupper`
- filesystem layout such as `temp/`
- deployment-time external binaries configured in settings
- include order within legacy request flows

Because of this, code in `includes/` should be treated as stateful and context-sensitive unless proven otherwise.

## Known documentation warnings
1. **`online.php` has side effects on include.** It reads and writes runtime state, updates DB values, and may exit or redirect.
2. **`db_query.php` changes control flow.** Query failures can log, mail, alert, and terminate a request.
3. **Document flows require filesystem + shell tooling.** Document code interacts with both DB schema and OS-level tools.
4. **Translation helpers are not pure lookups.** Fallback logic may read from CSV and write translations into the DB at runtime.
5. **Include order matters.** Many helpers depend on globals and prior bootstrap state rather than explicit function contracts.

## Safe maintenance guidance
- Prefer changes in module-local helpers before expanding central files unless the behavior is truly cross-cutting.
- Test changes in at least one login flow and one business module flow after editing `online.php`, `db_query.php`, or `std_func.php`.
- When changing document logic, verify both DB behavior and filesystem side effects.
- When changing query helpers, verify both PostgreSQL and MySQLi code paths if both are supported.
- Document any newly introduced globals, settings keys, or external binary requirements.
