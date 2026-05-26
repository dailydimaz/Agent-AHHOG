# Google Search Console MCP — Tool Reference

**Source:** [mcp-gsc by AminForou](https://github.com/AminForou/mcp-gsc)

This MCP server provides **direct, first-party access to the user's Google Search Console account** — not estimates or re-processed data, but the exact same numbers the user sees in the GSC dashboard.

---

## Why this beats Ahrefs' GSC integration

Ahrefs has `gsc-*` tools (via its own GSC integration), but they have limitations that the dedicated MCP GSC server doesn't:

| Capability | Ahrefs `gsc-*` tools | MCP GSC (this) |
| --- | --- | --- |
| Data source | GSC data re-processed via Ahrefs | Direct GSC API — exact dashboard numbers |
| Query + page combined filter | No | Yes (`get_advanced_search_analytics`) |
| Country / device filter | Limited | Full (`get_advanced_search_analytics`) |
| URL inspection (crawl/index status) | No | Yes (`inspect_url_enhanced`, `batch_url_inspection`) |
| Indexing issue detection | No | Yes (`check_indexing_issues`) |
| Sitemap management | No | Yes (`get_sitemaps`, `manage_sitemaps`) |
| Property management | No | Yes (`list_properties`, `get_site_details`) |
| Period comparison | No | Yes (`compare_search_periods`) |
| Page-level query breakdown | No | Yes (`get_search_by_page_query`) |

**Rule:** when MCP GSC is connected, **prefer it over Ahrefs `gsc-*` tools** for all GSC-related queries. Fall back to Ahrefs `gsc-*` only if MCP GSC is unavailable.

---

## Availability check

MCP GSC is **optional** — Agent AHHOG works fully without it.

At the start of a session, check for MCP GSC tools. Tool names typically start with `mcp__gscServer__` or `gscServer:` depending on the client. Look for `list_properties`, `get_search_analytics`, or `get_capabilities`.

**If connected:** use MCP GSC for all GSC tasks instead of Ahrefs `gsc-*` tools.

**If not connected:** silently fall back to Ahrefs `gsc-*` tools (if Ahrefs is connected) and note the data is GSC via Ahrefs. Do **not** prompt the user to set up MCP GSC unless they specifically ask about first-party GSC access or URL inspection — they may not need it.

**If neither is connected:** tell the user you don't have GSC access and explain both options:
> "I don't have access to your Google Search Console data. Two ways to connect: (1) link GSC to your Ahrefs account (gives GSC trends and keyword data via Ahrefs), or (2) set up the mcp-gsc server for direct GSC access including URL inspection and indexing tools."

---

## Setup (for users who want to connect)

### Quickest: uvx method (no cloning)

```bash
# Install uv
curl -LsSf https://astral.sh/uv/install.sh | sh
source $HOME/.local/bin/env
echo 'source $HOME/.local/bin/env' >> ~/.zshrc

# Find full uvx path (needed for config)
which uvx
# Returns something like: /Users/yourname/.local/bin/uvx
```

**Claude Desktop config** (`~/Library/Application Support/Claude/claude_desktop_config.json`):

OAuth (recommended for personal use):
```json
{
  "mcpServers": {
    "gscServer": {
      "command": "/FULL/PATH/TO/uvx",
      "args": ["mcp-search-console"],
      "env": {
        "GSC_OAUTH_CLIENT_SECRETS_FILE": "/full/path/to/client_secrets.json"
      }
    }
  }
}
```

Service account (for teams/automation):
```json
{
  "mcpServers": {
    "gscServer": {
      "command": "/FULL/PATH/TO/uvx",
      "args": ["mcp-search-console"],
      "env": {
        "GSC_CREDENTIALS_PATH": "/full/path/to/service_account.json",
        "GSC_SKIP_OAUTH": "true"
      }
    }
  }
}
```

After saving: **fully quit and reopen Claude Desktop** (`Cmd+Q`, not just close window).

### Credentials setup (one-time)

1. Go to [Google Cloud Console](https://console.cloud.google.com/) → create/select a project
2. Enable the [Search Console API](https://console.cloud.google.com/apis/library/searchconsole.googleapis.com)
3. Go to Credentials → Create Credentials:
   - **OAuth client ID** (personal use) → Desktop app → Download JSON → save as `client_secrets.json`
   - **Service Account** (team) → Keys tab → Add Key → JSON → download → add the email to GSC property with Full access
4. Use the absolute file path in the config above

---

## All 20 tools

### Discovery and auth

| Tool | What it does | When to call |
| --- | --- | --- |
| `get_capabilities` | Lists all available tools + current auth status | **Always call first** when unsure if MCP GSC is working or auth is valid |
| `reauthenticate` | Re-runs OAuth browser login (switch accounts) | When auth errors appear or user wants to switch Google accounts |

### Property management

| Tool | What it does | Inputs |
| --- | --- | --- |
| `list_properties` | All GSC properties the account can access | Nothing |
| `get_site_details` | Details and verification info for a property | `site_url` |

### Search analytics — the core of GSC marketing work

| Tool | What it does | Inputs | Notes |
| --- | --- | --- | --- |
| `get_search_analytics` | Top queries and pages with clicks, impressions, CTR, position | `site_url`, date range | Start here for any keyword performance question |
| `get_performance_overview` | Summary of site performance for a period | `site_url`, date range | Quick top-line health check |
| `compare_search_periods` | Side-by-side comparison of two time periods | `site_url`, two date ranges | Before/after analysis — content launches, algorithm updates |
| `get_search_by_page_query` | Search queries driving traffic to a specific page | `site_url`, `page_url` | What terms a single page ranks for |
| `get_advanced_search_analytics` | Analytics with filters: country, device, query pattern, page | `site_url`, filters | **Most powerful query** — segment by mobile/desktop, country, specific queries |

**Key parameter for all analytics tools:**
- Date range: typically `startDate` and `endDate` in `YYYY-MM-DD` format
- `row_limit`: up to 500 rows per call (default 25)
- `dataState`: `"all"` (default, matches GSC dashboard) or `"final"` (2-3 day lag but confirmed data)

### URL inspection and indexing

| Tool | What it does | Inputs |
| --- | --- | --- |
| `inspect_url_enhanced` | Full crawl and index status for one URL | `site_url`, `page_url` |
| `batch_url_inspection` | Inspect up to 10 URLs at once | `site_url`, list of URLs |
| `check_indexing_issues` | Check multiple URLs for indexing problems with diagnosis | `site_url`, list of URLs |

Use these when: a page isn't ranking, traffic dropped suddenly, a newly published page isn't appearing in search, or during a site audit.

### Sitemap management

| Tool | What it does | Inputs |
| --- | --- | --- |
| `get_sitemaps` | Lists all sitemaps for a property | `site_url` |
| `list_sitemaps_enhanced` | Detailed sitemap info: errors, warnings, indexed count | `site_url` |
| `manage_sitemaps` | Submit a new sitemap or delete one | `site_url`, `action`, `sitemap_url` |

Note: `delete_sitemap` and site deletion (`add_site`, `delete_site`) are **disabled by default** in the MCP server. If the user needs these, they must set `GSC_ALLOW_DESTRUCTIVE=true` in their MCP config — Agent AHHOG does not enable this and will not instruct the user to enable it unless they specifically ask.

---

## GSC workflows

### Weekly SEO performance review

```
get_performance_overview → get_search_analytics (top 50 queries) → compare_search_periods (vs prior week)
```

Report: clicks/impressions trend, top query movers (wins and losses), CTR outliers.

### Indexing audit

```
list_properties → get_search_analytics (top pages by impressions) → batch_url_inspection (top 10 pages) → check_indexing_issues
```

Report: coverage by indexed/not-indexed, prioritized fix list.

### Page-level deep dive

```
get_search_by_page_query → inspect_url_enhanced → (if Ahrefs connected) site-explorer-organic-keywords filtered to that URL
```

Report: what queries the page ranks for, crawl/index status, and (if available) Ahrefs' view of the same page.

### Content opportunity mining

```
get_advanced_search_analytics with filter: position > 10, impressions > 100 → sort by impressions desc
```

This surfaces "position 11–20 with high impressions" — the classic quick-win category. Combine with `keywords-explorer-overview` (Ahrefs) to check KD and suggested intent.

### Algorithm update impact

```
compare_search_periods (before vs after update date) → get_advanced_search_analytics (filtered to biggest losers)
```

Report: delta in clicks/impressions/CTR/position for affected queries, pattern across page types.

---

## Combining GSC MCP with other Agent AHHOG sources

| Question | GSC MCP provides | Complement with |
| --- | --- | --- |
| "Why did traffic drop?" | Click/impression/position delta | Ahrefs: ranking changes, competitor gains |
| "Which pages have indexing issues?" | Crawl/index status per URL | Ahrefs: site-audit-issues for technical context |
| "What content is close to page 1?" | Position 11–20 queries | Ahrefs: keywords-explorer-overview for KD |
| "Is our featured snippet still ours?" | Current SERP via position data | SerpApi: live SERP snapshot (with approval) |
| "Does organic traffic convert?" | Clicks to landing page | PostHog: funnel from landing → conversion |
| "Did our content launch work?" | Click/impression delta before/after | PostHog: conversion delta before/after |

---

## Data freshness

GSC data has an inherent delay: typically 2–3 days for `dataState: "final"`, up to 24 hours for `dataState: "all"`. This is a GSC API limitation, not an MCP limitation. Always note the data delay in reports when freshness matters.

For truly live SERP data (what's ranking right now, not what GSC reported yesterday), SerpApi is the right tool — with user approval per the SerpApi approval protocol.
