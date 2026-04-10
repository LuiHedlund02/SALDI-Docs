# ProjectManager subsystem

Audience: maintainers, integrators

`projectManager/` is an embedded project-management and issue-tracking subsystem layered onto the main ERP application.

## Purpose
The subsystem supports:
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
- `includes/ProjectManager.php` — main service layer for projects, issues, comments, work logs, notifications, and stats
- schema SQL files — ordered/unordered schema plus sample data

The visible UI pages call into the shared service layer rather than implementing all business logic inline.

## Install and schema prerequisites
Current install path is:
1. use `status.php` to inspect health and setup state
2. run installer (`install_project_management.php` or `install_direct.php`)
3. schema SQL is loaded and executed
4. optional sample data may be inserted
5. return to `status.php` or UI pages for verification

Operational notes:
- the subsystem is PostgreSQL-oriented
- tables must exist before normal UI/API use
- health/install pages are part of the expected lifecycle, not just one-time setup leftovers

## Auth and session coupling
This subsystem is tightly coupled to the main ERP auth/session model.

Current assumptions:
- uses ERP session vars such as `bruger_id` and `brugernavn`
- expects ERP bootstrap includes such as `../includes/connect.php` and `../includes/online.php`
- is not a standalone authenticated application
- in some paths may fall back to `user_id = 1` if no better session context is found

That fallback should be treated as a risk in audit-sensitive workflows.

## API surfaces
### `api/issues.php`
This is the main AJAX endpoint for issue/task lifecycle operations.

Current behaviors visible in the endpoint:
- `GET ?project_id=<id>` returns issues for a project with optional `status_id`, `assignee_id`, and `priority_id` filters
- `GET ?search=<term>` searches issues and returns a short result list
- `POST {"action":"create", ...}` creates an issue through `ProjectManager::createIssue()`
- `POST {"action":"update", ...}` updates an issue through `ProjectManager::updateIssue()`
- `POST {"action":"add_comment", ...}` adds an issue comment
- `POST {"action":"log_work", ...}` logs work against an issue

### `api/notifications.php`
This is the user-notification surface for the subsystem.

Current behaviors visible in the endpoint:
- `GET ?count_unread=1` returns unread notification count for the current user
- `GET ?limit=<n>` returns the current user’s notifications, default limit `20`
- `POST {"action":"mark_read","id":...}` marks one notification as read
- `POST {"action":"mark_all_read"}` marks all current-user notifications as read

Both endpoints depend on ERP session/user context and can fall back to `user_id = 1` when session state is missing.

## Entity map
The effective entity model centers on:
- projects
- issues/tasks
- comments/activity
- work logs/time entries
- notifications
- status/board metadata

The service layer in `includes/ProjectManager.php` is the best place to verify exact naming and lifecycle rules.

## Existing local docs
This is one of the better-documented parts of the repo already.

Key local docs:
- `projectManager/README.md`
- `projectManager/PROJECT_MANAGEMENT_README.md`
- schema SQL files
- installer/status pages

Read those local docs before refactoring the subsystem-level service layer.

## Main risks
1. **session-coupling risk** — behavior depends on ERP login/session state.
2. **default-user fallback risk** — some flows may attribute actions to `user_id = 1`.
3. **schema-assumption risk** — UI/API assumes tables already exist.
4. **partial standalone illusion** — looks like a subsystem, but depends heavily on host app runtime.

## Safe maintenance guidance
- treat `config.php` and `includes/ProjectManager.php` as the core of the subsystem
- verify session propagation into UI and API endpoints after auth-related changes
- test status/install flow before and after schema changes
- review existing local docs before refactoring this area
