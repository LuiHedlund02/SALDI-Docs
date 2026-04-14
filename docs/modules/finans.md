# `finans/` module

Audience: maintainers, finance-support developers, reviewers, implementers

## Purpose
`finans/` is SALDI’s financial core. It covers journal entry, posting, imports, bank reconciliation, budgets, VAT/tax reporting, financial statements, and statutory export flows such as SAF-T.

A key design detail is that many workflows in this module are driven by the active financial year stored in `grupper` with `art = 'RA'`.

Provenance:
- **Verified locally**: `finans/kladdeliste.php` and `finans/regnskab.php` load in the audit runtime after login
- **Code-inferred**: posting, VAT, reconciliation, and deeper accounting-side effects below were not fully executed end-to-end in the local verification pass

## What belongs to this module
Current responsibilities include:
- cash journal entry and editing
- posting and simulation of accounting entries
- open-item updates and balance recalculation
- bank import and bank reconciliation
- budget maintenance
- financial reports and VAT reports
- SAF-T/statutory export flows

## Main entry points

### `kassekladde.php`
Primary journal-entry UI.

Current behavior:
- renders the editable cash-journal grid
- supports drag/drop line ordering and duplicate-line behavior
- loads many lookup widgets for account, amount, debtor, creditor, employee, department, project, currency, and date
- supports document attachment and unsaved-change warnings
- uses a wide helper layer under `kassekladde_includes/`

This file appears to be the main user-facing page for preparing accounting entries before posting.

### `bogfor.php`
Main posting engine.

Current behavior:
- handles real posting and simulation modes
- blocks posting of journals already marked as posted
- uses the active financial-year period derived from `grupper.art = 'RA'`
- updates open items and VAT/difference-account behavior
- calls `genberegn($regnaar)` before/after posting-related work
- references `equalizeMatchingRecords()` to reconcile matching/open-post behavior

This is one of the highest-risk files in the module because it changes accounting state.

### `rapport.php`
Main report entry/dispatcher.

Current behavior:
- routes to multiple financial outputs such as regnskab, kontokort, momsangivelse, kontrolspor, provisionsrapport, listeangivelse, and SAF-T-related reporting
- works with filter/setup helpers in `rapport_includes/`

## Journal helper layer
Main helper folder:
- `kassekladde_includes/`

Current responsibilities from filenames:
- lookup/search helpers (`accountSearch.php`, `debitorLookup.php`, `creditorLookup.php`, `employeeLookup.php`, `departmentLookup.php`, `projectLookup.php`, `currencyLookup.php`, `amountSearch.php`)
- open-item helpers (`openpost_inc.php`, `insertOpenPost.php`)
- balance/date support (`dateBalance.php`)
- line management (`duplicate_line.php`, `drag-handle-ajax-call.php`)
- UX protection (`unsavedWarning.php`)
- invoice-related field updates (`updateInvoiceField.php`)

These helpers are part of the journal-entry workflow and should be documented as UI/process support, not as isolated utilities.

## Posting, recalculation, and open items
Important files include:
- `bogfor.php`
- `genberegn.php`
- `openpostdato.php`

Current responsibilities:
- turning journal lines into posted accounting entries
- handling simulation versus actual commit
- recalculating monthly totals in `kontoplan.md01..md12` from transaction data
- syncing open-item dates and matching behavior after posting

This area is likely where financial consistency is enforced.

## Import and reconciliation flows
Important files include:
- `importer.php`
- `bankimport.php`
- `bankReconcile.php`
- `rapport_includes/bankReconcile.php`

Current responsibilities:
- uploading and importing bank/journal data
- mapping imported columns into journal structure
- storing per-user import configuration in `grupper`
- comparing/matching bank statement data against posted Saldi entries

Because user-specific mappings are involved, two users may experience different import behavior for the same source format.

## Budgets
Important file:
- `budget.php`

Current responsibilities:
- monthly budget grid maintenance by account and period
- save/edit workflow for budget values
- autofill from prior-year actuals
- CSV export support

Budget logic depends on the currently active financial year configuration.

## Reporting and statutory outputs
Important files include:
- `rapport.php`
- `rapport_includes/forside.php`
- `rapport_includes/regnskab.php`
- `rapport_includes/kontokort.php`
- `rapport_includes/kontokort_moms.php`
- `rapport_includes/momsangivelse.php`
- `saft.php`
- `saftCreator.php`

Current responsibilities:
- financial statements and account-card style reports
- VAT-focused reports and declarations
- audit/control-track style outputs
- SAF-T export generation

VAT logic appears split between posting and reporting code, which should be documented explicitly.

## Data and dependency notes
This module appears coupled to:
- shared runtime from `includes/online.php`, `std_func.php`, `grid.php`
- financial-year configuration in `grupper` (`art = 'RA'`)
- open-item logic and matching helpers
- account and transaction tables used by recalculation/reporting
- importer mappings stored per user

## Critical accounting/process risks to document
1. **Double posting risk** — already-posted journals must be blocked consistently.
2. **VAT drift risk** — VAT logic is spread across posting and reports, so discrepancies are possible.
3. **Open-item desynchronization** — posting, date sync, and matching must remain aligned.
4. **Import mapping errors** — per-user mapping in `grupper` can misclassify imported transactions.
5. **Financial-year mismatch** — wrong `RA` configuration can place postings/reports in the wrong period.

## Safe maintenance guidance
- Treat `bogfor.php` and `genberegn.php` as high-risk accounting code; validate with realistic finance scenarios.
- Test both simulation and real-posting paths after changes to posting logic.
- When changing import logic, verify behavior with at least one existing user-specific mapping.
- When changing VAT logic, compare report output before and after on known sample data.
- After changes in posting/import/reconciliation flows, verify both balances and open-item dates.

## Troubleshooting and verification
### If posting fails or balances look wrong
Inspect first:
- `bogfor.php`
- `genberegn.php`
- `openpostdato.php`
- relevant `grupper` rows with `art = 'RA'`

Check for:
- journals already marked as posted
- wrong active financial year
- open-item dates or matching state drifting after posting
- VAT or difference-account behavior changing between posting and reports

### If import or bank reconciliation fails
Inspect first:
- `importer.php`
- `bankimport.php`
- `bankReconcile.php`
- `rapport_includes/bankReconcile.php`

Check for:
- user-specific import mappings in `grupper`
- changed column expectations in import files
- mismatches between statement data and posted entries

### Post-change verification
After finance changes, verify at least:
- `kassekladde.php` opens and saves normally
- one simulation path and one real posting path behave as expected in a safe test environment
- one report in `rapport.php` still matches expected figures
- open-item related pages still load and reflect the expected state
