# Upgrades and releases

Status: draft
Audience: ops, maintainers, release owners

This page provides a practical release/upgrade checklist for the current SALDI deployment model.

## Important context
SALDI is a source-as-webroot legacy PHP application with:
- generated runtime config (`includes/connect.php`)
- writable runtime directories (`temp/`, `logolib/`)
- external binary dependencies
- cross-module coupling through shared platform code

A safe release is not just “copy new files.” It should include environment, runtime, and smoke-test checks.

## Before release
### 1. Review scope
Confirm whether the change touches any of:
- install or runtime config
- DB access/bootstrap (`includes/`)
- login/session/index flow
- backup/restore/admin behavior
- integrations/API auth
- document/PDF generation
- user-visible workflows in core modules

If yes, ensure the related docs were updated in the same change.

### 2. Confirm environment assumptions
Verify on the target host:
- `temp/` is writable
- `logolib/` is writable
- external binaries still exist at expected paths
- DB credentials/config in `includes/connect.php` are correct
- enough disk space exists for temp/log/backup artifacts

### 3. Prepare rollback material
Before deploying:
- take a fresh backup
- record current app revision/version if available
- capture current tool-path/config state if it changed recently
- confirm who can perform rollback if needed

## During release
### 1. Deploy cautiously
Because the app is source-as-webroot:
- avoid partial file copies if possible
- keep shared platform files (`includes/`, `index/`, `admin/`) consistent as a set
- be especially cautious when changing mirrored or duplicated frontend shell files

### 2. Watch high-risk areas
Pay extra attention when the release touches:
- `includes/online.php`
- `includes/db_query.php`
- `index/`
- `admin/`
- `docsIncludes/`
- API/integration auth flows

These have broad blast radius.

## Post-release smoke tests
Run at least these checks:

### Platform
- login works
- logout/session-expiry behavior is normal
- app does not incorrectly redirect to installer

### Core runtime
- one page using shared `includes/` loads correctly
- one report/accounting flow works
- one document upload/preview flow works if relevant to the release

### Business modules
Test at least one representative flow in the module(s) changed, for example:
- Debitor: open debtor card / order or invoice-related path
- Finans: open journal / report page
- Lager: open item list / item card
- Kreditor: open supplier/order page
- Rental/Sager/Systemdata: open the changed workflow directly

### Integrations
If touched by the release, verify at least one relevant path:
- REST API auth works
- legacy API key flow works if still used
- SOAP or webshop sync logs/errors look normal
- remote booking/payment flow still initializes if touched

### Ops
- one backup-related path still works
- temp/log files are still being written in expected locations
- external tool paths still resolve

## Upgrade-specific cautions
If the release includes schema or runtime-behavior changes:
- verify install/first-run assumptions still hold
- check for runtime schema mutation code that may behave differently after deploy
- verify old and newer UI paths where both exist
- confirm no stale cached temp artifacts are masking failures

## If rollback is needed
1. stop or reduce active usage if possible
2. restore previous code set
3. restore DB/data only if necessary and only with a validated procedure
4. re-run the core smoke tests
5. document the failure mode before attempting a new release

## Documentation rule
A release should be considered incomplete if code changes were shipped without updating:
- the relevant module/platform/integration doc
- `docs/README.md` or `docs/architecture/module-map.md` when navigation changed
- ops/runbook docs when setup, tool paths, or runtime expectations changed
