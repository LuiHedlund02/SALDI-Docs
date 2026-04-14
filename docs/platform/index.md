# `index/` platform shell

Audience: maintainers, frontend/backend developers

The `index/` area provides the main application shell: login, dashboard routing, and iframe-based navigation.

Provenance:
- **Verified locally**: installer redirect behavior, successful login, master-admin menu access, shell/dashboard loading, and in-shell loading of finance/debtor/inventory/rental/admin entry pages in the audit runtime
- **Code-inferred**: broader routing assumptions, legacy deep-link compatibility, and some shell/module coupling details still come primarily from repo inspection

## Purpose
This area is responsible for:
- login and first-entry behavior
- shell layout and top-level navigation
- iframe-based page loading
- session/cookie-driven runtime state
- install redirect behavior when configuration is missing

It is not a modern SPA shell; it is a stateful legacy PHP shell that coordinates many module pages.

## Main entry points
- `index.php` — primary login/entry page
- `login.php` — login processing path
- `main.php` — shell/dashboard loading behavior
- `dashboard.php` — dashboard entry
- `menu.php` — menu/shell navigation
- `logud.php` — logout path
- `install.php` — installer entry when configuration is missing

## Login and install flow
**Verified locally**: missing config returns users to the installer path, and successful login forwards into the shell.

A typical first-entry path is:
1. user opens `index/index.php`
2. `index.php` checks whether `../includes/connect.php` exists and is non-empty
3. if missing, users are redirected to `install.php`
4. otherwise login form state is prepared, CSP nonce is generated, and language/remember-me cookies are read
5. successful login creates or updates the `online` row and forwards into the shell

Important install guard:
```php
if (!file_exists("../includes/connect.php")) {
    print "<meta http-equiv=\"refresh\" content=\"2;url=install.php\">";
    exit;
}
```

## Session and cookie model
Current shell behavior relies on both PHP session state and cookies.

Important examples:
- `languageId` — persisted across the site and reused by login/dashboard pages
- `saldi_huskmig` — remember-me cookie carrying remembered login/account fragments
- `nonce` — CSP nonce stored in session and reused for inline scripts on login pages

Operationally this means:
- cookie path changes affect more than one page
- CSP-related edits must preserve nonce injection into inline scripts
- login/session regressions can surface as module-load failures later in the shell

## Shell runtime model
**Verified locally**: the outer shell in `main.php` can load dashboard, finance, debtor, inventory, rental, and backup/admin entry pages after login in the audit environment.

Typical shell behavior includes:
- authenticated session bootstrap through shared includes
- loading the outer shell in `main.php`
- loading module pages into an iframe/container
- preserving route state in the hash for navigation reloads
- coordinating parent/iframe updates through `update_iframe()`

The outer shell is responsible for top-level navigation, while many real module pages assume the shell/session context already exists.

## Iframe navigation model
**Code-inferred with partial local confirmation**: menu-driven iframe loading was verified locally for several core routes, but direct deep-link compatibility still depends on page-specific behavior.

The navigation model is legacy but consistent:
- menu entries call `update_iframe("/path/to/page.php")`
- the outer shell updates the iframe source and URL hash together
- dashboard/module pages can be reloaded by hash-based navigation
- some pages still need to behave sensibly when opened directly, not only inside the iframe shell

This means routing changes can break both menu clicks and direct deep links.

## Current routing examples from `main.php`
Representative shell targets include:
- finance: `/finans/kladdeliste.php`, `/finans/regnskab.php`, `/finans/rapport.php`
- debtor: `/debitor/ordreliste.php`, `/debitor/debitor.php`, `/debitor/rapport.php`
- rental: `/rental/index.php`, `/rental/settings.php`, `/rental/remote.php`
- creditor: `/kreditor/ordreliste.php`, `/kreditor/kreditor.php`, `/kreditor/rapport.php`
- inventory: `/lager/varer.php`, `/lager/modtageliste.php`, `/lager/rapport.php`
- system/admin: `/systemdata/kontoplan.php`, `/systemdata/syssetup.php`, `/admin/backup.php`

## CSP and inline-script handling
**Verified locally**: the login shell emits CSP nonce handling and still loads correctly in the audit runtime.

The login shell uses explicit CSP nonce handling.

Practical consequences:
- inline scripts on login/forgot-password pages require a valid nonce
- template cleanups must preserve nonce attributes on inline `<script>` blocks
- copying older snippets into the login shell can break under CSP even when they work elsewhere

## Main risks
1. **Session-global coupling** — changes to auth/session globals can break many modules at once.
2. **Iframe behavior assumptions** — some pages depend on being loaded under the shell rather than standalone.
3. **Install/bootstrap path confusion** — missing config sends users into installer behavior.
4. **Cookie/state drift** — cookie changes can alter language, remember-me, or shell behavior across modules.
5. **Shell-wide blast radius** — small changes in `index/` can affect every module entry path.

## Safe maintenance guidance
- Test login, logout, session-expiry, and first-install/missing-config flows after shell changes.
- Preserve CSP nonce handling when editing templates or script injection.
- Verify both direct-load and iframe-loaded behavior for any new or changed page under the shell.
- Be cautious when renaming URLs or changing hash/navigation assumptions; legacy links may still depend on current paths.
- After shell edits, smoke-test at least one page each from finance, debtor, inventory, rental, and admin.
