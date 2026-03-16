---
name: podfetcher-tools
description: Use this skill when you need to search podcast shows, list show episodes, or fetch episode transcripts from the Podfetcher API via the bundled CLI or MCP server.
---

# Podfetcher Tools

Use the bundled Node clients in this directory to interact with the Podfetcher API.

## Requirements

- Node.js 20+
- `PODFETCHER_API_KEY` set to a valid Podfetcher API key

The default API base URL is `https://api.podfetcher.com`. Override it only when targeting a non-production environment.

## Entry Points

Run commands from this directory, or reference these files by absolute path from another workspace:

- CLI: `node src/cli.js`
- MCP server: `node src/mcp.js`
- SDK import: `./src/sdk.js`

If the package is installed globally from npm, the binaries are:

- `podfetcher`
- `podfetcher-mcp`

## CLI Commands

### Search shows

```bash
node src/cli.js shows search --q "<query>" [--limit <n>] [--cursor <cursor>] [--json]
```

- `--q` is required
- Returns `items[]` with `showId`, `title`, and `author`
- If present, pass `nextCursor` into `--cursor` for the next page

### List episodes

```bash
node src/cli.js shows episodes --show-id <showId> [--from <iso>] [--to <iso>] [--since <iso>] [--order-by publishedAt] [--order asc|desc] [--limit <n>] [--cursor <cursor>] [--json]
```

- `--show-id` is required
- Returns `items[]` with `episodeId`, `publishedAt`, `title`, and `transcriptStatus`

### Fetch transcript

```bash
node src/cli.js transcripts fetch --episode-id <episodeId> [--wait] [--poll-interval-ms <ms>] [--wait-timeout-ms <ms>] [--idempotency-key <key>] [--json]
```

- `--episode-id` is required
- Without `--wait`, the API may return a queued job with `jobId` and `status=PROCESSING`
- With `--wait`, the client polls until the transcript is ready or the timeout expires

## Global CLI Options

- `--api-key <key>` or `PODFETCHER_API_KEY`
- `--base-url <url>` or `PODFETCHER_BASE_URL`
- `--api-key-header <header>` or `PODFETCHER_API_KEY_HEADER`
- `--timeout-ms <ms>`
- `--json`

## Typical Workflow

```bash
# 1. Find a show
node src/cli.js shows search --q "lex fridman" --limit 3 --json

# 2. List recent episodes
node src/cli.js shows episodes --show-id <showId> --order-by publishedAt --order desc --limit 5 --json

# 3. Fetch the transcript and wait for completion
node src/cli.js transcripts fetch --episode-id <episodeId> --wait --json
```

## MCP Server

Start the MCP server over stdio:

```bash
node src/mcp.js
```

Available tools:

- `search_shows`
- `list_episodes`
- `fetch_transcript`

Example config:

```json
{
  "mcpServers": {
    "podfetcher": {
      "command": "node",
      "args": ["/absolute/path/to/podfetcher-tools/src/mcp.js"],
      "env": {
        "PODFETCHER_API_KEY": "pk_live_..."
      }
    }
  }
}
```

## Error Handling

- HTTP errors are formatted as `[HTTP <status>] <code>: <message>`
- Missing API key errors are reported before the request is sent
- Exit code is `1` on error and `0` on success
