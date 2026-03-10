# Zotero MCP — Local Fork

Fork of [54yyyu/zotero-mcp](https://github.com/54yyyu/zotero-mcp). Lets Claude Desktop semantic-search your Zotero library.

## Prerequisites

Zotero 7+ (with "Allow other applications..." on), Claude Desktop, Python 3.10+, [uv](https://astral.sh/uv), an [OpenAI API key](https://platform.openai.com/api-keys).

## Setup

```bash
git clone https://github.com/menyoung/zotero-mcp.git ~/claude/zotero-mcp
cd ~/claude/zotero-mcp
git checkout local-tweaks
uv tool install . --force          # installs ~/.local/bin/zotero-mcp
```

Edit `~/Library/Application Support/Claude/claude_desktop_config.json` — see existing config for env var examples. Key vars: `ZOTERO_LOCAL`, `ZOTERO_API_KEY`, `ZOTERO_LIBRARY_ID`, `ZOTERO_LIBRARY_TYPE` (for groups), `ZOTERO_EMBEDDING_MODEL`, `OPENAI_API_KEY`.

Build index (Zotero must be open): `ZOTERO_LOCAL=true zotero-mcp update-db --fulltext --force`

Restart Claude Desktop (Cmd+Q, reopen). Hammer icon = tools loaded.

## Common commands

| Command | What it does |
|---|---|
| `zotero-mcp update-db --fulltext --force` | Full reindex (~17 min / 900 papers) |
| `zotero-mcp update-db --fulltext` | Index new papers only |
| `zotero-mcp db-status` | Check index stats |
| `uv tool install . --force` | Reinstall after code changes |

Always set `ZOTERO_LOCAL=true` when running CLI commands. Restart Claude Desktop after reinstalling.

## Updating from upstream

```bash
git fetch upstream         # upstream = 54yyyu/zotero-mcp
git checkout main && git merge upstream/main
git checkout local-tweaks && git rebase main
uv tool install . --force
```

## Branch stack

```
upstream/main
  pr/bugfix-daily-update-config      # PR #144
    pr/local-library-env-vars        # env var library selection
      pr/metadata-header-embedding   # header prepend + child notes indexing
        local-tweaks                 # batch_size, maxpages, advanced_search desc
```

## Config files

| File | Purpose |
|---|---|
| `~/Library/Application Support/Claude/claude_desktop_config.json` | MCP server config (API keys, env vars) |
| `~/.config/zotero-mcp/config.json` | Embedding model, update schedule |
| `~/.config/zotero-mcp/chroma_db/` | ChromaDB index (wiped by `--force`) |
| `~/Zotero/zotero.sqlite` | Zotero's DB (read-only, never edit) |

## Notes

- Two MCP instances (personal + group 4903531) share one ChromaDB collection. `--force` wipes both.
- Local API (localhost:23119) is read-only for item creation. Writes (notes, tags) go through web API — needs `ZOTERO_API_KEY`.
- "Could not reconstruct embedding function" warnings are harmless.
- Embedding cost: ~$0.01 per full rebuild with `text-embedding-3-large`.
- No hammer icon? Check Zotero is open, Cmd+Q restart Claude Desktop, check JSON for typos.
- Bad search results? Rebuild with `--fulltext --force`. A few indexing errors out of hundreds is normal (corrupt PDFs).
- After macOS/Python update: `uv tool install . --force --reinstall`
