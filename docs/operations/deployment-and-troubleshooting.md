# Deployment and troubleshooting

Audience: ops, maintainers, senior developers

## Deployment model
SALDI is deployed as a source-as-webroot PHP application. There is no obvious CI/CD or bundling pipeline in the repository.

Operationally, this means:
- the source tree is part of the runtime deployment
- writable directories inside the tree matter
- external binaries and DB tooling are part of the runtime environment
- backup/restore and document flows often execute shell commands directly

## Required runtime directories
Important writable directories include:
- `temp/`
- `logolib/`
- `includes/` during install time

The `temp/` directory is especially important because it is used for:
- logs
- backup/restore scratch files
- generated archives/dumps
- integration logs in some subsystems

## Generated/config files
Important generated/runtime files include:
- `includes/connect.php`
- temp backup artifacts such as `*.sql`, `*.tar`, `*.tar.gz`, `*.sdat`
- retired/deleted-account backups under `nedlagte_regnskaber/`

A stale or incorrect `includes/connect.php` can also interfere with the installer and startup flow.

## External binaries and tool paths
The runtime/admin area expects or can configure paths for tools such as:
- `ps2pdf`
- `weasyprint`
- `pdftk`
- `ncftp`
- `pg_dump` / `mysqldump`
- `psql` / `mysql`
- `gzip`
- `gunzip`
- `tar`
- `curl` / `wget` in some integration paths

These tools are used across:
- backup/restore
- document/PDF handling
- integrations and sync jobs

## Backup and restore concerns
Current behavior suggests:
- backup/restore is shell-driven and path-sensitive
- restore may rebuild or replace DB state
- users may need to be logged out for restore flows
- DB admin rights are assumed for some operations
- artifacts are unpacked/written into `temp/` before import

Treat backup/restore as privileged operations and test them in a non-production environment first.

## Logging and temp usage
Examples of operational logging inferred from the repo:
- login/session logs in `temp/`
- REST/integration logs under `temp/$db/`
- invoice/mail related logs in `temp/$db/`
- Dandomain and other integration-specific logs in temp-like paths

When debugging, check:
- repo-root `temp/`
- `temp/$db/`
- any module-specific temp/log files referenced by the code path you are testing

## Common failure categories
### 1. Permissions and writable-path issues
Symptoms:
- install fails
- uploads/backups/previews fail
- logs are missing

Check:
- `temp/`, `logolib/`, and install-time write access to `includes/`

### 2. DB connection and privilege issues
Symptoms:
- install fails to create DB/schema
- backup/restore fails
- session/account switching fails

Check:
- DB host/user/password
- DB admin privileges
- PostgreSQL `pg_hba.conf` / trust settings if relevant

### 3. Missing external binaries
Symptoms:
- PDF generation fails
- backup/restore fails
- archive creation/extraction fails

Check:
- admin tool-path settings
- actual binary presence on host
- shell user permissions

### 4. Temp/log scratch-space issues
Symptoms:
- integrations fail silently
- backups/restores produce partial artifacts
- document previews or conversions fail

Check:
- free space
- directory existence
- per-db temp subdirectories and file ownership

### 5. Integration-specific failures
Symptoms:
- webshop sync failures
- SOAP or REST auth failures
- invoice PDF/mail failures

Check:
- module-specific logs in `temp/`
- auth configuration and tenant/account context
- external service reachability

## Safe maintenance guidance
- Treat shell-executing code as production-sensitive.
- Validate path assumptions before changing backup/restore/document code.
- After changes in install/admin/runtime code, test:
  - fresh login
  - one business module
  - one backup/restore-adjacent flow
  - one integration/logging path
- Keep docs updated when adding a new binary dependency, writable directory, or generated file.

## Suggested operational checklist
Before declaring an environment healthy:
- login works
- `includes/connect.php` is correct
- `temp/` and `logolib/` are writable
- admin settings show valid tool paths
- one backup flow works
- one PDF/document flow works
- one API/integration log path is verified

## Rollback triggers for deploy/config incidents
Consider immediate rollback or environment isolation when:
- login or bootstrap fails across multiple accounts
- document/PDF generation is down for invoicing-critical flows
- backup/restore artifacts stop being trustworthy
- API or remote-booking auth breaks for production callers
- temp/log path failures prevent diagnosis of a production issue
