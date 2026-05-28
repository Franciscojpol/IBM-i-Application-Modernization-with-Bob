# Lab 4 Change Log

## 2026-05-28

### Scope
- Lab 4 - IBM i MCP mode continuity
- Repository: IBM-i-Application-Modernization-with-Bob

### Changes Applied
1. Re-synchronized MCP source port configuration with env-driven setup.
- File: .bob/tools/samco-tools.yaml
- Change: `port: 54999` -> `port: ${DB2i_PORT}`
- Reason: The active file had drifted from the Lab 4 environment-variable convention and from previous change notes.

2. Fixed environment variable precedence in MCP launcher.
- File: .bob/start-ibmi-mcp.cjs
- Change: `.env` values now override pre-existing process environment values during launcher bootstrap.
- Reason: Stale DB2i credentials inherited by the process could shadow updated `.env` credentials and cause repeated authentication failures.

3. Verified Lab 4 Exercise 2 readiness with live IBM i metadata checks.
- File: lab4-ibmi-mcp-mode.md
- Change: added an optional SQL fallback section using QSYS2 services for system status checks.
- Reason: keep a practical execution path when only minimal MCP tools are enabled.

### Validation
- Consistency check passed against:
  - .bob/README.md (env-based DB2i_PORT guidance)
  - .bob/start-ibmi-mcp.cjs (required DB2i_* environment keys)
- Runtime MCP validation passed:
  - `list_service_categories` returned successful data from IBM i (29 service categories).
  - `QSYS2.ACTIVE_JOB_INFO` and `QSYS2.SYSTEM_STATUS_INFO` were confirmed available on the connected system.
  - Lab 4 Exercise 4 security checks completed:
    - `Show me users with limited capabilities`: no rows returned.
    - `QSYS2.OBJECT_PRIVILEGES` scoped to `OBJECT_SCHEMA='SAMCO'` and `AUTHORIZATION_NAME='*PUBLIC'` with `DATA_READ='YES'`: 0 rows.

### Notes
- No credentials or secret values were added or exposed.
- Lab 4 Exercise 5 (Bob CLI) intentionally deferred for a later session.
- Added troubleshooting note for npm 404 on `@ibm/ibmi-bob` and guidance to keep Exercise 5 deferred when package is unavailable.

## 2026-05-21

### Scope
- Lab 4 - IBM i MCP mode preparation
- Repository: IBM-i-Application-Modernization-with-Bob
- Branch: lab3-dds-sql-rla

### Changes Applied
1. Aligned MCP tool source port with Lab 4 environment-driven setup.
- File: .bob/tools/samco-tools.yaml
- Change: `port: 54999` -> `port: ${DB2i_PORT}`
- Reason: Match Lab 4 guidance based on Mapepire port configuration via environment variables.

2. Updated setup documentation to match current repository state.
- File: .bob/README.md
- Change: removed dependency on missing `.bob/mcp.json.example` and documented `.bob/mcp.json` as starting point.
- Reason: Avoid onboarding confusion and keep instructions executable as-is.

3. Added traceability notes in changed files.
- File: .bob/README.md
- File: .bob/tools/samco-tools.yaml
- Change: explicit "Change Note" entries with Lab 4 reference and date.
- Reason: Keep direct, local audit trail in edited artifacts.

### Backups Created
- .bob/backup-lab4-20260521/README.md.bak
- .bob/backup-lab4-20260521/samco-tools.yaml.changed-section.bak
- .bob/backup-lab4-20260521/README.md.before-traceability-note.bak
- .bob/backup-lab4-20260521/samco-tools.yaml.header.before-traceability-note.bak

### Validation
- Syntax/errors check passed for:
  - .bob/README.md
  - .bob/tools/samco-tools.yaml

### Notes
- No credential values were exposed in documentation updates.
- Runtime connectivity to IBM i/Mapepire still depends on environment validity (DB2i_HOST, DB2i_USER, DB2i_PASS, DB2i_PORT) and server availability.
