# Admin Platform

Audience: maintainers, ops, senior reviewers

The `admin/` area contains SALDI’s administrative control surface: account management, user administration, system settings, backups/restores, and licensing.

## Purpose
This area is used for privileged workflows that can affect:
- account metadata and availability
- database creation, deletion, and switching
- user and admin access
- backup and restore operations
- external-tool paths and operational settings
- license/feature availability

Because of that, `admin/` should be treated as one of the highest-risk folders in the repository.

## Main entry points

### Modern console
- `admin_panel.php` — primary admin dashboard and all-in-one admin console

### Legacy/admin workflow pages
- `vis_regnskaber.php` — account list/edit flow
- `aaben_regnskab.php` — switch/open active account context
- `opret.php` — create a new `regnskab` and database/schema
- `slet_regnskab.php` — delete/archive an account and drop the database
- `admin_brugere.php` — admin-user management
- `admin_settings.php` — system settings and executable paths
- `backup.php` — create backups
- `restore.php` — restore backups/import dumps
- `license_manager.php` — feature/license management

## Main workflows

### Account and user administration
Current capabilities include:
- creating and deleting accounts (`regnskaber`)
- editing account metadata and closure state
- switching the active account/database for a session
- managing admin users and permission strings
- setting login-related texts and feature flags

### Backup and restore
Current capabilities include:
- database dump creation
- archive creation/extraction
- restore of uploaded dump files
- migration-oriented restore behavior in some paths
- writing temporary backup artifacts into `temp/`

This workflow depends on external tools and filesystem permissions.

### Licensing and features
`license_manager.php` and related SQL/setup files manage feature availability for modules such as:
- booking
- lager
- kreditor

License state influences access outside the `admin/` folder as well.

### Settings and operational paths
`admin_settings.php` manages settings such as:
- `ps2pdf`
- `weasyprint`
- `pdftk`
- `ftp`
- `dbdump`
- `zip`
- `unzip`
- `tar`
- alert/news text and related operational settings

These are deployment-sensitive values and should be kept aligned with the server environment.

## Operational prerequisites
Before using destructive admin flows, verify:
- `temp/` is writable
- backup/restore binaries exist and are executable
- `includes/connect.php` points at the intended environment
- the operator has DB privileges required for dump/import/drop/create flows
- a fresh backup exists before restore or account-deletion work

## License-to-feature caution
The license layer is not isolated to the admin UI. If a feature disappears after admin or settings changes, confirm:
- license/feature rows in admin-related tables
- the module’s own access checks
- whether the current account is marked closed or feature-limited

## Key risks
The admin layer contains multiple security and operational hotspots:

1. **Remote requests with SSL verification disabled** — some cURL calls explicitly disable peer/host verification.
2. **Direct shell execution** — backup/restore and maintenance flows execute shell commands.
3. **Destructive filesystem commands** — commands such as `rm -rf` appear in backup/restore-related logic.
4. **Destructive DB operations** — account deletion includes `DROP DATABASE` and related cleanup.
5. **String-built SQL** — some queries are assembled dynamically in ways that deserve review.

## Smoke-test checklist after admin changes
After touching `admin/`, verify at least:
- admin dashboard loads
- settings page still saves and displays tool paths correctly
- one backup path works or at least reaches artifact creation
- restore UI still validates input correctly without running a destructive restore in production
- account switching/opening still works
- one licensed module remains accessible for a known-valid account

## Safe maintenance guidance
- Treat every admin entry point as privileged and potentially destructive.
- Review both SQL behavior and shell behavior before modifying admin pages.
- Prefer safer wrappers and parameterized DB calls where possible.
- Re-enable SSL verification for remote requests unless there is a clearly documented exception.
- Test backup/restore in a disposable environment before shipping changes.
- When changing account-creation or deletion flows, verify effects on `regnskab`, `kundedata`, temp files, and DB state together.
