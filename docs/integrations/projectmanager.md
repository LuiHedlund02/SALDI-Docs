# ProjectManager subsystem

Status: draft
Audience: maintainers, integrators

`projectManager/` is an embedded project-management and issue-tracking subsystem layered onto the main ERP application.

## Purpose
The subsystem appears to support:
- projects
- issues/tasks
- boards/views
- comments/activity
- notifications
- work logging/time tracking
- basic health/install checks

## Main entry points
- `projectManager/index.php` — dashboard
- `projectManager/projects.php` — project list and create flow
- `projectManager/project_view.php` — per-project board/detail view
- `projectManager/status.php` — install/health check
- `projectManager/install_project_management.php` — installer
- `projectManager/install_direct.php` — alternate/direct install path
- `projectManager/api/issues.php` — issue-related AJAX/API endpoint
- `projectManager/api/notifications.php` — notifications endpoint

## Architecture
Core pieces include:
- `projectManager/config.php` — subsystem config, auth/session helpers, utilities
- `projectManager/includes/ProjectManager.php` — main service layer for projects, issues, comments, work logs, notifications, and stats
- schema SQL files — ordered/unordered schema plus sample data

The visible UI pages appear to call into the shared service layer rather than implementing all business logic inline.

## Install flow
Observed install path:
1. use `status.php` to inspect health and setup state
2. run installer (`install_project_management.php` or similar)
3. schema SQL is loaded and executed
4. optional sample data may be inserted
5. return to `status.php` or UI pages for verification

The subsystem is PostgreSQL-oriented and expects its schema to exist before normal use.

## Auth and session coupling
This module is tightly coupled to the main ERP auth/session model.

Observed assumptions:
- uses ERP session vars such as `bruger_id` and `brugernavn`
- expects ERP DB/bootstrap includes such as `../includes/connect.php` and `../includes/online.php`
- is not a standalone authenticated application
- in some paths may fall back to `user_id = 1` if no better session context is found

That fallback should be treated as a risk in audit-sensitive workflows.

## Existing docs
This is one of the better-documented parts of the repo already.

Key local docs:
- `projectManager/README.md`
- `projectManager/PROJECT_MANAGEMENT_README.md`
- schema SQL files
- installer/status pages

## Main risks
1. **session-coupling risk** — behavior depends on ERP login/session state
2. **default-user fallback risk** — some flows may attribute actions to `user_id = 1`
3. **schema-assumption risk** — UI/API assumes tables already exist
4. **partial standalone illusion** — looks like a subsystem, but depends heavily on host app runtime

## Safe maintenance guidance
- treat `config.php` and `includes/ProjectManager.php` as the core of the subsystem
- verify session propagation into UI and API endpoints after auth-related changes
- test status/install flow before and after schema changes
- review existing local docs before refactoring this area

## Suggested next expansion
This draft should later be extended with:
- schema/entity map
- project/issue lifecycle notes
- API endpoint summary for `api/issues.php` and `api/notifications.php`
- auth/session dependency checklist
