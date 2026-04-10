# Release smoke sheet

Audience: release owners, ops

Use this as the short go/no-go checklist immediately after a deploy.

## Always run
- [ ] Login works
- [ ] Logout works
- [ ] App does not redirect to installer unexpectedly
- [ ] `temp/` and `logolib/` are writable
- [ ] One shared-runtime page loads
- [ ] One report or finance page loads

## If shared platform changed
- [ ] `includes/online.php` path still works
- [ ] iframe navigation in `index/main.php` still works
- [ ] one admin page still loads
- [ ] CSP/login page scripts still work

## If documents / PDF / reports changed
- [ ] one document preview works
- [ ] one upload/attach flow works
- [ ] one report page still loads
- [ ] expected binaries resolve (`ps2pdf`, `weasyprint`, `pdftk`, `convert` if relevant)

## If business modules changed
- [ ] Debitor/Kreditor: one card + one order/payment path work
- [ ] Finans: one journal/report path works
- [ ] Lager: one list/card/receiving path works
- [ ] Rental/Booking: one booking path works
- [ ] Systemdata: one settings/user/year path works

## If integrations changed
- [ ] REST API login/auth works
- [ ] API v2 auth works if used
- [ ] SOAP/legacy integration still works if affected
- [ ] Remote booking still initializes if affected
- [ ] integration logs/errors are visible

## If ops/config changed
- [ ] `includes/connect.php` is correct
- [ ] admin tool paths are valid
- [ ] one backup-related path still works
- [ ] rollback owner and last good backup are known

## Stop and rollback if
- login/bootstrap is broken
- accounting, stock, or booking state is corrupted
- backup/restore trust is lost
- production integrations cannot authenticate
