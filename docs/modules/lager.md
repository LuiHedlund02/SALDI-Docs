# `lager/` module

Audience: maintainers, inventory-support developers, implementers

## Purpose
`lager/` is SALDI’s inventory and product-management module. It handles product master data, stock levels, receiving, counting, corrections, warehouse transfers, pricing, labels, status views, and inventory history/traceability.

Provenance:
- **Verified locally**: `lager/varer.php` loads in the audit runtime after login, and the legacy redirect path was verified/fixed during local smoke work
- **Code-inferred**: receiving, counting, transfers, and stock-side side effects below still come primarily from repo inspection

A notable architectural trait is that this module contains both:
- legacy top-level pages
- a newer list/navigation flow under `lager/lister/`

## What belongs to this module
Current responsibilities include:
- product and variant master data
- stock tracking by warehouse/location
- goods receiving and receiving lists
- stock counting and stock regulation
- warehouse transfers
- pricing and cost updates
- label printing
- stock reports and status monitoring
- stock history and movement traceability

## Main entry points

### `lister/vareliste.php`
Current main product-list entry point.

Current behavior:
- loads the newer list UI for products
- uses `topLineVarer.php` for navigation/header behavior
- builds a searchable product grid with clickable rows into `varekort.php`
- supports performance logging into `temp/$db/vareliste_performance.log`

This appears to be the best place to start when understanding the current inventory UI.

### `varekort.php`
Main product card editor.

Current behavior:
- edits product master data and stock-related settings
- includes both `productCardIncludes/` and `varekort_includes/`
- supports variants, pricing, VAT behavior, barcodes, supplier/shop-related fields, and stock-related settings
- performs runtime schema checks and `ALTER TABLE` operations for some columns if they do not exist

This is one of the most important files in the module and also one of the riskiest because it mixes UI, business logic, and runtime schema mutation.

### Legacy product entry
- `varer.php` still exists as an older entry point and forwards into inventory-related behavior.
- Legacy pages should be treated as active compatibility paths, not necessarily dead code.

## Product card and item master
Important files/directories:
- `varekort.php`
- `productCardIncludes/`
- `varekort_includes/`

Current responsibilities:
- product identity and description fields
- barcodes and aliases
- VAT and price handling
- variants and stock-item behavior
- supplier/shop-related data
- packaging/weight/volume fields
- advanced pricing calculation options

The product card appears to be a central place where master-data rules affect downstream stock, pricing, POS, and shop sync behavior.

## Receiving goods
Important files include:
- `modtageliste.php`
- `modtagelse.php`
- `orderapi.php`

Current responsibilities:
- receiving lists and receiving forms
- matching expected quantities from purchase-order-like flows against received quantities
- updating receiving records and warehouse state
- integrating receiving with order-related data

This area should be documented together as one receiving workflow rather than as isolated pages.

## Counting, corrections, and transfers
Important files include:
- `optalling.php`
- `lagerregulering.php`
- `lagerflyt.php`

Current responsibilities:
- stock counting
- direct stock corrections/regulation
- warehouse/stock transfers
- FIFO-aware transfer behavior that updates both batch tables and `lagerstatus`

Because transfers and corrections affect stock truth, these are high-risk maintenance areas.

## Status, alerts, and reporting
Important files include:
- `lagerstatus.php`
- `beholdningsliste.php`
- `minmaxstock.php`
- `lagerstatusmail.php`
- `rapport.php`

Current responsibilities:
- operational inventory status views
- min/max stock reporting
- status mail/alert flows
- inventory-oriented report output

Some pages appear operational, while others are more reporting/export focused. That distinction should be made explicit in future docs.

## Pricing and cost updates
Important files include:
- `pricelist.php`
- `opdater_kostpriser.php`

Current responsibilities:
- list-based pricing management
- cost-price update routines
- interaction with master-data pricing fields and possibly downstream sales/shop behavior

## Labels and printing
Important files include:
- `labelprint.php`
- `labelprint_includes/`

Current responsibilities:
- label layout selection
- printer-specific output behavior
- different renderer paths for old/new label handling and hardware-specific formats

## History and traceability
Important files include:
- `stockLog.php`
- `varespor.php`

Current responsibilities:
- stock movement history
- item traceability across receipts, transfers, and other stock-affecting events

These files are likely important for debugging stock discrepancies.

## Data and dependency notes
This module appears coupled to:
- shared runtime from `includes/online.php`, `std_func.php`, `grid.php`
- stock/status data such as `lagerstatus`
- batch-tracking tables such as `batch_kob` and `batch_salg`
- product master data in `varer`
- purchase/receiving flows and possibly shop/POS behaviors
- a mixed old/new UI architecture

## Critical inventory/process risks to document
1. **FIFO transfer risk** — transfer logic can alter batch allocation in unexpected ways if assumptions are wrong.
2. **Runtime schema mutation risk** — `varekort.php` performs schema changes at runtime, which can complicate upgrades and deployments.
3. **Receiving duplication risk** — receiving/order integration can create duplicate or partial stock updates.
4. **Manual correction risk** — stock regulation/count flows can bypass expected audit checks if not verified carefully.
5. **Stale/inconsistent reporting risk** — stock/status views may diverge if updates are not atomic across related tables.

## Safe maintenance guidance
- Treat `varekort.php`, `modtagelse.php`, `lagerflyt.php`, and stock-regulation flows as high-risk areas.
- When editing product-card logic, verify downstream effects on variants, pricing, labels, and shop/POS-related behavior.
- Test receiving, transfer, and stock-report pages together after inventory-affecting changes.
- Be cautious with any code that performs `ALTER TABLE` at runtime; document new schema dependencies immediately.
- When changing batch or FIFO logic, verify both quantity totals and traceability/history outputs.

## Troubleshooting and verification
### If product-card or setup changes ripple unexpectedly
Inspect first:
- `varekort.php`
- `productCardIncludes/`
- `varekort_includes/`

Check for:
- runtime `ALTER TABLE` behavior creating schema drift
- product master-data changes affecting POS, shop sync, or pricing unexpectedly
- variant/barcode fields no longer lining up with downstream flows

### If receiving, transfer, or stock totals look wrong
Inspect first:
- `modtagelse.php`
- `modtageliste.php`
- `lagerflyt.php`
- `lagerregulering.php`
- `stockLog.php`
- `varespor.php`

Check for:
- duplicate receiving updates
- FIFO or batch allocation changing unexpectedly
- transfer logic updating `lagerstatus` without matching traceability rows

### Post-change verification
After inventory changes, verify at least:
- `lister/vareliste.php` still opens and links into `varekort.php`
- one product-card save works
- one receiving or transfer path updates stock as expected
- one stock log or traceability view still explains the resulting movement
