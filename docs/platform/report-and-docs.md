# Report and document workflows

Audience: maintainers, finance/document-flow developers

The `reportFunc/` and `docsIncludes/` areas look like helper folders, but they contain important runtime behavior and in some cases state-changing actions. Treat them as workflow code, not just UI fragments.

## Why this area is high risk
These folders cross several boundaries at once:
- database reads and writes
- filesystem operations
- shell-command execution
- external-service calls
- bookkeeping side effects

Examples visible in the code include:
- `accountChart.php` updating `openpost.udlignet` / `udlign_id` from request parameters
- document preview and conversion using `convert`, `mogrify`, `weasyprint`, and `pdftk`
- XML preview conversion through EasyUBL
- `insertDoc.php` / `updateCashDraft.php` touching bookkeeping-related fields as part of attachment flows

## `reportFunc/`
This area supports report rendering and report-entry flows, especially around accounting-style views.

Important files include:
- `accountChart.php`
- `showOpenPosts.php`
- `frontPageTopMenu.php`
- `frontPageOldMenu.php`

### Current responsibilities
- rendering account-card / ledger style reports
- showing open-post aging and related account actions
- building report filter/navigation UIs
- connecting users from dashboard/report entry pages into deeper report flows

### Mutating report actions
Some report URLs are not read-only.

Representative example from `accountChart.php`:
```php
if ($unAlign || $unAlignId) {
    $qtxt = "update openpost set udlignet='0',udlign_id='0' where konto_id = '$unAlignAccount'";
    db_modify($qtxt,__FILE__ . " linje " . __LINE__);
}
```

That makes `accountChart.php` both a report page and an open-post mutation surface.

### Report action matrix
| Area | Main file | Reads | Mutates | Main risk |
|---|---|---|---|---|
| Account card / ledger | `reportFunc/accountChart.php` | `openpost`, account/report data | yes, unalign/open-post reset actions | URL-driven bookkeeping mutation |
| Open posts | `reportFunc/showOpenPosts.php` | `openpost` and debtor/creditor joins | mostly read-focused | date/filter assumptions can hide mismatches |
| Front-page report menus | `frontPageTopMenu.php`, `frontPageOldMenu.php` | navigation/filter state | low | stale links into risky report actions |

## `docsIncludes/`
This area manages document-pool browsing, uploads, previews, file moves, and attachment persistence.

Important files include:
- `docPool.php`
- `insertDoc.php`
- `uploadDoc.php`
- `showDoc.php`
- `listDocs.php`
- `moveDoc.php`
- `deleteDoc.php`
- `emailDoc.php`
- `convertOldDoc.php`
- `updateCashDraft.php`

### Current responsibilities
- browsing the document pool
- previewing PDFs, images, and XML documents
- moving files between pool and attached-document locations
- inserting `documents` rows and maintaining DB/filesystem consistency
- handling upload and rename/cleanup flows
- updating bookkeeping draft context alongside document attachment in some paths

## Document lifecycle
A practical lifecycle view for the current implementation is:
1. file lands in the pool or upload path
2. `docPool.php` enumerates it and may enrich metadata/logging
3. `insertDoc.php` or `uploadDoc.php` creates/links the `documents` row and moves the file
4. `listDocs.php` / `showDoc.php` render attachment lists and previews
5. `moveDoc.php`, `deleteDoc.php`, or `emailDoc.php` perform secondary workflow actions
6. bookkeeping-related helpers such as `updateCashDraft.php` may update accounting context around the attachment

## Preview and rendering matrix
| File type / output | Main path | Dependencies | Notes |
|---|---|---|---|
| PDF | `showDoc.php`, `docPool.php` | filesystem, browser PDF support | often direct preview |
| Images | `showDoc.php`, `docPool.php` | `convert` / `mogrify` | preview/resizing path |
| XML invoices | `docPool.php`, `showDoc.php` | EasyUBL API, temp cache | falls back when API key is missing |
| HTML → PDF | shared document/print helpers | `weasyprint` | depends on configured binary path |
| PS → PDF / merge | shared print helpers | `ps2pdf`, `pdftk` | common in document output flows |

## Filesystem and storage notes
Expect document flows to touch both DB and filesystem state.

Typical concerns:
- file rename/move from pool to attached-document location
- temp preview files under `temp/` or `temp/$db/`
- `.info`/metadata companions in the pool
- attachment rows in `documents`
- stale preview or cache files after failed conversion

When debugging, verify both the row-level DB state and the actual file path on disk.

## Bookkeeping side-effect checklist
When changing `insertDoc.php`, `updateCashDraft.php`, or adjacent flows, verify:
- attachment still appears on the intended source record
- no duplicate `documents` row is created
- no orphaned file remains in the pool after a successful move
- `kassekladde` or related draft state is updated only when intended
- preview still works after move/rename
- email/send operations still reference the correct final file

## Practical guidance
### For report code
- Do not assume report pages are safe to prefetch or cache as static views.
- Review GET parameters for mutating behavior before changing routing or links.
- Test open-post/account-card behavior with realistic data after changes.

### For document code
- Verify both DB state and filesystem state after every change.
- Check for orphaned files, stale `documents` rows, and broken previews.
- Be cautious with shell command construction and path handling.
- Test XML, PDF, and image preview paths separately.
- Confirm whether document attachment changes also affect bookkeeping drafts.

## Critical risks
1. **Mutating-report risk** — report pages can alter state through URL parameters.
2. **Silent bookkeeping updates** — document attachment flows can update accounting drafts unexpectedly.
3. **Filesystem/DB drift** — move/delete/upload operations can desynchronize file storage and DB records.
4. **External-preview dependency** — XML preview depends on EasyUBL and cached temp output.
5. **Shell-command risk** — path handling and shell execution deserve careful review.
