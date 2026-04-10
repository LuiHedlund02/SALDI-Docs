# `sager/` module

Status: draft
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
Observed in and around `sager.php`:
- list and filter cases
- create/edit case records
- view detailed case information
- manage job/opgave edits
- manage case contacts
- copy/order-related flows in some paths

This appears to be the main operational center of the module.

### Planning
Important files:
- `planlaeg.php`
- `planlaeg_sager.php`
- `planlaeg_opgaver.php`
- `planlaeg_beregning.php`
- `ajax_planlaeg.php`
- `ajax_planlaeg_sag.php`

Observed behavior:
- planning at case level
- planning at task/opgave level
- date/drag-drop/dynamic updates through AJAX helpers

### Payroll and time
Important files:
- `loen.php`
- `loenliste.php`
- `vis_loen.php`
- `loenIncludes/` such as `retLoen.php`, `afregning.php`, `visListe.php`, `rates.php`

Observed behavior:
- payroll/time list views and detailed views
- editing/settlement-like behavior
- helper includes for rendering and payroll logic

### Documents, offers, and templates
Important files:
- `tilbud.php`
- `template_list.php`
- `template_form.php`
- TinyMCE template/content assets

Observed behavior:
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

Observed behavior:
- case-related attachments and file views
- note-taking and case history support

### Customer and employee links
Important files:
- `ansatte.php`
- `kunder.php`
- `medarbejdermappe.php`

Observed behavior:
- connects cases to responsible employees and relevant customers
- suggests the module acts as an operational hub across internal and external stakeholders

## Architectural notes
- heavily procedural PHP
- UI and state appear driven by URL parameters, sessions, and shared includes
- uses its own asset stack and frontend conventions compared with some other modules
- likely one of the modules where global shell behavior and module-specific frontend behavior overlap

## Critical risks to document
1. **Workflow sprawl risk** — many different business processes live in one folder.
2. **Planning-state risk** — planning and AJAX updates may be easy to break with partial UI changes.
3. **Payroll sensitivity** — time/payroll flows are operationally sensitive and likely audit-relevant.
4. **Template/document coupling** — offer and template changes can affect output quality across many cases.
5. **Attachment/history fragmentation** — attachments, notes, and employee/customer links are spread across many files.

## Safe maintenance guidance
- Start in `sager.php` to understand the main case flow before touching subpages.
- Test planning, payroll, and document/template flows separately after changes.
- Treat module-specific JS/CSS as part of the runtime contract, not just presentation.
- Verify attachment and note flows when editing shared case logic.

## Suggested future expansion for this doc
This draft should later be extended with:
- case lifecycle description
- planning flow map
- payroll/time process notes
- document/template ownership map
- attachment/note data-flow summary
- smoke-test checklist for `sager/`
