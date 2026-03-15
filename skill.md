# Podfetcher CLI Skill

Use the `podfetcher` CLI to search podcasts, list episodes, and fetch transcripts via the Podfetcher API.

## Setup

Set environment variables before running any command:

```bash
export PODFETCHER_API_KEY="pk_live_your_key_here"
```

API keys are generated from the Podfetcher dashboard and always start with `pk_live_`. The base URL defaults to `https://api.podfetcher.com` — no need to set it.

Or pass `--api-key` and `--base-url` flags directly on each command.

The CLI binary is `node src/cli.js` (from this directory) or `podfetcher` if installed globally via npm.

## Commands

### Search shows

Find podcast shows by keyword.

```bash
podfetcher shows search --q "<query>" [--limit <n>] [--cursor <cursor>]
```

- `--q` (required): search query string
- `--limit`: max results to return
- `--cursor`: pagination cursor from a previous response
- `--json`: output raw JSON

Example:
```bash
podfetcher shows search --q "machine learning" --limit 5
podfetcher shows search --q "machine learning" --limit 5 --json
```

Output fields per show: `showId`, `title`, `author`
Pagination: if there are more results, `nextCursor` is printed — pass it as `--cursor` on the next call.

---

### List episodes

List episodes for a given show.

```bash
podfetcher shows episodes --show-id <showId> [options]
```

- `--show-id` (required): the show ID (e.g. `pi_1001`)
- `--from`: ISO date string, only episodes published on or after this date
- `--to`: ISO date string, only episodes published on or before this date
- `--since`: ISO date string, shorthand for episodes since a point in time
- `--order-by`: field to sort by (e.g. `publishedAt`)
- `--order`: sort direction — `asc` or `desc`
- `--limit`: max results to return
- `--cursor`: pagination cursor
- `--json`: output raw JSON

Example:
```bash
podfetcher shows episodes --show-id pi_1001 --order-by publishedAt --order desc --limit 10
podfetcher shows episodes --show-id pi_1001 --from 2024-01-01 --to 2024-12-31 --json
```

Output fields per episode: `episodeId`, `publishedAt`, `title`, `transcriptStatus`

---

### Fetch transcript

Request or retrieve the transcript for an episode.

```bash
podfetcher transcripts fetch --episode-id <episodeId> [options]
```

- `--episode-id` (required): the episode ID (e.g. `ep_pi_1001_002`)
- `--wait`: poll until the transcript is ready (blocks until done or timeout)
- `--poll-interval-ms`: polling interval when `--wait` is set (default: 1000)
- `--wait-timeout-ms`: max time to wait in ms when `--wait` is set (default: 60000)
- `--idempotency-key`: custom idempotency key (auto-generated if omitted)
- `--json`: output raw JSON

Example — fire and forget (returns immediately, transcript may still be processing):
```bash
podfetcher transcripts fetch --episode-id ep_pi_1001_002
```

Example — wait for transcript to be ready:
```bash
podfetcher transcripts fetch --episode-id ep_pi_1001_002 --wait --wait-timeout-ms 120000
```

**Possible outcomes:**

1. Transcript is **ready**: prints `episodeId`, `source`, `tokensCharged`, and the full transcript text.
2. Transcript is **processing**: prints `jobId` and `status=PROCESSING` (and estimated tokens if available). Re-run with the same `--episode-id` later, or use `--wait` to block.

---

## Global flags (available on all commands)

| Flag | Env var | Default |
|------|---------|---------|
| `--api-key <key>` | `PODFETCHER_API_KEY` | — |
| `--base-url <url>` | `PODFETCHER_BASE_URL` | `https://api.podfetcher.com` |
| `--api-key-header <header>` | `PODFETCHER_API_KEY_HEADER` | `X-API-Key` |
| `--timeout-ms <ms>` | — | `15000` |
| `--json` | — | false |

---

## Typical workflow

```bash
# 1. Find a show
podfetcher shows search --q "lex fridman" --limit 3 --json

# 2. List recent episodes
podfetcher shows episodes --show-id <showId> --order-by publishedAt --order desc --limit 5 --json

# 3. Fetch transcript (wait for it)
podfetcher transcripts fetch --episode-id <episodeId> --wait --json
```

## Error handling

- HTTP errors are printed as `[HTTP <status>] <code>: <message>`
- If `--api-key` is missing, the CLI exits with: `Missing API key. Provide --api-key or set PODFETCHER_API_KEY.`
- Exit code is `1` on any error, `0` on success
