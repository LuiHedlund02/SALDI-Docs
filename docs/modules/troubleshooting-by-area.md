# Troubleshooting by area

Audience: support, maintainers, release owners

Use this page as the cross-module triage map when a problem spans more than one folder.

## Login, shell, or session failures
Inspect first:
- `index/index.php`
- `index/login.php`
- `index/main.php`
- `includes/online.php`
- `includes/connect.php`

Typical symptoms:
- installer redirect loop
- pages loading outside the shell incorrectly
- missing language or session context

## Posting, balances, or open-post mismatches
Inspect first:
- `finans/bogfor.php`
- `finans/genberegn.php`
- `includes/rapportfunc.php`
- `debitor/betalingsliste.php`
- `kreditor/betalinger.php`

Typical symptoms:
- wrong open items
- posted journals not matching reports
- mismatched debtor/creditor balances

## Stock, receiving, or FIFO discrepancies
Inspect first:
- `lager/varekort.php`
- `lager/modtagelse.php`
- `lager/lagerflyt.php`
- `lager/stockLog.php`
- `lager/varespor.php`

Typical symptoms:
- wrong available quantity
- transfers not reflected correctly
- receiving duplicated or only partially applied

## Rental or remote booking failures
Inspect first:
- `rental/rental.php`
- `rental/api/api.js`
- `remoteBooking/api.php`
- `remoteBooking/index.php`
- `remoteBooking/sendMail.php`

Typical symptoms:
- blocked dates wrong
- booking created without correct order/payment state
- remote booking page loads but payment or mail fails

## Supplier intake or payment-run issues
Inspect first:
- `kreditor/ordre.php`
- `kreditor/modtag.php`
- `kreditor/betalinger.php`
- `kreditor/ublimport.php`
- `kreditor/scanAttachments/`

Typical symptoms:
- duplicate supplier documents
- receipt not updating inventory/accounting as expected
- payment runs containing wrong or stale open items

## Configuration, rights, or fiscal-year problems
Inspect first:
- `systemdata/syssetup.php`
- `systemdata/diverse.php`
- `systemdata/regnskabsaar.php`
- `systemdata/kontoplan.php`
- `systemdata/brugere.php`

Typical symptoms:
- user suddenly loses access
- wrong active accounting year
- downstream modules behaving differently after setup changes

## Backup, restore, or environment breakage
Inspect first:
- `admin/admin_settings.php`
- `admin/backup.php`
- `admin/restore.php`
- `docs/operations/configuration-runbook.md`
- `docs/operations/backup-restore-runbook.md`

Typical symptoms:
- dump/import tools fail
- temp artifacts missing
- environment works until a restore/deploy, then breaks broadly

## Document, PDF, or attachment problems
Inspect first:
- `includes/docsIncludes/docPool.php`
- `includes/docsIncludes/insertDoc.php`
- `includes/docsIncludes/showDoc.php`
- `includes/udskriv.php`
- `docs/platform/report-and-docs.md`

Typical symptoms:
- preview missing
- attachment row exists but file missing, or the reverse
- PDF rendering failing after host/tool-path changes
