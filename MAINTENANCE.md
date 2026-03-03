# Maintenance Guide

## Pull upstream changes

```bash
cd ~/claude/zotero-mcp
git fetch upstream
git checkout main
git merge upstream/main
git checkout local-library-env-vars
git rebase main
```

After pulling, reinstall the binary (see below).

## Reinstall binary after source changes

```bash
cd ~/claude/zotero-mcp

# Global binary at ~/.local/bin/zotero-mcp (used by Claude Desktop)
uv tool install . --force

# Local .venv copy (for development/testing)
uv pip install .
```

Both are non-editable — you must re-run these after any source changes.

## Update semantic search database

The `--fulltext` flag reads the local SQLite database, which indexes ALL libraries regardless of env vars. So one command covers everything:

```bash
ZOTERO_LOCAL=true ~/.local/bin/zotero-mcp update-db --fulltext
```

Add `--force` to wipe and rebuild from scratch:

```bash
ZOTERO_LOCAL=true ~/.local/bin/zotero-mcp update-db --fulltext --force
```

Without `--fulltext`, indexing goes through pyzotero HTTP (one library at a time, respects env vars, metadata only — fast but less comprehensive).

## Check database status

```bash
~/.local/bin/zotero-mcp db-status
~/.local/bin/zotero-mcp db-inspect
```

## Claude Desktop config

Config file: `~/Library/Application Support/Claude/claude_desktop_config.json`

Two MCP connectors configured:
- `zotero-personal` — personal library (default, no library env vars needed)
- `zotero-3dcvd` — group library (`ZOTERO_LIBRARY_TYPE=group`, `ZOTERO_LIBRARY_ID=4903531`)

Both use `ZOTERO_LOCAL=true` and point to `~/.local/bin/zotero-mcp`.

After any binary reinstall or config change, quit and reopen Claude Desktop.

## Key things to know

- **One binary, two connectors**: Both MCP instances run the same binary with different env vars.
- **Shared ChromaDB collection**: Both libraries share one collection (`zotero_library`). `--force` wipes everything.
- **Branch state**: The fix lives on branch `local-library-env-vars`, one commit ahead of `main`. The fix makes `get_local_zotero_client()` respect `ZOTERO_LIBRARY_TYPE` and `ZOTERO_LIBRARY_ID` env vars instead of hardcoding user library.
- **ChromaDB warnings**: "Could not reconstruct embedding function" warnings during indexing are a pre-existing upstream bug. They don't affect functionality.
