# Frontend assets

Audience: maintainers, frontend/full-stack developers

SALDI uses multiple frontend stacks in parallel. These should be treated as module-scoped runtime environments, not as one shared frontend bundle.

## Runtime truth
The real source of frontend behavior is the PHP shell/include chain, not historical docs.

Important runtime shells include:
- `includes/top_header.php`
- `includes/docsIncludes/docPool.php`
- `index/main.php`
- `sager/sager.php`
- `includes/top_header_sager.php`
- `includes/top_header_sager_small.php`
- `topmenu/header.php`
- `includes/topmenu/header.php`

Historical materials in `doc/` and `guides/` are useful reference, but they do not define current runtime assets.

## Shell/include matrix
| Area | Typical shell/header | Frontend traits | Risk |
|---|---|---|---|
| Main ERP shell | `index/main.php`, `includes/top_header.php` | broad shared JS/CSS, menu/header coupling | global blast radius |
| Document pool | `includes/docsIncludes/docPool.php` | more self-contained JS, file preview UI, local storage/session storage | preview/conversion regressions |
| Sager | `sager/sager.php`, `includes/top_header_sager*.php` | its own UI stack and layout assumptions | module-specific drift |
| Topmenu variants | `topmenu/header.php`, `includes/topmenu/header.php` | mirrored header behavior | copy drift between variants |

## Main frontend stacks

### Global shell
Used by many accounting, order, inventory, and admin pages.

Typical traits:
- older jQuery-based baseline
- shared menu/header/footer assets
- shared utility scripts and UI plugins
- broad blast radius if changed

### DocPool/documents stack
Used by document-related pages.

Typical traits:
- document-specific CSS/JS
- file preview state in browser storage
- AJAX-heavy listing and rename/update flows
- more isolated runtime assumptions than the global shell

### Sager stack
Used by `sager/` pages.

Typical traits:
- its own legacy jQuery/jQuery UI setup
- module-specific plugins and CSS
- tight coupling to sager-specific UI behavior

## Version coexistence
Multiple versions of the same libraries coexist in the repo, including:
- several jQuery versions
- several jQuery UI versions
- multiple flatpickr loading paths (local and CDN-based)

Do not assume that upgrading one file upgrades the application globally.

## CSS ownership map
The codebase has several overlapping CSS baselines, including:
- `css/standard.css`
- `css/std.css`
- `css/std2.css`
- `css/unified-components.css`

Practical rule:
- shared selectors can affect many old pages at once
- page-specific CSS often wins over “cleanup” attempts in common stylesheets
- load order and include path matter as much as selector names

## Mirrored and duplicated shells
Some header/menu files are effectively mirrored or lightly diverged copies. This creates version-drift risk: changing one shell may not update another.

Check for duplicate implementations before assuming a shell fix is global, especially around:
- top menu/header variants
- sager-specific top headers
- document-pool specific embedded scripts

## Safe maintenance checklist
1. Identify the page’s real include chain before changing assets.
2. Treat frontend changes as module-scoped unless proven global.
3. Avoid introducing a second version of the same library on one page.
4. Prefer page/module-local CSS for new UI work when possible.
5. Compare shared CSS baselines before editing a common selector.
6. When changing a mirrored shell/header, search for duplicate copies first.
7. After shared frontend edits, smoke-test at least one page in finance, debtor, inventory, admin, and sager/docPool if relevant.

## Critical risks
- mixed library versions on the same or related pages
- global-selector blast radius in shared CSS
- duplicate shell/header drift
- assuming historical docs describe current runtime behavior
- underestimating module-specific frontend stacks
