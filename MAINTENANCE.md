# Zotero MCP — Quick Start & Maintenance

This is a local fork of [zotero-mcp](https://github.com/StevenYuyy/zotero-mcp) that lets Claude Desktop search your Zotero library by meaning, not just keywords. You talk to Claude, Claude talks to Zotero through this plugin.

## What you need before starting

1. **Zotero 7+** — installed and running. In Preferences, turn on "Allow other applications on this computer to communicate with Zotero."
2. **Claude Desktop** — installed from [claude.ai](https://claude.ai/download).
3. **Python 3.10+** — check with `python3 --version` in Terminal.
4. **uv** (Python package manager) — install with:
   ```bash
   curl -LsSf https://astral.sh/uv/install.sh | sh
   ```
5. **An OpenAI API key** — for the embedding model that powers semantic search. Get one at [platform.openai.com/api-keys](https://platform.openai.com/api-keys). Costs ~$0.01 to index a full library.

## First-time setup

Open Terminal (Cmd+Space, type "Terminal", hit Enter). Copy-paste each block.

### 1. Clone and install

```bash
git clone https://github.com/menyoung/zotero-mcp.git ~/claude/zotero-mcp
cd ~/claude/zotero-mcp
git checkout local-library-env-vars
uv tool install . --force
```

This puts a `zotero-mcp` command at `~/.local/bin/zotero-mcp`.

### 2. Configure Claude Desktop

Open this file in a text editor (create it if it doesn't exist):

```
~/Library/Application Support/Claude/claude_desktop_config.json
```

Paste the following, replacing `YOUR_OPENAI_API_KEY` with your actual key:

```json
{
  "mcpServers": {
    "zotero": {
      "command": "/Users/YOURUSERNAME/.local/bin/zotero-mcp",
      "env": {
        "ZOTERO_LOCAL": "true",
        "ZOTERO_EMBEDDING_MODEL": "openai",
        "OPENAI_API_KEY": "YOUR_OPENAI_API_KEY",
        "OPENAI_EMBEDDING_MODEL": "text-embedding-3-large"
      }
    }
  }
}
```

Replace `YOURUSERNAME` with your macOS username (run `whoami` in Terminal if unsure).

To add a **group library** as a separate connector, add another entry inside `mcpServers` with `"ZOTERO_LIBRARY_TYPE": "group"` and `"ZOTERO_LIBRARY_ID": "YOUR_GROUP_ID"`. The group ID is the number in the URL when you view the group on zotero.org.

### 3. Build the search index

Make sure Zotero is open, then:

```bash
ZOTERO_LOCAL=true ~/.local/bin/zotero-mcp update-db --fulltext --force
```

This reads your Zotero database, extracts text from PDFs, and creates embeddings. Takes 10-15 minutes for ~800 papers. You'll see a progress counter — wait for "Errors: 0" at the end.

### 4. Restart Claude Desktop

Quit Claude Desktop completely (Cmd+Q) and reopen it. You should see a hammer icon in the chat input area — that means the Zotero tools are loaded.

### 5. Try it

Ask Claude things like:
- "Search my Zotero library for papers about grain boundary migration"
- "Find papers related to dislocation dynamics in BCC metals"
- "What papers do I have on phase-field modeling?"

## Day-to-day use

**You don't need to do anything special.** Just have Zotero running when you use Claude Desktop. The search index auto-updates daily.

To manually refresh the index after adding new papers:

```bash
ZOTERO_LOCAL=true ~/.local/bin/zotero-mcp update-db --fulltext
```

(Without `--force`, this only indexes new papers — much faster.)

To check what's indexed:

```bash
~/.local/bin/zotero-mcp db-status
```

---

## Maintenance reference

Everything below is for when things break or need updating.

### Pull upstream changes

If the original zotero-mcp project has updates you want:

```bash
cd ~/claude/zotero-mcp
git fetch upstream
git checkout main
git merge upstream/main
git checkout local-library-env-vars
git rebase main
```

If you haven't set the upstream remote yet:

```bash
cd ~/claude/zotero-mcp
git remote add upstream https://github.com/StevenYuyy/zotero-mcp.git
```

After pulling, reinstall (see next section).

### Reinstall after source changes

Any time you (or Claude Code) edit the Python source files, you must reinstall:

```bash
cd ~/claude/zotero-mcp
uv tool install . --force --reinstall
```

Then quit and reopen Claude Desktop.

### Force-rebuild the search index

If search results seem wrong or you changed embedding settings:

```bash
ZOTERO_LOCAL=true ~/.local/bin/zotero-mcp update-db --fulltext --force
```

`--force` wipes and rebuilds everything from scratch. Takes 10-15 min.

### Inspect the database

```bash
~/.local/bin/zotero-mcp db-status
~/.local/bin/zotero-mcp db-inspect
```

### Claude Desktop config

Config file location:

```
~/Library/Application Support/Claude/claude_desktop_config.json
```

Current setup has two Zotero connectors:
- **zotero-personal** — your personal library (no extra env vars needed)
- **zotero-3dcvd** — group library (group ID 4903531)

Both use local mode and OpenAI `text-embedding-3-large` for embeddings.

### Config files

| File | What it does |
|---|---|
| `~/Library/Application Support/Claude/claude_desktop_config.json` | Claude Desktop's MCP server config (API keys, env vars) |
| `~/.config/zotero-mcp/config.json` | zotero-mcp's own config (embedding model, update schedule) |
| `~/.config/zotero-mcp/chroma_db/` | The search index itself (ChromaDB storage) |
| `~/Zotero/zotero.sqlite` | Zotero's own database (read-only, never edit this) |

### Key things to know

- **One binary, two connectors**: Both MCP instances run the same `zotero-mcp` binary with different environment variables.
- **Shared search index**: Both libraries share one ChromaDB collection (`zotero_library`). Using `--force` wipes everything.
- **Branch state**: Our local fixes live on branch `local-library-env-vars`. This is a few commits ahead of `main` with support for group libraries and improved semantic search quality.
- **ChromaDB warnings**: "Could not reconstruct embedding function" warnings during indexing are harmless. Ignore them.
- **Embedding costs**: OpenAI `text-embedding-3-large` costs about $0.13 per million tokens. A full 800-paper index rebuild uses ~$0.01.

## Troubleshooting

**Claude Desktop doesn't show the hammer icon:**
- Is Zotero open?
- Quit Claude Desktop fully (Cmd+Q) and reopen.
- Check the config JSON for typos (missing commas are the usual culprit).

**"No results" from semantic search:**
- Run `~/.local/bin/zotero-mcp db-status` to check if the index exists.
- Rebuild with `--fulltext --force` if needed.

**Errors during indexing:**
- A few errors out of hundreds is normal (corrupt PDFs, etc.).
- If you see 100+ errors, something is wrong — check that Zotero is running and your OpenAI API key is valid.

**After a macOS update or Python update:**
- Reinstall: `cd ~/claude/zotero-mcp && uv tool install . --force --reinstall`
