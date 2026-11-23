# Apify Run Health Monitor MCP Server

[![Glama score](https://glama.ai/mcp/servers/mambalabsdev/mcp-apify-run-health-monitor/badges/score.svg)](https://glama.ai/mcp/servers/mambalabsdev/mcp-apify-run-health-monitor) [![MCP Registry](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fregistry.modelcontextprotocol.io%2Fv0%2Fservers%3Fsearch%3Dcom.mambabuilt%252Fmcp-apify-run-health-monitor%26limit%3D1&query=%24.servers%5B0%5D._meta%5B%22io.modelcontextprotocol.registry%2Fofficial%22%5D.status&label=mcp%20registry&color=blue)](https://registry.modelcontextprotocol.io/v0/servers?search=com.mambabuilt/mcp-apify-run-health-monitor&limit=1) [![npm version](https://img.shields.io/npm/v/@mambalabsdev/mcp-apify-run-health-monitor)](https://www.npmjs.com/package/@mambalabsdev/mcp-apify-run-health-monitor) [![npm downloads](https://img.shields.io/npm/dm/@mambalabsdev/mcp-apify-run-health-monitor)](https://www.npmjs.com/package/@mambalabsdev/mcp-apify-run-health-monitor) [![license](https://img.shields.io/github/license/mambalabsdev/mcp-apify-run-health-monitor)](https://github.com/mambalabsdev/mcp-apify-run-health-monitor/blob/main/LICENSE)

An MCP server for the Mamba Labs Apify Run Health Monitor actor on Apify. Which of your Apify actors failed, slowed down, or returned nothing.

## What's Inside

- [What it does](#what-it-does)
- [Quick start](#quick-start)
- [Prerequisites](#prerequisites)
- [Example prompts](#example-prompts)
- [Tool](#tool)
- [Inputs](#inputs)
- [Pricing](#pricing)
- [Full actor documentation](#full-actor-documentation)
- [License](#license)

## What it does

Give your own Apify API token and the actors to check, or the word all for every actor with a run in the window. The actor reads your run list and the newest datasets from the Apify API and returns one health row per actor: runs by status, failure rate, the latest run, median and slowest duration, slow runs, empty dataset runs, and platform usage, with a verdict and plain reasons. Each failed, timed out, aborted, empty, slow, or stuck run also gets a free alert row with its Console link.

All of the work runs on Apify. This package is a thin client that starts the actor run, waits for it, and hands back the dataset rows.

## Quick start

You need Node.js 18 or newer and an Apify account with an API token.

Add this to your Claude Desktop config (`claude_desktop_config.json`), Cursor, or any MCP client:

```json
{
  "mcpServers": {
    "mamba-apify-run-health-monitor": {
      "command": "npx",
      "args": [
        "-y",
        "@mambalabsdev/mcp-apify-run-health-monitor"
      ],
      "env": {
        "APIFY_TOKEN": "your-apify-token"
      }
    }
  }
}
```

Restart the client. The `check_apify_run_health` tool appears.

## Prerequisites

- Node.js 18 or newer.
- An Apify API token from [console.apify.com/account/integrations](https://console.apify.com/account/integrations), set as `APIFY_TOKEN`. The server lists its tool without a token; a call needs one.

## Example prompts

- "Check all my Apify actors from the last 24 hours and list the unhealthy ones."
- "Which of my scrapers returned an empty dataset this week?"
- "Show the failed and timed out runs for apify/web-scraper in the last 3 days."

## Tool

`check_apify_run_health`. One tool. Example arguments:

```json
{
  "actors": [
    "all"
  ],
  "lookback_hours": 24
}
```

## Inputs

Every buyer facing input of the actor. Omitted inputs take the actor's own defaults.

| Input | Type | Required | Description |
|---|---|---|---|
| `actors` | array | no | Actor IDs, username/name, or Apify Console or Store URLs, one per line. Use the single word all to check every actor that started a run in the window. Store actors you run count too: their runs on your account are read. |
| `api_token` | string | no | Your own Apify API token, from Apify Console, Settings, API and Integrations. It is sent only to api.apify.com and is never stored, logged, or written to a row. A scoped token needs read access to actors, runs, and datasets. With no token, the run writes one free row and reads nothing. |
| `lookback_hours` | number | no | How far back to read runs, 1 to 2160 hours (default 24). Runs started in this window are counted. |
| `max_failure_pct` | number | no | An actor whose failed plus timed out runs pass this share of its finished runs is marked unhealthy, 0 to 100 (default 10). |
| `min_items` | number | no | A succeeded run whose dataset holds fewer items than this counts as empty (default 1). Set 0 to skip the dataset check. |
| `datasets_per_actor` | number | no | How many of each actor's newest succeeded runs get their dataset item count read, 0 to 20 (default 3). |
| `slow_factor` | number | no | A succeeded run that takes more than this many times the actor's median run time, and at least 30 s longer, counts as slow, 1.2 to 100 (default 2). Needs 3 or more succeeded runs in the window. |
| `count_aborted_as_failed` | boolean | no | yes or no (default no). Aborted runs are often stopped by hand, so by default they lower the verdict to warning but do not count in the failure rate. |
| `alert_rows` | boolean | no | yes or no (default yes). Writes a free row for each problem run, newest first, up to 20 per actor. |
| `include_idle_actors` | boolean | no | yes or no (default no). With actors set to all, also checks every actor your account owns that started no run in the window; each gets the verdict no_runs and is charged like any other actor checked. |
| `max_actors` | number | no | Most actors to check in one run, 1 to 1000 (default 100). With all, the actors with the most runs come first. |
| `max_runs` | number | no | Most runs to read from your run list, newest first, 100 to 50000 (default 10000). Each health row says when this limit cut the window short. |
| `max_charge_usd` | string | no | Optional. The run stops before a charge that would pass this amount. Rows already written stay, and one free row names the first actor not checked. |

## Pricing

Pay per event on Apify, billed to your Apify account. Prices in USD.

| Event | Free plan | Bronze | Silver | Gold |
|---|---|---|---|---|
| `actor-checked`, one actor checked, with its alert rows | $0.003 | $0.00285 | $0.0027 | $0.00255 |
| `apify-actor-start`, per run start | $0.002 | $0.002 | $0.002 | $0.002 |

Alert rows, the row for a missing or refused token, actors not found, and refused input are free. Mamba Labs pays the platform costs of the run. Mamba Labs pays the platform costs of the run, Apify proxy traffic included.

## Full actor documentation

[apify.com/mambalabs/apify-run-health-monitor](https://apify.com/mambalabs/apify-run-health-monitor)

## License

MIT. See [LICENSE](LICENSE).
