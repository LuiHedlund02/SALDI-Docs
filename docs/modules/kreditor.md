# `kreditor/` module

Status: draft
Audience: maintainers, accounting-support developers, integrators

## Purpose
`kreditor/` is SALDI’s supplier and accounts-payable domain. It handles supplier master data, purchase orders, goods receipt, posting, payment runs, open items, reporting, UBL import, and scan-based intake through Paperflow.

This is a business-critical module because it touches both supplier operations and accounting state.

## What belongs to this module
Observed responsibilities include:
- supplier cards and supplier lists
- purchase-order creation and editing
- goods receipt and receipt-side posting
- supplier payments and payment-list management
- open items and creditor ledger views
- supplier reports/statements
- UBL / OIOUBL import
- OCR/scan intake and document-to-order workflows

## Main entry points

### `kreditor.php`
Main supplier list / landing page.

Observed behavior:
- renders the main creditor overview and search/filter UI
- links into supplier cards
- uses a datagrid/list style interface
- acts as the main entry into creditor workflows

### `kreditorkort.php`
Main supplier card editor.

Observed behavior:
- edits supplier identity, address, bank, VAT/currency, payment terms, and contact data
- serves as master-data source for orders, payments, and reporting
- connects to related actions such as print, labels, account fusion, and ledger views

### `ordreliste.php`
Main purchase-order hub.

Observed behavior:
- lists supplier orders and status-based views
- supports order follow-up and supplier-focused filtering
- appears to be the practical start of the purchasing flow

### `ordre.php`
Main supplier order editor.

Observed behavior:
- creates and edits supplier orders
- manages order lines and item-related behavior
- coordinates receipt/posting-related actions
- is one of the most workflow-heavy files in the module

### `modtag.php`
Goods-receipt handler.

Observed behavior:
- records receipt of supplier-order lines
- updates stock and accounting-adjacent state
- closes the loop between purchasing and inventory/accounting impact

### `betalinger.php` / `betalingsliste.php`
Payment-run preparation and payment-list overview.

Observed behavior:
- builds and edits supplier payment lines
- supports review/printing/export of payment lists
- exposes list views for saved payment runs and their state

### `rapport.php`
Reporting dispatcher for the creditor area.

Observed behavior:
- provides open-item, ledger, balance, reminder, and statement-style outputs
- acts as the reporting hub for supplier/account-payable views

### `ublimport.php`
UBL/OIOUBL import entry point.

Observed behavior:
- imports supplier documents/files into the purchasing flow
- can create or enrich supplier/order data from imported material

## Helper architecture
Key helper folders:
- `orderIncludes/` — shared supplier-order logic such as open/closed order handling, line movement, splitting, selection, and account insertion
- `kredLstIncludes/` — list/top-line helpers for creditor views
- `scanAttachments/` — Paperflow-based intake, OCR extraction, front-page review, content display, creditor/order creation, and cleanup

A large part of the real workflow logic is split into these helper areas rather than living only in the entry pages.

## Typical business flow
A common end-to-end path appears to be:
1. Create or update supplier card
2. Create supplier order
3. Receive goods / update receipt state
4. Post supplier-side accounting/inventory impact
5. Register or prepare payment
6. Review via ledger/open-item/report views

Imported/scan-based flows may enter in the middle by creating an order from a supplier document.

## Critical risks to document
1. **Receipt/posting mismatch** — receiving can affect both stock and accounting state.
2. **Duplicate intake risk** — Paperflow or UBL imports may create partial or duplicate supplier/order data.
3. **Payment-run accuracy risk** — wrong open items or bank export logic can create payment mistakes.
4. **Supplier master-data drift** — bad supplier-card data propagates into orders, payments, and reports.
5. **Open-item/report inconsistency** — ledger, statement, and matching views must stay aligned with posting results.

## Safe maintenance guidance
- Treat `ordre.php`, `modtag.php`, `betalinger.php`, and import/scan flows as high-risk.
- Validate a full supplier flow after changes: supplier card → order → receipt → payment → report.
- Test both manual order entry and imported/OCR-based intake when changing shared purchasing logic.
- Keep helper-folder responsibilities intact where possible instead of adding more logic to already-large entry pages.

## Suggested future expansion for this doc
This draft should later be extended with:
- supplier data model overview
- receipt/posting sequence diagram
- payment-run/export format notes
- Paperflow intake flow map
- UBL import expectations and field mapping
- AP smoke-test checklist
