# Backup and restore runbook

Status: draft
Audience: ops, senior maintainers

This runbook summarizes how backup and restore appear to work in the current SALDI codebase and what to validate before using them in production.

## Important warning
Backup/restore in SALDI is a privileged, shell-driven workflow. It assumes:
- writable temp directories
- available external binaries
- strong DB privileges
- safe path handling
- low user activity during restore

Do not test restore first in production.

## Main artifacts and temp paths
Observed backup artifacts include:
- `temp/backup.info`
- `temp/<db>.sql`
- `temp/<db>_<timestamp>.tar`
- `temp/<db>_<timestamp>.tar.gz`
- `temp/<db>_<timestamp>.sdat`
- persistent copies under `temp/backup/<db>/`

Restore flows may unpack or stage files under:
- `temp/<db>/`
- `temp/<db>/temp/`
- temporary names such as `restore.gz` or `restore.sql`

Deleted/retired accounts may also be archived under:
- `nedlagte_regnskaber/<db>/`

## External tool dependencies
Observed tool dependencies include:
- `pg_dump` / `mysqldump`
- `psql` / `mysql`
- `tar`
- `gzip`
- `gunzip`

The admin/settings layer may also manage related tools such as `ps2pdf`, `weasyprint`, `pdftk`, and `ncftp`, but the core backup/restore workflow depends mainly on DB dump/import and archive tools.

## Destructive assumptions and risks
Restore and account-deletion flows appear to assume:
- the DB admin can terminate sessions, drop, and recreate databases
- uploaded backup files are trusted SALDI artifacts
- shell execution is allowed on the host
- users are logged out or at least not actively writing during restore

High-risk operations observed or implied:
- DB replacement/rebuild
- archive extraction into temp/runtime paths
- shell commands over paths and filenames
- deletion flows that can end in `DROP DATABASE`
- retired-account archive handling outside the live DB

## Suggested safe backup procedure
1. Confirm `temp/` is writable and has enough disk space.
2. Confirm DB/tool paths in admin settings are valid.
3. Run the backup from the admin area.
4. Verify expected artifacts were created (`.sql`, archive, final `.sdat`).
5. Copy the final backup artifact out of the webroot/server scratch area.
6. Record:
   - timestamp
   - database/account name
   - app version if known
   - environment/host

## Suggested safe restore procedure
1. Use a disposable or staging environment first.
2. Confirm all expected binaries exist and are executable.
3. Ensure target users are logged out or the environment is isolated.
4. Place/upload the backup file through the intended restore path.
5. Confirm the restore file type is one SALDI expects (`.sdat` or `.sql`).
6. Run restore and monitor temp files/log output.
7. After restore, validate login and one representative workflow before reopening the system.

## Post-restore validation checklist
After restore, verify at minimum:
- login works
- the expected account/database opens
- `includes/connect.php` still points at the intended DB
- `temp/` is writable and new logs can be created
- one finance/accounting page loads
- one debtor/creditor/inventory page loads
- one document/PDF flow works
- one integration log path is writable
- backup/restore temp artifacts are cleaned up or understood

## Common failure categories
### 1. Missing or invalid external binaries
Symptoms:
- dump/import/archive creation fails
- restore stops before import

### 2. Temp-path and permission problems
Symptoms:
- artifacts not created
- extraction/import partially completes
- logs missing

### 3. DB privilege problems
Symptoms:
- cannot create/drop/select target DB
- cannot terminate sessions
- restore aborts mid-flight

### 4. Bad or unexpected input artifact
Symptoms:
- unsupported MIME/type handling
- wrong SQL dialect/encoding
- corrupted or partial archive extraction

### 5. Live-user interference
Symptoms:
- inconsistent state after restore
- active sessions point at stale data
- locks/conflicts during DB replacement

## Safe maintenance guidance
- Keep backup/restore docs and admin behavior aligned when changing paths or tooling.
- Treat path handling and shell execution as security-sensitive.
- After changing backup/restore code, test on a throwaway database first.
- Always verify both artifact creation and application usability after restore.
