# `debitor/` module

Status: draft
Audience: maintainers, finance-support developers, integrators

## Purpose
`debitor/` is the customer and accounts-receivable domain inside SALDI. It covers more than customer records: it includes debtor master data, order and invoice flows, posting, payment handling, reminders/collections, CRM features, commission/MySale behavior, rental hooks, POS-related flows, and document/export paths.

## What belongs to this module
Observed responsibilities include:
- customer master data and debtor lists
- debtor history and commission views
- order and invoice generation
- posting/bookkeeping handoff
- payment collection and matching
- reminder (`rykker`) and collection (`inkasso`) flows
- CRM/sales-adjacent workflows
- invoice/output standards like UBL, OIOXML, Peppol, EasyUBL
- integrations to rental and POS features

## Main entry points

### `debitor.php`
Primary debtor list/dispatcher page.

Observed behavior:
- serves as the main entry to debtor list workflows
- routes by `valg` into debtor, history, commission, and rental-related views
- loads shared runtime/includes and grid rendering
- stores and updates user-specific list configuration in `grupper` records (`art='DLV'`)
- reads settings related to features such as MySale and rental visibility

### `debitorkort.php`
Main customer/debtor card editor.

Observed behavior:
- creates/updates customer master data in `adresser`-related flows
- handles contact, address, delivery, payment, account, and customer metadata
- updates per-user settings such as tracked `debitorId`
- includes anonymization path via `anonymize.php`
- integrates with order/history/navigation flows through `returside` and related parameters

### Other important UI pages
- `debitorvisning.php` configures list columns, widths, sorting, categories, and visibility options
- `debitor_historik.php` provides debtor history-oriented views
- `debitor_kommission.php` provides commission/MySale-related customer views

## Core workflow map
Typical high-level business path:
1. Create or update debtor/customer record
2. Create order or import order data
3. Post/book the order or invoice
4. Generate output or send via print/email/EDI
5. Register and match payments
6. Escalate to reminder/collection flow when needed

## Orders, invoices, and posting
Important files include:
- `csv2ordre.php`
- `bogfor.php`
- `genfakturer.php`
- `increaseInvoiceNumber.php`
- `fakturadato.php`

Observed responsibilities:
- importing order data from CSV
- posting invoices/orders into accounting-related flows
- generating recurring or repeated invoice behavior
- managing invoice numbering and invoice dates
- branching into downstream document/export/send behavior after posting

## Output, email, and EDI-related files
Important files include:
- `api.php`
- `easyUBL.php`
- `formprintfunc.php`
- `formularprint.php`
- `oioubl_dok.php`
- `oioxml_dok.php`
- `peppol.php`
- `ubl2ordre.php`

Observed responsibilities:
- generating invoice/order documents
- building external invoice payloads
- handling standards-based output such as UBL, OIOXML, Peppol, and EasyUBL
- supporting print/email/send workflows after posting

## Payments, matching, and collections
Important files include:
- `betalingsliste.php`
- `betalinger.php`
- `betalinger_settings.php`
- `udlign_openpost.php`
- `rykker.php`, `ny_rykker.php`, `rykkerprint.php`, `rykkertjek.php`, `afslut_rykker.php`
- `inkasso.php`
- `payments/` helpers such as `mobilepay.php`, `flatpay.php`, `vibrant.php`, `lane3000.php`, `pbs_import.php`

Observed responsibilities:
- payment list generation and payment settings
- receipt/payment matching against open items
- reminder and dunning flows
- collection escalation
- payment-provider and payment-import related hooks

## CRM, commission, rental, and POS extensions
### CRM
- `crmkalender.php`, `crmopret.php`, `crmvisning.php`
- indicates debtor data is also used in customer relationship and scheduling/sales workflows

### Commission / MySale
- `debitor_kommission.php`, `mailTxt.php`, `mySaleMailTxt.php`
- indicates special customer subsets and communication flows linked to MySale/commission behavior

### Rental hooks
- `rental.php` and `rentalIncludes/`
- indicates overlap between debtor records and rental reservations/items/timelines

### POS hooks
- `pos_ordre.php`, `pos_ordre_includes/`, `pos_print/`, `saftCashRegister*.php`
- indicates debtor records can feed sales/POS-related flows and cash register reporting

## Helper architecture
Key helper folders:
- `debLstIncludes/` — debtor list rendering and top-line/list behavior
- `orderIncludes/` — order-related helper logic
- `ordLstIncludes/` — order list support
- `func/` — smaller debtor-related helpers
- `pos_ordre_includes/` — POS-specific order support
- `rentalIncludes/` — rental-specific helper logic and local notes

## Data and dependency notes
This module appears coupled to:
- shared runtime in `includes/online.php`, `std_func.php`, and `grid.php`
- settings and `grupper` records for per-user view/config behavior
- accounting/posting behavior outside the folder boundary
- document generation and external delivery standards
- PSP/bank/payment integrations under `payments/`

## Critical business-process risks to document
1. **Invoice number integrity** — changes around posting/numbering can create duplicate or skipped invoice numbers.
2. **Accounting period and invoice-date correctness** — wrong posting or invoice dates can affect compliance and ledger accuracy.
3. **Payment matching errors** — incorrect open-post matching can distort balances and customer statements.
4. **Reminder and collection correctness** — wrong dunning/inkasso behavior can create legal and customer-service issues.
5. **EDI/document delivery failures** — malformed UBL/OIOXML/Peppol output can break customer or public-sector invoicing.

## Safe maintenance guidance
- Treat posting (`bogfor.php`) and numbering/date logic as high-risk; verify with realistic business scenarios.
- When editing debtor card fields, confirm downstream effects on order, invoice, payment, and export flows.
- Test both document generation and delivery paths after changes to invoice logic.
- Verify payment hooks carefully when touching `payments/` or open-post reconciliation logic.
- Preserve helper-folder boundaries where possible instead of adding more logic into already-large entry files.

## Suggested future expansion for this doc
This draft should later be extended with:
- debtor data model overview
- posting flow sequence diagram
- payment/reminder lifecycle
- EDI/export matrix by format
- key settings and feature flags
- test checklist for safe changes
