# Lab 4 Change Log

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
