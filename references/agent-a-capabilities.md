# Ahrefs' Agent Capability Mapping

Source snapshot: https://ahrefs.com/, checked 2026-06-01.

This reference maps public Ahrefs' Agent capabilities to what Agent AHHOG should do in Claude/Codex. Agent AHHOG is independent and MCP-bound: it should mirror the job shape and artifact quality, not claim Ahrefs' Agent's full internal access.

## Access boundary

The public Ahrefs' Agent page says Ahrefs' Agent has full, unrestricted Ahrefs access with UI-parity endpoints. Agent AHHOG does not. It uses:

- Ahrefs MCP tools that are connected in the current client.
- Ahrefs MCP `doc` tool for parameter details and newly exposed endpoints.
- GSC MCP when connected for direct first-party Google Search Console data.
- PostHog MCP for conversion/product analytics.
- Optional SerpApi only when approved for live SERP/trend gaps.
- Local files and explicitly requested destination MCPs for delivery.

If a capability exists in Ahrefs' Agent but not in connected MCP tools, be direct: "I do not see that endpoint/tool exposed here. I can still produce [closest artifact] from [available data]."

## Public Ahrefs' Agent capability groups

Ahrefs' Agent positions itself as one AI marketing operator that can:

- Build content calendars.
- Find keyword cannibalization.
- Create link building strategy.
- Audit technical health.
- Analyze competitor backlinks.
- Find unlinked brand mentions.
- Benchmark against industry.
- Create client pitch decks.
- Map content gaps.
- Analyze SERP volatility.
- Track AI search visibility.
- Monitor brand mentions.
- Track share of voice.
- Plan site migrations.
- Map internal linking structure.
- Compare domain authority.
- Forecast traffic.

It also advertises prebuilt skills/apps such as:

- Content Gap Analysis.
- AI Mention Gap Analysis.
- Anchor Text Analysis.
- SERP Feature Opportunities.
- Linkbait Opportunities.
- Site Audit Issue Fixer.
- Site Audit Discovery.
- Trending Keyword Research.
- Brand Authority & E-E-A-T Audit.
- SEO to AI Query Converter.
- AI Brand Sentiment.
- Declining Content Detection.
- Fix Keyword Cannibalization.
- AI Bot & Web Analytics.
- Link Intersect Prospecting.
- Programmatic SEO Keywords.
- Broken Link Building.
- AI Citation Freshness Audit.
- Community Content Research.
- Page Traffic Opportunities.

Ahrefs' Agent is described as always-on, proactive, and autonomous: monitoring, alerting, surfacing opportunities, running analyses, building reports, and taking action.

## Business-type routing

Use these as routing hints when the user names their business type:

| Business type | Emphasize |
| --- | --- |
| Agency | Multi-client reports, pitch decks, landing pages, outreach sequences, Notion/Docs/HubSpot delivery when requested |
| Ecommerce | Catalog/page freshness, SKU/category SEO, AI visibility alerts, technical health, Linear/Slack tickets when requested |
| SaaS | Programmatic pages, product launches, lifecycle messaging, battlecards, conversion and funnel analysis |
| Enterprise | Multi-brand/multi-market reporting, copy variants, shared SEO/content governance, consolidated dashboards |
| Freelancer | Fast audits, briefs, niche validation, lightweight reports, recurring monitoring |

## Capability-to-workflow map

| Ahrefs' Agent capability | Agent AHHOG workflow |
| --- | --- |
| Build content calendar | `content-calendar` |
| Find keyword cannibalization | `keyword-cannibalization` |
| Create link building strategy | `link-building-strategy` |
| Audit technical health | `site-audit-triage`, `site-audit-discovery`, `site-audit-issue-fixer` |
| Analyze competitor backlinks | `backlink-analysis`, `link-intersect` |
| Find unlinked brand mentions | `unlinked-mentions` |
| Benchmark against industry | `competitor-overview`, `share-of-voice`, `e-e-a-t-audit` |
| Create client pitch decks | `client-pitch-deck` |
| Map content gaps | `content-gap` |
| Analyze SERP volatility | `rank-tracker-report`, `serp-features`, SerpApi if live SERP is approved |
| Track AI search visibility | `brand-radar-visibility`, `ai-mention-gap` |
| Monitor brand mentions | `brand-radar-visibility`, `unlinked-mentions` |
| Track share of voice | `share-of-voice` |
| Plan site migration | `migration-plan` |
| Map internal linking structure | `internal-link-map` |
| Compare domain authority | `competitor-overview`, `batch-analysis` |
| Forecast traffic | `traffic-forecast` |
| Anchor Text Analysis | `anchor-text-analysis` |
| Linkbait Opportunities | `linkbait-opportunities` |
| Trending Keyword Research | `trending-keyword-research` |
| SEO to AI Query Converter | `seo-to-ai-query-converter` |
| AI Brand Sentiment | `ai-brand-sentiment` |
| AI Bot & Web Analytics | `ai-bot-web-analytics` |
| AI Citation Freshness Audit | `ai-citation-freshness` |
| Community Content Research | `community-content-research` |
| Page Traffic Opportunities | `page-traffic-opportunities` |

## Integration behavior

The public Ahrefs' Agent page lists integrations across workspace, marketing, and data automation tools: Notion, Linear, Slack, Airtable, Fathom, HubSpot, Gong, WordPress, Mailchimp, Resend, Semrush, GitHub, Firehose, and Apify.

Agent AHHOG should not assume any of these are connected. Use them only if the user asks for that destination and the tool is available. Always save the local artifact first.

## Monitoring and automation behavior

When the user asks for "monitor", "alert", "always on", "watch this", "notify me", or "recurring":

1. Identify the metric/event/entity, threshold, cadence, owner, and destination.
2. Prefer native connected systems: PostHog dashboards/insights/annotations, Slack notifications, Linear/GitHub tickets, Notion reporting pages.
3. If no destination is connected, save a monitoring spec with exact queries, thresholds, and implementation steps.
4. Never invent that an alert has been configured. Report only what was actually created.

## App behavior

Ahrefs' Agent also advertises plug-and-play apps. In this skill, "app" means one of three things:

- A ready-to-run workflow spec, if the user wants strategy/ops.
- A dashboard/report configured in PostHog or another connected destination, if tools exist.
- Local code, only when the current workspace is a compatible app/repo and the user asks to build or modify it.

Do not create or deploy external apps without explicit user instruction and available tooling.
