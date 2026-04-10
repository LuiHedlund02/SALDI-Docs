# Frontend assets

Status: draft
Audience: maintainers, frontend/full-stack developers

SALDI uses multiple frontend stacks in parallel. These should be treated as module-scoped runtime environments, not as one shared frontend bundle.

## Runtime truth
The real source of frontend behavior is the PHP shell/include chain, not historical docs.

Important runtime shells include:
- `includes/top_header.php`
- `includes/docsIncludes/docPool.php`
- `sager/sager.php`
- `includes/top_header_sager.php`
- `includes/top_header_sager_small.php`
- `topmenu/header.php`
- `includes/topmenu/header.php`

Historical materials in `doc/` and `guides/` are useful reference, but they do not define current runtime assets.

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
- newer jQuery than the broad legacy shell
- document-specific CSS/JS such as autocomplete and datepicker helpers
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

## CSS baselines
Several overlapping CSS baselines exist, including:
- `css/standard.css`
- `css/std.css`
- `css/std2.css`
- `css/unified-components.css`

Load order and page-specific include chains matter. A change to a shared selector can affect many pages unexpectedly.

## Mirrored or duplicated shells
Some header/menu files appear mirrored or duplicated, such as topmenu header variants. This creates version-drift risk: changing one shell may not update another.

## Safe maintenance guidance
1. Identify the page’s real include chain before changing assets.
2. Treat frontend changes as module-scoped unless proven global.
3. Avoid introducing a second version of the same library on one page.
4. Prefer page/module-local CSS for new UI work when possible.
5. Compare shared CSS baselines before editing a common selector.
6. When changing a mirrored shell/header, search for duplicate copies first.

## Critical risks to document
- mixed library versions on the same or related pages
- global-selector blast radius in shared CSS
- duplicate shell/header drift
- assuming historical docs describe current runtime behavior
- underestimating module-specific frontend stacks

## Suggested future expansion for this doc
This draft should later be extended with:
- shell/include matrix by module
- JS library version map
- CSS baseline ownership map
- safe-upgrade checklist for shared frontend dependencies
