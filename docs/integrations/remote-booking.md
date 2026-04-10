# Remote booking flow

Status: draft
Audience: integrators, support, maintainers

`remoteBooking/` is the public-facing booking/payment flow that sits on top of SALDI rental and order logic.

## Purpose
This area provides:
- public product selection
- date/availability lookup
- customer capture
- booking creation
- payment-link flow
- payment polling
- confirmation mail after approval

It is closely coupled to `rental/`, `ordrer`, and payment/mail configuration.

## Main entry points
- `remoteBooking/index.php?db=...` — public booking UI entry
- `remoteBooking/index.js` — main frontend flow
- `remoteBooking/api.php` — backend JSON API
- `remoteBooking/createCust.php` — customer creation helper
- `remoteBooking/vibrantPaymentLink.php` — payment-link creation
- `remoteBooking/vibrantPaymentIntent.php` — payment status polling
- `remoteBooking/sendMail.php` — confirmation mail sender

## Public booking lifecycle
Observed high-level flow:
1. user opens the public booking page for a tenant DB
2. frontend loads products and available periods
3. user selects product and date range
4. customer/contact details are collected
5. backend creates or reuses customer data
6. backend creates a tentative rental booking and linked sales order
7. payment link is opened
8. frontend polls payment status repeatedly
9. on success, booking/order is finalized and confirmation mail is sent
10. on reject/timeout, booking/order is rolled back or cleaned up

## Backend API behavior
### `api.php`
Observed endpoints/actions include:
- `getAllProducts`
- `getAllDates`
- `getClosedDates`
- `createCust`
- `createBooking`
- `updateBooking`
- `getOrder`

Observed behavior includes:
- reading rental/public-booking product configuration
- calculating blocked dates from bookings/reservations/closed days
- creating tentative `rentalperiod` rows with expiry
- creating linked orders/order lines
- finalizing or deleting booking/order state based on payment outcome

## Data assumptions
Observed tables include:
- `rentalremote`
- `rentalremoteperiods`
- `rentalitems`
- `rentalperiod`
- `rentalreserved`
- `rentalclosed`
- `rentalsettings`
- `rentalpayment`
- `rentalmail`
- `adresser`
- `ordrer`
- `ordrelinjer`
- `varer`
- `grupper`
- `kontoplan`

This confirms that remote booking is not isolated; it depends on both rental configuration and ERP sales/accounting data.

## Payment and mail notes
Observed cautions:
- payment uses Vibrant helpers and polling, not a webhook-first model
- mail is sent after approval using PHPMailer-based logic
- there appears to be at least one suspicious template/include naming mismatch (`new-email.php` vs `new-mail.php`) that deserves verification
- frontend pricing should not be treated as the final source of truth; backend order creation logic is authoritative

## Main risks
1. **booking/payment coupling risk** — booking success depends on both order creation and payment state.
2. **polling-only payment flow** — repeated polling can fail or timeout in ways a webhook flow would not.
3. **cross-module data risk** — rental, customer, order, and accounting assumptions all meet here.
4. **doc gap risk** — API contract and schema expectations live mostly in code today.
5. **environment/config risk** — payment/mail configuration must be correct per tenant.

## Safe maintenance guidance
- test the full public flow, not just individual endpoints
- verify success, timeout, and rejection paths separately
- confirm both booking rows and linked orders are cleaned up correctly on failure
- verify mail template/config correctness after changes
- compare frontend and backend pricing behavior before shipping booking changes

## Suggested next expansion
This draft should later be extended with:
- endpoint-by-endpoint request/response notes
- payment-state transition notes
- booking/order cleanup rules
- tenant-config checklist for payment/mail/public booking
