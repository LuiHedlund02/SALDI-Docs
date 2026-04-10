# Configuration Runbook

Status: draft
Audience: ops, maintainers

Use this checklist when changing environment-specific configuration after install, restore, or server moves.

## Writable directories
SALDI is deployed as a source-as-webroot PHP app, so filesystem permissions matter.

Must be writable:
- `temp/` — runtime scratch space, logs, backup/restore artifacts, generated PDFs, imports
- `logolib/` — company logos and PDF background files
- `includes/` — during install only, because the installer writes `includes/connect.php`

Common runtime pattern:
- the app creates `temp/$db/` on demand for per-account files
- if `temp/` is not writable, many imports, document flows, and restore paths fail

## Generated/runtime files
Key generated files and artifacts:
- `includes/connect.php` — written by the installer; environment-specific and required by almost every entry point
- `temp/$db/*.log` — module-specific logs
- `temp/$db/*.ps`, `*.pdf`, `restore.gz`, `restore.sql` — temporary document/restore output
- archive/dump files such as `*.tar`, `*.tar.gz`, `*.sdat`
- deleted-account backups may be stored under `nedlagte_regnskaber/`

If `includes/connect.php` is missing, the app redirects into the installer flow.

## DB and tool-path assumptions
Database connection settings live in `includes/connect.php` and are generated during install.
They define the active DB host, user, password, database name, and DB type.

Operational tool paths are stored in the `settings` table and managed from `admin/admin_settings.php`.
Common entries:
- `ps2pdf`
- `weasyprint`
- `pdftk`
- `ftp` (usually `ncftp`)
- `dbdump` (`pg_dump` for PostgreSQL, `mysqldump` for MySQL/MariaDB)
- `zip` (`gzip`)
- `unzip` (`gunzip`)
- `tar`

Defaults are typically `/usr/bin/...` or `/bin/...`, but the app can use custom host paths.
Login also reads several of these settings into session state, so path changes are often picked up on the next login.

## `connect.php` rules
`includes/connect.php` is the bootstrap point for DB access.

Important behavior:
- installer writes it after successful setup
- many pages assume it exists and include it directly
- a stale or wrong file can break login, admin, backup/restore, and API flows
- if DB credentials or DB type change, regenerate or update this file before testing

Treat it as deployment-specific, not as a normal source file.

## Temp and log behavior
Typical repo behavior:
- logs are written under `temp/` and often under `temp/$db/`
- document generation writes PS/PDF output there before sending or merging
- restore flows unpack into `temp/$db/` before importing
- order/integration flows may leave trace logs such as `shop.log` or `.ht_invoice_pdf.log`

When troubleshooting config problems, check:
1. `temp/`
2. `temp/$db/`
3. any module-specific path mentioned in the code path you are testing

## Verify after configuration changes
After editing DB credentials, tool paths, or runtime permissions, verify:
- fresh login works
- `includes/connect.php` points at the expected DB and the app no longer redirects to install
- `temp/`, `logolib/`, and any required subdirs are writable by the web server user
- admin settings show valid paths for `ps2pdf`, `weasyprint`, `pdftk`, `dbdump`, `zip`, `unzip`, `tar`, and `ftp`
- one PDF/document flow works and produces output in `temp/$db/`
- one backup/restore-adjacent flow works in the target environment
- one representative integration or import path writes its expected log file

If tool paths changed, log out and log back in before retesting so cached settings are refreshed.
