# Agent AHHOG — Claude Code Skill

A Claude Code skill that brings the public Ahrefs Agent capability model into MCP-bound Claude/Codex workflows, then extends it with conversion analytics. It turns **Ahrefs** data (search, links, rankings, AI visibility), **PostHog** data (on-site behavior, funnels, conversions, retention), and optional **GSC/live SERP** sources into finished marketing artifacts.

The name **AHHOG** = **A**hrefs + Post**HOG**, the two data sources it joins. (PostHog's mascot is a hedgehog, hence the HOG.)

The combination is the point. Ahrefs tells you who arrives and from where; PostHog tells you what they do once they land. Joined, they answer what marketing actually cares about: which SEO investment produces revenue, and where the journey breaks.

## What it does

When this skill is active in Claude Code, Claude becomes an **operator** that:

- Listens for marketing intents (audits, content gaps, link prospecting, brand monitoring, AI visibility, conversion analysis, content ROI, etc.)
- Pulls real data from the Ahrefs MCP server and/or the PostHog MCP server
- Synthesizes the data into prioritized findings — prioritized by conversion value when both sources are available
- Delivers finished artifacts — markdown reports, CSV exports, dashboards, tickets, monitoring specs, content calendars, pitch decks — saved to disk first

It ships with:

- **30+ single-source Ahrefs' Agent-style workflows** (content calendars, content gap, cannibalization, content decay, site audit discovery/triage/fixer, backlink analysis, anchor text analysis, broken link building, link intersect, linkbait opportunities, unlinked mentions, SERP features, Brand Radar visibility, AI mention gaps, share of voice, AI-cited pages, AI citation freshness, AI brand sentiment, competitor overview, keyword research, trending keyword research, SEO-to-AI query conversion, programmatic SEO, AI bot/web analytics, page traffic opportunities, web analytics deep dive, rank tracker report, E-E-A-T audit, migration plan, internal link map, traffic forecast, client pitch deck)
- **6 combined Ahrefs × PostHog workflows** (seo-to-conversion, landing-page-cro, content-roi, channel-truth, audience-segment-seo, campaign-impact)
- **PostHog-only analysis** (funnels, retention, paths, cohorts, session replays)
- The composition logic to mix them when a single request spans several.

Agent AHHOG is not official Ahrefs' Agent. The public Ahrefs' Agent page says Ahrefs' Agent has full Ahrefs UI-parity access; this skill uses the Ahrefs MCP tools currently connected in your environment and is explicit when a public Ahrefs' Agent capability is not exposed through MCP.

## Prerequisites

1. **Claude Code** installed.
2. **Ahrefs MCP** (for SEO/link/ranking/AI-visibility work) — needs an Ahrefs account, Lite plan or higher:

   ```bash
   claude mcp add --transport http ahrefs https://api.ahrefs.com/mcp/mcp
   ```

   Setup guide: https://docs.ahrefs.com/en/mcp/docs/claude-code

3. **PostHog MCP** (for conversion/behavior work) — free to call; needs a PostHog account:

   ```bash
   claude mcp add --transport http posthog https://mcp.posthog.com/mcp
   ```

   Or authenticate with a personal API key using the "MCP Server" preset. Setup guide: https://posthog.com/docs/model-context-protocol

   You can also scope the PostHog server to one project or a subset of tools via query params, e.g. `?features=insights,dashboards` — see the PostHog docs.

4. **SerpApi MCP** (optional — for live SERP data only) — needs a SerpApi account (free plan: 250 searches/month):

   ```bash
   claude mcp add --transport http serpapi https://mcp.serpapi.com/YOUR_API_KEY/mcp
   ```

   Replace `YOUR_API_KEY` with your key from https://serpapi.com/dashboard. **This MCP is off by default** — Agent AHHOG only proposes using it when Ahrefs can't provide sufficiently fresh data, and always asks your explicit approval (with credit cost stated) before calling it. The skill works fully without this connected.

5. **GSC MCP** (optional — direct Google Search Console access) — by [AminForou](https://github.com/AminForou/mcp-gsc), for URL inspection, indexing, and more granular GSC analytics than Ahrefs provides:

   ```bash
   # Install uv first (if not already installed)
   curl -LsSf https://astral.sh/uv/install.sh | sh && source $HOME/.local/bin/env

   # Add to Claude Desktop config (~/Library/Application Support/Claude/claude_desktop_config.json):
   # { "mcpServers": { "gscServer": { "command": "/full/path/to/uvx",
   #   "args": ["mcp-search-console"],
   #   "env": { "GSC_OAUTH_CLIENT_SECRETS_FILE": "/path/to/client_secrets.json" } } } }
   ```

   Requires a Google Cloud project with Search Console API enabled + OAuth credentials. Full setup: https://github.com/AminForou/mcp-gsc

   When connected, Agent AHHOG **automatically prefers this over Ahrefs' built-in GSC tools** — it's more accurate (direct first-party) and adds URL inspection and indexing capabilities. The skill works without it; Ahrefs `gsc-*` tools are the fallback.

6. Optional: Notion, Slack, GitHub, Linear, Google Drive, HubSpot, WordPress, Airtable, Mailchimp, Resend, Semrush, Apify, Firehose, or similar MCPs/connectors for output destinations.

## Installing the skill

Copy the `agent-ahhog/` folder into your Claude Code skills directory:

```bash
# project-level
cp -r agent-ahhog/ .claude/skills/

# or user-level
cp -r agent-ahhog/ ~/.claude/skills/
```

Restart Claude Code so it picks up the new skill.

## Using it

Just talk to Claude naturally:

SEO / Ahrefs:
- "Audit my site, ahrefs.com"
- "Find content gaps between us and our top three competitors in the US market"
- "Build a content calendar for the next quarter"
- "Create a link building strategy and prospect list"
- "Build me a link prospecting list — competitors are example1.com and example2.com"
- "How is our brand showing up in ChatGPT vs. competitors?"
- "Turn these SEO keywords into AI-search prompts"
- "Find page traffic opportunities from GSC"
- "Create a client pitch deck for this prospect"

Conversion / PostHog:
- "Where do users drop off in our signup funnel?"
- "Do visitors from our blog come back?"
- "Watch a few sessions on our pricing page and tell me why people leave"

Closed-loop / both:
- "Which keywords actually drive signups?"
- "This page gets tons of traffic but doesn't convert — why?"
- "Which blog posts are worth keeping vs. pruning?"
- "Did our site migration last month help or hurt?"

The skill auto-triggers on any phrase matching its scope. It will:
1. Confirm targets, competitors, market, and conversion event if not clear.
2. Discover the PostHog event schema (when conversions are involved), then pull Ahrefs and/or PostHog data.
3. Synthesize and prioritize — by conversion value when both sources are available.
4. Save a finished markdown report (and CSV where useful) to your working directory.

PostHog writes are tiered: creating dashboards, cohorts, draft experiments, surveys, and annotations happens without friction. Changes to live feature flags or experiments are confirmed when needed. Deletions are never performed by the skill.

## File structure

```
agent-ahhog/
├── SKILL.md                          # Main skill instructions (always loaded when triggered)
├── README.md                         # This file
└── references/                       # Loaded on demand
    ├── ahrefs-mcp-tools.md           # Map of ~125 Ahrefs MCP tools, when to call each
    ├── agent-a-capabilities.md       # Public Ahrefs' Agent capability mapping to Agent AHHOG workflows
    ├── posthog-mcp-tools.md          # Marketing-relevant PostHog MCP tools + write tiers (no delete)
    ├── serpapi-mcp-tools.md          # SerpApi opt-in rules, approval protocol, engine reference
    ├── gsc-mcp-tools.md              # Direct GSC MCP (20 tools): priority vs Ahrefs gsc-*, workflows
    ├── workflows.md                  # Ahrefs' Agent-style and single-source Ahrefs workflow recipes
    ├── combined-workflows.md         # 6 Ahrefs × PostHog closed-loop recipes + PostHog-only workflows
    ├── output-formats.md             # Markdown / Slack / Notion / Linear / CSV / HTML templates
    └── prompting-patterns.md         # How to interpret common user phrasings + source routing
```

## Costs and limits

Ahrefs MCP calls consume API units from your Ahrefs plan — track usage in Ahrefs → Account settings → Limits and usage.

Official Ahrefs' Agent may expose capabilities that are not available through Ahrefs MCP. Agent AHHOG does not pretend those endpoints exist; it uses the connected MCP surface, the Ahrefs MCP `doc` tool, and closest-available workflows.

PostHog MCP is free to call (it doesn't add to your PostHog bill), but it queries and can modify your real project data. This skill is write-enabled for full marketing analytics — it can build dashboards, create cohorts, run experiments, and launch surveys. Writes are risk-tiered: routine creation happens without friction, while changes to **live** feature flags/experiments are confirmed at the moment they happen. **Deletions are disabled** — the skill never deletes anything; it explains how to do it yourself instead.

SerpApi MCP consumes search credits from your plan (free: 250/month). Agent AHHOG always states the estimated credit cost and waits for your explicit approval before making any SerpApi call — no credit is ever spent without you saying yes.

## Customizing

The workflow library in `references/workflows.md` is the most useful thing to customize. Add your own workflow recipes following the same format:

```markdown
## your-workflow-name
**Triggers:** "natural phrases users say"
**Inputs needed:** [...]
**MCP calls:** [...]
**Synthesis:** [...]
**Output:** [...]
```

The skill will compose them with the built-in ones.

## Acknowledgements

Agent AHHOG is an independent skill inspired by the public capabilities of Ahrefs' Agent (https://ahrefs.com/), extended with PostHog product analytics. It is not affiliated with or endorsed by Ahrefs, PostHog, Letaido, or Anthropic. It uses the official Ahrefs and PostHog MCP servers per each provider's terms of use.
