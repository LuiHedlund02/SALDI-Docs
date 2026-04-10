# `produktion/` module

Status: draft
Audience: maintainers, inventory/finance-support developers

## Purpose
`produktion/` is a small but important module for production-order handling. It appears to bridge order workflow, stock awareness, and accounting-style reporting.

Even though the folder is small, changes here can affect production order status, stock visibility, and downstream financial/reporting behavior.

## Main entry points
- `ordre.php` — create/edit a production order and move it through its lifecycle
- `ordreliste.php` — list/filter production orders
- `rapport.php` — production-related reporting using account/report style output

## Main workflow
Observed production-order lifecycle:
1. draft/proposal
2. approved
3. ready for receive/delivery flow
4. posted/closed

Status transitions appear to be the main control mechanism in the module.

## Order handling
### `ordre.php`
Observed behavior includes:
- creating new production orders
- editing order header and lines
- protecting against duplicate submit/back-button issues
- handling status changes and copy/credit-note-like paths
- checking stock-related conditions before certain actions

This is the core business file in the module.

## List and overview
### `ordreliste.php`
Observed behavior includes:
- filtering orders by lifecycle stage
- using views such as proposals, active orders, and closed/posted orders
- sorting by order/date/customer/employee-like fields
- locking or coordinating edits when an order is already open elsewhere

## Stock and warehouse links
The module appears linked to inventory behavior through:
- stock checks on items/products
- references to `batch_kob` and `batch_salg`
- links into `../lager/varekort.php`
- hints of composite-item handling via `samlevare`

There is no obvious full BOM/stykliste engine in this folder, but production orders clearly depend on inventory state.

## Reporting
### `rapport.php`
Observed behavior:
- filters by fiscal year and account range
- provides outputs in a `kontokort` / `openpost` style
- supports production-related traceability through account/report views rather than manufacturing KPI dashboards

This means production reporting is tied closely to accounting/reporting conventions in the broader app.

## Critical risks to document
1. **Status-transition risk** — wrong status logic can allow posting/closing at the wrong time.
2. **Stock-consistency risk** — production changes may rely on current warehouse/batch state.
3. **Composite-item ambiguity** — indirect `samlevare` behavior may be easy to break if assumptions are wrong.
4. **Posting/report alignment risk** — production state and production-related accounting/report output must stay consistent.
5. **Small-module blind spot** — because the folder is small, it may look simple while still carrying high-impact cross-module logic.

## Safe maintenance guidance
- Start analysis in `ordre.php`; it appears to contain most real business logic.
- Test at least one full lifecycle path after changes: create → approve → receive/deliver → post/close.
- Verify both stock-facing and report-facing behavior after updates.
- Check linked inventory behavior in `lager/` when changing product/stock assumptions.

## Suggested future expansion for this doc
This draft should later be extended with:
- status-transition table
- production-order data-flow diagram
- stock/batch dependency notes
- reporting use-cases and expected outputs
- smoke-test checklist for production changes
