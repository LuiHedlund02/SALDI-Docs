# SOAP services and ProjectManager

Audience: integrators, maintainers

This page documents two special integration surfaces in the SALDI repository:
- `soapserver/` and `soapklient/` — legacy SOAP compatibility layer
- `projectManager/` — embedded project-management subsystem

## SOAP server: `soapserver/`
The SOAP server exposes WSDL-described operations such as:
- `logon`
- `logoff`
- `invoice`
- `singleselect`
- `multiselect`
- `singleinsert`
- `singleupdate`
- `addorderline`

### Important behavior
Although exposed as SOAP/WSDL, the service behaves like a legacy compatibility layer:
- payloads are effectively tab-separated rather than rich typed objects
- server code relies on relative include/temp paths
- operations depend on legacy request/session assumptions
- table/action validation is limited and should not be treated as strong authorization

### Main risks
- weak input/table protection in some operations
- relative-path and writable-temp assumptions
- legacy password/encoding handling
- callers may assume stronger SOAP semantics than the implementation actually provides

For per-operation detail, see [`soap-operations.md`](soap-operations.md).

## SOAP client: `soapklient/`
`soapklient/` is the client-side wrapper layer for calling these SOAP services.

### Important behavior
- fetches and rewrites WSDLs dynamically
- depends on base URL assumptions for locating service definitions
- performs encoding conversions between legacy character sets and UTF-8
- relies on local writable paths for downloaded/rewritten WSDL files

### Integration guidance
- verify effective base URLs in each environment
- confirm writable paths for generated WSDL artifacts
- test non-ASCII data explicitly
- treat SOAP as a fragile legacy integration, not a modern stable API

## ProjectManager: `projectManager/`
`projectManager/` is an embedded issue/project tracking subsystem.

### Main entry points
- `index.php`
- `projects.php`
- `project_view.php`
- `status.php`
- `install_project_management.php`
- `install_direct.php`
- `api/issues.php`
- `api/notifications.php`

### Documentation status
This is one of the better-documented subsystems in the repo:
- `projectManager/README.md`
- `projectManager/PROJECT_MANAGEMENT_README.md`
- schema SQL files
- install/status pages

### Important assumptions
- expects the surrounding ERP auth/session to exist
- in some paths may fall back to `user_id = 1` if session context is missing
- depends on local schema/install state rather than being fully self-contained

### Integration guidance
- do not treat ProjectManager as independently authenticated
- verify session propagation into its API endpoints and UI pages
- review default-user fallback behavior before exposing new entry points or automation

For subsystem-specific details, see [`projectmanager.md`](projectmanager.md).

## Key risks
1. **SOAP contract ambiguity** — WSDL exists, but runtime semantics are legacy and loosely typed.
2. **SOAP environment fragility** — path, encoding, and writable-temp assumptions can break deployments.
3. **Weak server-side protection** — some SOAP operations deserve security review before wider use.
4. **ProjectManager auth coupling** — ERP session assumptions can create authorization surprises.
5. **Default-user fallback risk** — `user_id = 1` fallback can misattribute or over-authorize actions.
