# Remote booking flow

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

## Tenant selection
This flow is tenant-specific from the first request.

Current assumptions:
- public UI expects `?db=<tenant_db>` on `index.php`
- frontend JavaScript then calls `api.php?...&id=<tenant_db>`
- backend PHP reads the tenant from `$_GET["id"]` in `api.php`
- if the wrong tenant value is supplied, customer/order/booking state is created in the wrong tenant or fails to resolve expected settings

Treat the public `db` parameter and the backend `id` parameter as the same tenant identifier flowing through two layers of the implementation.

## Public booking lifecycle
A typical runtime path is:
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

## API surface in `api.php`
The API is action/query-string driven rather than a REST router.

Known actions include:
- `getAllProducts`
- `getAllDates`
- `getClosedDates`
- `createCust`
- `createBooking`
- `updateBooking`
- `getOrder`
- `getRentalReserved`

### `getAllProducts`
Purpose:
- returns active remote-booking products from `rentalremote`
- enriches them with product fields from `varer`
- includes period definitions from `rentalremoteperiods`

Response traits:
- includes remote-booking row data plus product name, number, unit, price, discount fields, and nested periods
- price is derived from `varer.salgspris * 1.25` in current code

### `getAllDates`
Purpose:
- returns booked and reserved periods for a product across all linked rental items

Response traits:
- combines `rentalperiod` rows and `rentalreserved` rows into one date-block list
- uses Unix timestamps for period fields in the raw booking data path

### `createCust`
Purpose:
- creates or reuses a debtor/customer in `adresser`
- normalizes email to lowercase
- if email already exists, reuses that customer and forwards the update to `createCust.php`
- otherwise finds the next free customer number in the `1000..9999` range

Example request body:
```json
{
  "name": "Jane Doe",
  "addr": "Examplevej 1",
  "zip": "2100",
  "city": "København Ø",
  "tlf": "+45 12345678",
  "email": "jane@example.test"
}
```

Example response:
```json
{ "id": 123 }
```

### `createBooking`
Purpose:
- creates a tentative `rentalperiod` row with an expiry time
- extends the end date across any configured closed days
- calls shared order creation logic to build the linked sales order

Important runtime behavior:
- request body uses date strings such as `start_date` and `end_date`
- server converts them to Unix timestamps
- expiry time is currently set to roughly 6 minutes (`time() + 360`)
- response returns the created booking ID

Example response:
```json
{ "id": 987 }
```

## Data assumptions
Tables used directly or indirectly here include:
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

## Payment and mail behavior
Current cautions:
- payment uses Vibrant helpers and polling, not a webhook-first model
- booking success depends on both order creation and payment status updates
- confirmation mail is sent after approval using PHPMailer and tenant-specific rows in `rentalmail`
- `sendMail.php` includes `new-email.php`, while some surrounding docs/code references `new-mail.php`; verify template naming before refactoring
- mail errors are written into `temp/$db/error-<timestamp>.json`

## Cleanup and failure handling
When a booking flow fails, verify all of these together:
- tentative `rentalperiod` rows
- linked `ordrer` / `ordrelinjer`
- payment status rows/settings
- customer creation side effects
- confirmation mail state

Because the flow is polling-based, timeouts and browser-close scenarios deserve explicit testing.

## Main risks
1. **booking/payment coupling risk** — booking success depends on both order creation and payment state.
2. **polling-only payment flow** — repeated polling can fail or timeout in ways a webhook flow would not.
3. **cross-module data risk** — rental, customer, order, and accounting assumptions all meet here.
4. **tenant-parameter risk** — public links depend on the correct `db` value.
5. **environment/config risk** — payment/mail configuration must be correct per tenant.

## Safe maintenance guidance
- test the full public flow, not just individual endpoints
- verify success, timeout, and rejection paths separately
- confirm both booking rows and linked orders are cleaned up correctly on failure
- verify mail template/config correctness after changes
- compare frontend and backend pricing behavior before shipping booking changes
