# `sager/` module

Audience: maintainers, operations/support developers

## Purpose
`sager/` is a broad work-management module. It combines case/job management, planning, payroll/time workflows, documents/templates, notes, attachments, and links to customers and employees.

This is one of the more UI-heavy and workflow-rich areas in the repository.

## Main entry points
- `sager.php` — main hub for case/job workflows
- `leftmenu.php` — navigation/switchboard behavior
- `planlaeg.php` — planning landing page
- `loen.php` — payroll/time entry point
- `tilbud.php` — offer/document-related entry point

## Main subareas

### Case/job management
Current behavior in and around `sager.php`:
- list and filter cases
- create/edit case records
- view detailed case information
- manage job/opgave edits
- manage case contacts
- copy/order-related flows in some paths

This is the main operational center of the module.

### Planning
Important files:
- `planlaeg.php`
- `planlaeg_sager.php`
- `planlaeg_opgaver.php`
- `planlaeg_beregning.php`
- `ajax_planlaeg.php`
- `ajax_planlaeg_sag.php`

Current behavior:
- planning at case level
- planning at task/opgave level
- date/drag-drop/dynamic updates through AJAX helpers

### Payroll and time
Important files:
- `loen.php`
- `loenliste.php`
- `vis_loen.php`
- `loenIncludes/` such as `retLoen.php`, `afregning.php`, `visListe.php`, `rates.php`

Current behavior:
- payroll/time list views and detailed views
- editing/settlement-like behavior
- helper includes for rendering and payroll logic

### Documents, offers, and templates
Important files:
- `tilbud.php`
- `template_list.php`
- `template_form.php`
- TinyMCE template/content assets

Current behavior:
- document and offer generation/editing
- reusable text/template management
- rich-text editing for case-related content

### Attachments and notes
Important files:
- `bilag_sager.php`
- `bilag_body_sager.php`
- `bilag_mappe.php`
- `bilag_ansatmappe.php`
- `view_bilag_sager.php`
- `notat.php`

Current behavior:
- case-related attachments and file views
- note-taking and case history support

### Customer and employee links
Important files:
- `ansatte.php`
- `kunder.php`
- `medarbejdermappe.php`

Current behavior:
- connects cases to responsible employees and relevant customers
- acts as an operational hub across internal and external stakeholders

## Architectural notes
- heavily procedural PHP
- UI and state are driven by URL parameters, sessions, and shared includes
- uses its own asset stack and frontend conventions compared with some other modules
- is one of the modules where global shell behavior and module-specific frontend behavior overlap

## Critical risks
1. **Workflow sprawl risk** — many different business processes live in one folder.
2. **Planning-state risk** — planning and AJAX updates may be easy to break with partial UI changes.
3. **Payroll sensitivity** — time/payroll flows are operationally sensitive and likely audit-relevant.
4. **Template/document coupling** — offer and template changes can affect output quality across many cases.
5. **Attachment/history fragmentation** — attachments, notes, and employee/customer links are spread across many files.

## Troubleshooting and verification
- Start in `sager.php` to understand the main case flow before touching subpages.
- If planning breaks, inspect the AJAX endpoints and drag/drop date-handling helpers together.
- If payroll/time output looks wrong, review `loen.php` plus `loenIncludes/` rather than only the list pages.
- After changes, test planning, payroll, and document/template flows separately.
- Verify attachment and note flows when editing shared case logic.
