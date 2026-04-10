# `booking/` module

Status: draft
Audience: maintainers

## Purpose
`booking/` is not a full booking subsystem. It is a very small legacy gateway page that exposes a simple rentable-item listing.

## Main entry point
- `booking/index.php` — the only functional entry point
- `booking/index.html` — placeholder only

## Observed behavior
When called with `?id=<regnskab_id>`, the script:
1. validates that an account ID is provided
2. resolves `regnskab.id` to the target database
3. checks whether the `booking` feature is licensed
4. reuses an existing `online` session for that database when possible
5. otherwise creates a synthetic temporary `booking` user/session
6. loads rentable items from `rentalitems` and maps them to `varer`
7. renders a simple HTML table with item ID, SKU, name, price, and unit

## Data dependencies
This page depends on:
- `regnskab`
- `online`
- `brugere`
- `rentalitems`
- `varer`
- shared includes such as `std_func.php`, `connect.php`, `license_func.php`, and `online.php`

## Scope and limitations
- appears read-only
- does not contain a full booking UI or reservation workflow
- looks more like a compatibility access page than a primary product surface
- is tightly coupled to the wider SALDI runtime and database schema

## Risks to document
1. **Synthetic-session behavior** — temporary `online` rows may be easy to overlook when debugging.
2. **License dependency** — the page will fail or redirect differently based on booking feature state.
3. **Narrow but implicit scope** — because it looks tiny, its runtime dependencies are easy to underestimate.

## Safe maintenance guidance
- Treat this folder as a compatibility layer, not a standalone product area.
- If booking behavior needs product work, inspect `rental/` and `remoteBooking/` as well before making assumptions.
- After changes, verify access with both an existing session and a synthetic booking session.
