# `systemdata/` module

Audience: maintainers, admins, implementers

## Purpose
`systemdata/` is SALDI’s administrative configuration and master-data hub. It contains the setup screens and utilities used to configure accounting structure, users/permissions, forms, POS menus, GDPR routines, import/export helpers, and many system-level settings.

This area is configuration-heavy and has broad cross-module impact.

## Main navigation
Common entry points:
- `top.php`
- `left_menu.php`

These files provide the navigation map into the systemdata workflows.

## Main subareas

### Master data setup
Key files:
- `syssetup.php`
- `syssetupIncludes/`

Current behavior:
- manages grouped system setup by `art`
- covers setup such as VAT, debtor/creditor groups, warehouses, item groups, discount groups, forms, and related master data
- relies heavily on `grupper`-based configuration

### Miscellaneous system settings
Key files:
- `diverse.php`
- `diverseIncludes/`
- `sys_div_func.php`

Current behavior:
- handles general-purpose configuration such as POS options, SMTP/mail setup, labels, API-related settings, import/export settings, and other operational toggles

### Fiscal year management
Key files:
- `regnskabsaar.php`
- `regnskabskort.php`
- `financialYearInc/`
- `fiscalYearInc/`

Current behavior:
- creates, edits, activates, and deletes fiscal years
- copies or migrates year-related setup
- updates year state for users and online sessions
- can delete empty fiscal-year data in some flows

### Chart of accounts
Key files:
- `kontoplan.php`
- `chartOfAccountIncludes/`

Current behavior:
- maintains the chart of accounts
- supports account-structure configuration and related helper logic
- appears tightly linked to fiscal-year behavior and currency/account settings

### Users and permissions
Key files:
- `brugere.php`
- `brugerdata.php`
- `brugereRevisor.php`

Current behavior:
- manages users and rights matrices
- supports accountant/revisor assignment
- includes permission-related settings such as read-only behavior, user links, and other account-access controls

### Forms and print layouts
Key files:
- `formularkort.php`
- `load_form_data.php`
- `save_form_data.php`
- `formularimport.php`

Current behavior:
- edits form definitions and form layout data
- supports load/save endpoints and import workflows for forms/templates
- affects print/document behavior outside this module

### POS menu configuration
Key files:
- `posmenuer.php`
- `posmenuer_includes/systemButtons.php`
- `sys_div_func_includes/posChoices.php`

Current behavior:
- configures POS menus and button definitions
- manages POS-specific runtime choices and supporting setup

### GDPR and cleanup
Key files:
- `gdpr.php`
- `gdprInc/`

Current behavior:
- finds and deletes eligible stale customer/vendor records
- performs privacy-related cleanup routines

### Import/export tools
Key files:
- `importer_*`
- `exporter_*`

Current behavior:
- provides CSV-based bulk movement of accounts, addresses, goods, variants, forms, POS menus, and similar master/config data

## Key data dependencies
This module appears strongly coupled to:
- `grupper`
- `kontoplan`
- `brugere`
- `adresser`
- `formularer`
- POS-related tables
- fiscal-year state

Configuration changes here can affect multiple business modules at once.

## Critical risks to document
1. **Rights misconfiguration risk** — user/permission changes can alter access across the whole application.
2. **Fiscal-year setup risk** — mistakes in year creation/activation affect reporting and posting elsewhere.
3. **Chart/master-data drift** — changes to shared groups/accounts can break downstream module assumptions.
4. **Form/POS config breakage** — layout or menu changes can disrupt daily operations quickly.
5. **Bulk import/export risk** — CSV utilities can overwrite or move large amounts of reference data.

## Safe maintenance guidance
- Treat fiscal-year, chart, user-rights, and import/export code as high-risk.
- Test at least one downstream workflow after changing shared setup areas.
- Document any new `grupper` codes, settings keys, or CSV formats immediately.
- Be cautious with cleanup/GDPR routines and confirm record-selection rules before changing them.

## Troubleshooting and verification
### If rights or access suddenly change
Inspect first:
- `brugere.php`
- `brugerdata.php`
- `brugereRevisor.php`
- related permission fields stored in users/rights tables

Check for:
- read-only or revisor-related flags changing unexpectedly
- account/user links not matching the intended environment
- downstream modules disappearing because rights or license-related assumptions shifted

### If fiscal-year or chart behavior is wrong
Inspect first:
- `regnskabsaar.php`
- `regnskabskort.php`
- `kontoplan.php`
- helper folders `financialYearInc/` and `fiscalYearInc/`

Check for:
- the wrong active year
- copied/deleted year setup leaving partial data behind
- chart changes no longer matching finance/debtor/creditor expectations

### If forms, POS, or bulk imports break
Inspect first:
- `formularkort.php`
- `load_form_data.php`
- `save_form_data.php`
- `posmenuer.php`
- `importer_*` / `exporter_*`

Check for:
- template/layout drift in printed documents
- POS menu changes not reflected in runtime use
- CSV imports overwriting more shared setup than intended

### Post-change verification
After systemdata changes, verify at least:
- one user/rights change behaves as expected
- one fiscal-year or chart page still loads and saves correctly
- one downstream module affected by the changed setup still works
- one form/POS/import path works if it was touched
