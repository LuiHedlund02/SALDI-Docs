# `rental/` module

Status: draft
Audience: maintainers, booking/rental developers, support

## Purpose
`rental/` is SALDI’s rental administration module. It manages rentable items, bookings, reservations/spærringer, closed days, customer history, remote booking settings, and order/payment/mail integration.

The module uses a shared PHP backend plus page-specific JavaScript UIs.

## Main entry points
- `rental.php` — central backend and API surface
- `api/api.js` — shared frontend wrapper for rental endpoints
- `index.php` / `index.js` — main availability/calendar overview
- `booking.php` / `booking.js` — create booking flow
- `edit.php` / `edit.js` — edit booking/item state
- `items.php` / `items.js` — product/item overview
- `lookupcust.php` / `lookupcust.js` — customer booking history
- `daysoff.php` / `daysoff.js` — closed-day management
- `settings.php` / `settings.js` — module settings
- `remote.php` / `remote.js` — remote booking, mail, and payment settings
- `upload.php` — CSV import helper
- `header.php` — shared navigation/header

## Core concepts
Observed concepts include:
- rentable items/assets grouped under products
- booking periods
- reservations/spærringer
- closed days
- remote/public booking configuration
- customer history lookup
- optional linked sales orders/invoice-side flows

## Data model
`sqlCreation.php` appears to ensure the module schema exists and contains expected columns.

Important tables include:
- `rentalitems`
- `rentalperiod`
- `rentalreserved`
- `rentalclosed`
- `rentalsettings`
- `rentalmail`
- `rentalpayment`
- `rentalremote` / `rentalremoteperiods`
- `betalingslink`

This suggests the module is self-contained in concept, but still coupled to the wider ERP for customers, products, and orders.

## Backend and API model
### `rental.php`
Observed responsibilities include:
- backend request handling for bookings, items, reservations, settings, remote config, and customer lookup
- cleanup of expired external bookings
- ensuring missing remote-booking rows exist where needed
- creating/updating linked orders in the wider ERP flow

### `api/api.js`
Observed responsibilities include:
- wrapping all front-end calls to the rental backend
- exposing methods for bookings, items, reservations, closed days, settings, remote link/mail/payment config, and order creation/update

For frontend understanding, `api/api.js` is a practical source of truth for what the UI expects the backend to do.

## Main workflows
### Availability and bookings
- the calendar/overview UI calculates availability from bookings, reservations, and closed days
- booking creation selects a customer, item/product, and date range
- edit flows allow changes to booking dates/items and linked order behavior

### Items/assets
- products can have one or more rentable item records
- item maintenance includes create/update/delete style operations under product groupings

### Reservations and closed days
- reservations/spærringer seem to act as soft blocks/warnings
- closed days are calendar-level exclusions that influence availability and booking choices

### Remote booking, mail, and payment
- remote/public booking behavior is configured through `remote.php` and related settings tables
- payment and mail settings are stored inside rental-specific config tables
- remote/public booking appears tightly linked to payment status and confirmation flows

## Linked order behavior
`CreateOrder()` and `updateOrder()` in the backend indicate that rental bookings can create or update linked sales orders.

This means rental is not just a calendar system — it also affects the ERP order flow.

## Critical risks to document
1. **Availability-calculation risk** — bookings, reservations, and closed days must stay consistent.
2. **Linked-order risk** — rental changes can create or modify sales orders.
3. **Remote-booking coupling risk** — payment, booking state, and confirmation behavior depend on multiple moving parts.
4. **Schema-bootstrap risk** — runtime schema creation/repair may complicate upgrades and debugging.
5. **Item/product model risk** — confusion between products and rentable stand-items can create bad booking behavior.

## Safe maintenance guidance
- Start with `rental.php` and `api/api.js` to understand backend/frontend behavior together.
- Test booking creation, editing, reservation handling, and closed-day behavior together after changes.
- Verify linked order behavior whenever changing booking lifecycle logic.
- Be careful with runtime schema helpers and document new columns/tables immediately.

## Suggested future expansion for this doc
This draft should later be extended with:
- rental data-model diagram
- booking lifecycle diagram
- availability-calculation rules
- remote booking/payment flow notes
- order-linkage rules
- smoke-test checklist for rental changes
