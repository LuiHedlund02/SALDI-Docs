# Verification checklists

Audience: maintainers, release owners, reviewers

Use these checklists after risky changes, deployments, restores, or environment moves.

## 1. Platform baseline checklist
- [ ] Login works
- [ ] Logout works
- [ ] Session-expiry behavior is sane
- [ ] App does not unexpectedly redirect to installer
- [ ] `temp/` is writable
- [ ] `logolib/` is writable
- [ ] New logs can be written under `temp/` or `temp/$db/`
- [ ] `includes/connect.php` is non-empty and matches the active environment

If this fails, inspect:
- `includes/connect.php`
- `index/index.php`
- `index/login.php`
- `includes/online.php`
- `temp/` permissions and recent temp/log files

## 2. Shared runtime checklist (`includes` / `index` / admin)
Run this when touching shared platform code.

- [ ] One page using `includes/online.php` loads correctly
- [ ] One page performing DB writes still works
- [ ] One admin/settings page loads
- [ ] One shell/iframe navigation path works
- [ ] CSP/script behavior is intact on login/index pages
- [ ] No obvious missing include / missing global errors appear

If this fails, inspect:
- `includes/db_query.php`
- `includes/std_func.php`
- `index/main.php`
- `admin/admin_panel.php`
- PHP/webserver error log plus `temp/$db/` logs

## 3. Document/report workflow checklist
Run this when touching `docsIncludes/`, `reportFunc/`, PDF, or file handling.

- [ ] One PDF preview works
- [ ] One image preview works
- [ ] One upload/attach flow works
- [ ] No orphaned document rows/files are created in a simple test
- [ ] One report page still loads correctly
- [ ] No accidental state-changing report action was triggered unintentionally

If this fails, inspect:
- `includes/docsIncludes/docPool.php`
- `includes/docsIncludes/insertDoc.php`
- `includes/docsIncludes/showDoc.php`
- `includes/docsIncludes/updateCashDraft.php`
- `includes/reportFunc/accountChart.php`
- configured paths for `convert`, `mogrify`, `weasyprint`, `pdftk`, `ps2pdf`

## 4. Finance/accounting checklist
Run this when touching `finans/`, posting, VAT, reports, or open-item logic.

- [ ] Journal page opens
- [ ] One simulation or safe posting-related path works as expected
- [ ] One relevant report still matches expected data
- [ ] Open-item related pages still load
- [ ] No obvious financial-year mismatch appears

If this fails, inspect:
- `finans/kassekladde.php`
- `finans/bogfor.php`
- `finans/genberegn.php`
- `finans/bankReconcile.php`
- `finans/rapport.php`
- active `RA` rows in `grupper`

## 5. Debitor/kreditor checklist
Run this when touching customer/supplier/order/payment flows.

- [ ] One debtor or supplier card opens and saves
- [ ] One order list and one order/detail page load
- [ ] One payment/open-item related page loads
- [ ] No numbering/date logic looks broken in the tested scenario
- [ ] Any import/intake path changed by the release is tested directly

If this fails, inspect:
- `debitor/debitorkort.php`, `debitor/bogfor.php`, `debitor/betalingsliste.php`, `debitor/rykker.php`
- `kreditor/kreditorkort.php`, `kreditor/ordre.php`, `kreditor/modtag.php`, `kreditor/betalinger.php`, `kreditor/ublimport.php`
- `includes/ordrefunc.php`
- `openpost` and related numbering/payment configuration

## 6. Inventory/rental/booking checklist
Run this when touching stock, rental, or booking flows.

- [ ] Item/product list loads
- [ ] Item card or receiving/counting/transfer page works if touched
- [ ] Rental availability/booking page loads if touched
- [ ] Remote booking/public flow initializes if touched
- [ ] No obvious stock/availability mismatch appears in a simple scenario

If this fails, inspect:
- `lager/lister/vareliste.php`, `lager/varekort.php`, `lager/modtagelse.php`, `lager/lagerflyt.php`, `lager/stockLog.php`
- `rental/rental.php`, `rental/api/api.js`, `rental/edit.php`, `rental/remote.php`, `rental/sqlCreation.php`
- `remoteBooking/api.php`, `remoteBooking/index.php`

## 7. Integration checklist
Run this when touching APIs, auth, sync, or external-service flows.

- [ ] REST API auth still works
- [ ] Relevant endpoint still returns expected shape
- [ ] Legacy API/SOAP path still works if affected
- [ ] Integration logs are written in expected temp/log locations
- [ ] External service errors are visible and diagnosable

If this fails, inspect:
- `restapi/core/BaseEndpoint.php`, `restapi/core/JWTAuth.php`
- `restapi/endpoints/v1/auth/login.php`, `restapi/endpoints/v1/auth/refresh.php`
- `api/v2/includes/AuthMiddleware.php`, `api/v2/includes/ApiKeyManager.php`
- `api/rest_api.php`, `soapserver/`, `soapklient/`
- `temp/$db/` integration logs or `error_log`

## 8. Backup/restore/environment checklist
Run this after host moves, config changes, or ops work.

- [ ] `includes/connect.php` is correct
- [ ] Tool paths in admin settings are valid
- [ ] One backup-adjacent path works
- [ ] Backup creates `temp/<db>_<timestamp>.sdat`
- [ ] Backup also leaves a persistent copy under `temp/backup/<db>/`
- [ ] `temp/$db/` can be created and written during runtime/restore work
- [ ] Restore inputs are limited to supported file types (`.sdat` / `.sql`)
- [ ] Temp scratch space is sufficient
- [ ] A restore test plan exists before any production restore is attempted

If this fails, inspect:
- `admin/admin_settings.php`
- `admin/backup.php`
- `admin/restore.php`
- shell access to `pg_dump` / `mysqldump`, `psql` / `mysql`, `gzip`, `gunzip`, `tar`
- free space and ownership under `temp/`, `temp/$db/`, and `temp/backup/<db>/`

## Practical rule
You do not need every checklist for every change. Choose the sections touched by the work. For high-risk releases, run at least:
- platform baseline
- one relevant business-module checklist
- integration checklist if applicable
- backup/restore/environment checklist when config or deploy behavior changed
