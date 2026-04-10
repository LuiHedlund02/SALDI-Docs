# Upgrades and releases

Audience: ops, maintainers, release owners

This page provides a practical release/upgrade checklist for the current SALDI deployment model.

## Important context
SALDI is a source-as-webroot legacy PHP application with:
- generated runtime config (`includes/connect.php`)
- writable runtime directories (`temp/`, `logolib/`)
- external binary dependencies
- cross-module coupling through shared platform code

A safe release is not just “copy new files.” It should include environment, runtime, and smoke-test checks.

## Release phases

### 1. Before release
#### Review scope
Confirm whether the change touches any of:
- install or runtime config
- DB access/bootstrap (`includes/`)
- login/session/index flow
- backup/restore/admin behavior
- integrations/API auth
- document/PDF generation
- user-visible workflows in core modules

If yes, ensure the related docs were updated in the same change.

#### Confirm environment assumptions
Verify on the target host:
- `temp/` is writable
- `logolib/` is writable
- external binaries still exist at expected paths
- DB credentials/config in `includes/connect.php` are correct
- enough disk space exists for temp/log/backup artifacts

#### Prepare rollback material
Before deploying:
- take a fresh backup
- record current app revision/version if available
- capture current tool-path/config state if it changed recently
- confirm who can perform rollback if needed
- note whether the release changes posting, stock, schema, or configuration data that cannot simply be undone by copying old PHP files back

### 2. During release
#### Deploy cautiously
Because the app is source-as-webroot:
- avoid partial file copies if possible
- keep shared platform files (`includes/`, `index/`, `admin/`) consistent as a set
- be especially cautious when changing mirrored or duplicated frontend shell files

#### Watch high-risk areas
Pay extra attention when the release touches:
- `includes/online.php`
- `includes/db_query.php`
- `index/`
- `admin/`
- `docsIncludes/`
- API/integration auth flows

These have broad blast radius.

### 3. Post-release smoke tests
Run at least these checks.

#### Platform
- login works
- logout/session-expiry behavior is normal
- app does not incorrectly redirect to installer

#### Core runtime
- one page using shared `includes/` loads correctly
- one report/accounting flow works
- one document upload/preview flow works if relevant to the release

#### Business modules
Test at least one representative flow in the module(s) changed, for example:
- Debitor: open debtor card / order or invoice-related path
- Finans: open journal / report page
- Lager: open item list / item card
- Kreditor: open supplier/order page
- Rental/Sager/Systemdata: open the changed workflow directly

#### Integrations
If touched by the release, verify at least one relevant path:
- REST API auth works
- API v2 key-auth flow works if used
- SOAP or webshop sync logs/errors look normal
- remote booking/payment flow still initializes if touched

#### Ops
- one backup-related path still works
- temp/log files are still being written in expected locations
- external tool paths still resolve

## Rollback model
Not every SALDI change rolls back cleanly by restoring older PHP files.

### Code-only rollback is usually sufficient for:
- template and shell display regressions
- non-destructive navigation issues
- documentation-only mistakes
- isolated frontend regressions

### Backup-first rollback is usually required or should be considered for:
- posting/accounting changes already run against live data
- stock movements, receipts, or corrections already applied
- runtime schema mutations or installer/upgrader changes
- restore/import failures that partially changed data
- payment/booking flows that created linked order and rental state
- admin changes that altered account/config/license state

## Suggested rollback procedure
1. stop or reduce active usage if possible
2. decide whether the issue is code-only or data-affecting
3. if code-only, restore the previous code set and rerun smoke tests
4. if data-affecting, restore only through a validated backup/restore procedure
5. re-run the core smoke tests plus the affected module/integration checks
6. document the failure mode before attempting a new release

## Rollback triggers
Treat rollback as the default option when any of these occur post-release:
- login/install bootstrap breaks for more than one account
- posting or open-item logic produces inconsistent results
- stock or booking state is duplicated/corrupted
- backup/restore stops producing trustworthy artifacts
- API auth is down for production integrations
- document/PDF flows fail in a way that blocks invoicing or daily work

## Documentation rule
A release should be considered incomplete if code changes were shipped without updating:
- the relevant module/platform/integration doc
- `docs/README.md` or `docs/architecture/module-map.md` when navigation changed
- ops/runbook docs when setup, tool paths, or runtime expectations changed

For a shorter release checklist, use [`release-smoke-sheet.md`](release-smoke-sheet.md).
