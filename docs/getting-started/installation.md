# Installation

Audience: deployers, maintainers

This page documents the practical installation flow for SALDI based on the current repository and installer behavior.

## Deployment model
SALDI is deployed as a classic PHP application served directly from the source tree/webroot. There is no modern JS build pipeline visible in the repository.

A typical setup is:
- clone/copy the repo into the webroot
- create required writable directories
- ensure PHP + database server are available
- open the app in a browser and complete the installer

## Server requirements
Observed requirements from repo docs/code:
- Linux or other Unix-like host
- web server with PHP support
- PostgreSQL or MySQLi/MariaDB database
- PostgreSQL is the preferred DB in the existing docs

## Client/browser requirements
The installer and app assume a browser with:
- cookies enabled
- JavaScript enabled
- popup/new-window support enabled

Repository docs specifically mention Chrome/Chromium and Vivaldi as tested browsers.

## Required writable directories
Create these directories in the repo root before install:
- `temp/`
- `logolib/`

The installer/runtime also requires write access to:
- `includes/`
- `temp/`
- `logolib/`

Why:
- `includes/connect.php` is generated during install
- `temp/` is used for logs, temporary files, and backups
- `logolib/` stores uploaded logos/assets

## Generated files and runtime artifacts
Generated during install/runtime:
- `includes/connect.php` — created by the installer
- temp test files such as `temp/test.txt`
- backup/restore artifacts under `temp/`

Important note:
If `includes/connect.php` already exists, the app may treat the installation as already completed.

## Database support
SALDI supports:
- PostgreSQL
- MySQLi/MariaDB

Important note from the existing install docs:
- PostgreSQL restore/backup flows may require `trust` access for the DB admin user in `pg_hba.conf`
- backup/restore tooling assumes a DB admin with enough rights to create/drop databases and terminate sessions where needed

## External tools and paths
Installer/admin settings seed or rely on paths for tools such as:
- `ps2pdf`
- `weasyprint`
- `pdftk`
- `ncftp`
- `pg_dump` / `mysqldump`
- `gzip`
- `gunzip`
- `tar`

Not all are required for first install, but they matter for PDF generation, backup/restore, and related operations.

## Composer note
The repo contains a minimal Composer dependency under `includes/composer.json` (`phpmailer/phpmailer`).

Operationally, this means:
- Composer is not the main app bootstrap mechanism
- but mail/PDF-related flows may still expect the vendor autoloader to be present in the deployed environment
- verify where `vendor/` is installed in your deployment before enabling mail-related features

## Installation flow
1. Place the repository in the intended webroot path.
2. Create `temp/` and `logolib/`.
3. Ensure `includes/`, `temp/`, and `logolib/` are writable by the web server user.
4. Ensure the web server and DB server are running.
5. Open the application URL in the browser.
6. If `includes/connect.php` is missing, SALDI redirects to `index/install.php`.
7. In the installer, provide:
   - DB type
   - DB host
   - DB name
   - DB admin user/password
   - SALDI admin username/password
   - language / charset choices as applicable
8. Complete the installer so it creates the DB/schema and writes `includes/connect.php`.

## First login and first account creation
After installation:
1. return to the login page
2. log in using:
   - database/account name
   - SALDI administrator username
   - SALDI administrator password
3. enter the administration area
4. create the first real accounting/company account (`regnskab`) if required by the flow
5. select/show the created account and continue setup

Based on the legacy install text, the first-time flow may involve creating the initial database/admin and then creating the first usable account from the admin area.

## Common installation problems
- missing `temp/` or `logolib/`
- `includes/connect.php` not writable during install
- DB user lacks rights to create/drop/manage DB objects
- PostgreSQL restore/setup blocked by `pg_hba.conf`
- popup windows blocked by the browser
- missing external binaries discovered later in admin settings or backup/restore flows

## Safe post-install steps
After successful install:
- consider locking down permissions on `includes/connect.php`
- verify `temp/` remains writable
- review external tool paths in admin settings
- test login, one business module, and one backup/log flow before treating the environment as production-ready
