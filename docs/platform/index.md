# `index/` platform shell

Status: draft
Audience: maintainers, frontend/backend developers

The `index/` area provides the main application shell: login, dashboard routing, and iframe-based navigation.

## Purpose
This area is responsible for:
- login and first-entry behavior
- shell layout and top-level navigation
- iframe-based page loading
- session/cookie driven runtime state
- install redirect behavior when configuration is missing

It is not a modern SPA shell; it is a stateful legacy PHP shell that coordinates many module pages.

## Main entry points
- `index.php` — primary login/entry page
- `login.php` — login-related processing path
- `main.php` — shell/dashboard loading behavior
- `dashboard.php` — dashboard entry
- `menu.php` — menu/shell navigation

## Runtime model
Observed behavior includes:
- heavy use of PHP session globals such as authenticated user/account/year context
- cookie-based preservation of some user/session state
- CSP nonce generation for script safety in login/index flows
- redirect to install flow if `includes/connect.php` is missing
- iframe-based navigation where the outer shell and inner page content cooperate

Because many module pages assume the shell/session context already exists, changes here can affect the whole application.

## Key behavior notes
### Login and bootstrap
- login flow depends on shared includes and database state
- missing `connect.php` can redirect users into installation instead of normal login
- session expiry handling is part of the broader shell/runtime experience

### Iframe navigation
- navigation is not purely route-based; many pages are loaded inside an iframe/container flow
- parent/iframe coordination is part of the legacy UX model
- new pages may need to work both as direct loads and as iframe content

### CSP and script injection
- login/shell code includes CSP nonce handling
- any template changes should preserve nonce behavior for inline scripts that depend on it

## Key risks to document
1. **Session-global coupling** — changes to auth/session globals can break many modules at once.
2. **Iframe behavior assumptions** — some pages depend on being loaded under the shell rather than standalone.
3. **Install/bootstrap path confusion** — missing config sends users into install behavior.
4. **Cookie/state drift** — cookie changes can alter language, remember-me, or shell behavior across modules.
5. **Shell-wide blast radius** — small changes in `index/` can affect every module entry path.

## Safe maintenance guidance
- Test login, logout, session-expiry, and first-install/missing-config flows after shell changes.
- Preserve CSP nonce handling when editing templates or script injection.
- Verify both direct-load and iframe-loaded behavior for any new or changed page under the shell.
- Be cautious when renaming URLs or changing navigation assumptions; legacy links may still depend on current paths.

## Suggested future expansion for this doc
This draft should later be extended with:
- login/session flow diagram
- cookie/state summary
- iframe navigation map
- install redirect conditions
- shell smoke-test checklist
