# Report and document workflows

Status: draft
Audience: maintainers, finance/document-flow developers

The `reportFunc/` and `docsIncludes/` areas look like helper folders, but they contain important runtime behavior and in some cases state-changing actions. Treat them as workflow code, not just UI fragments.

## `reportFunc/`
This area supports report rendering and report-entry flows, especially around accounting-style views.

Important files include:
- `accountChart.php`
- `showOpenPosts.php`
- `frontPageTopMenu.php`
- `frontPageOldMenu.php`

### Observed responsibilities
- rendering account-card / ledger style reports
- showing open-post aging and related account actions
- building report filter/navigation UIs
- connecting users from dashboard/report entry pages into deeper report flows

### Important warning
Some report URLs may have side effects.

For example, `accountChart.php` appears able to alter settlement/open-post state via GET-driven actions such as unaligning records. This means these report pages are not purely read-only views.

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

### Observed responsibilities
- browsing the document pool
- previewing PDFs/images/XML and similar file types
- moving files between pool and attached-document locations
- inserting `documents` rows and maintaining DB/filesystem consistency
- handling upload and rename/cleanup flows
- in some cases updating bookkeeping draft state alongside document attachment

## Why this area is high risk
`docsIncludes/` crosses several boundaries at once:
- **database writes**
- **filesystem operations**
- **shell-command execution**
- **external-service calls**
- **bookkeeping side effects**

Examples inferred from code/scouting:
- file operations using shell tools such as `mv`, `rm`, `cp`, `convert`, `mogrify`, `weasyprint`
- XML preview conversion via an external EasyUBL service
- `insertDoc.php` and related flows that can silently create/update `kassekladde` rows or fields

## Practical guidance
### For report code
- Do not assume report pages are safe to prefetch/cache as static views.
- Review GET parameters for mutating behavior before changing routing or links.
- Test open-post/account-card behavior with realistic data after changes.

### For document code
- Verify both DB state and filesystem state after every change.
- Check for orphaned files, stale `documents` rows, and broken previews.
- Be cautious with shell command construction and path handling.
- Test XML, PDF, and image preview paths separately.
- Confirm whether document attachment changes also affect bookkeeping drafts.

## Critical risks to document
1. **Mutating-report risk** — report pages may alter state through URL parameters.
2. **Silent bookkeeping updates** — document attachment flows may update accounting drafts unexpectedly.
3. **Filesystem/DB drift** — move/delete/upload operations can desynchronize file storage and DB records.
4. **External-preview dependency** — XML preview depends on an external service and cached temp output.
5. **Shell-command risk** — path handling and shell execution deserve careful review.

## Suggested future expansion for this doc
This draft should later be extended with:
- report action matrix
- document lifecycle diagram
- preview/rendering matrix by file type
- filesystem layout notes
- bookkeeping-side-effect checklist
