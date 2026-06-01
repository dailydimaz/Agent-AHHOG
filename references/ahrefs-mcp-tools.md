# Ahrefs MCP — Tool Reference

This is the map of which Ahrefs MCP tool to call for which job. Tool names below use the bare names as they appear in the Ahrefs MCP server. Depending on your client, they may be prefixed (e.g. `mcp__ahrefs__site-explorer-metrics` or `Ahrefs:site-explorer-metrics`). Match by the suffix.

The Ahrefs MCP server hosts ~125 tools. They group into the categories below.

**Ahrefs' Agent parity note.** Ahrefs' public Ahrefs' Agent page says Ahrefs' Agent uses full Ahrefs UI-parity access. This reference is only for the Ahrefs MCP surface available to Agent AHHOG. If an Ahrefs' Agent workflow needs a report/filter that is not listed here, call the Ahrefs MCP `doc` tool to check whether a newer endpoint is exposed. If it is not exposed, say so and compose the closest analysis from available tools.

---

## Site Explorer — competitor & domain intelligence

Use these for any "what does this domain look like" question — your own site, a competitor, or a prospect.

| Tool | Returns | Use when |
| --- | --- | --- |
| `site-explorer-metrics` | Headline metrics (traffic, traffic value, keywords, DR, backlinks) for a target | First call for any new domain analysis |
| `site-explorer-metrics-history` | Time series of the above | Trend analysis, traffic loss diagnosis |
| `site-explorer-metrics-by-country` | Metrics broken down by country | International SEO, market sizing |
| `site-explorer-domain-rating` | Current DR | Quick authority check |
| `site-explorer-domain-rating-history` | DR over time | Authority trajectory |
| `site-explorer-url-rating-history` | URL Rating over time | Page-level authority change |
| `site-explorer-organic-keywords` | Every keyword the target ranks for | Keyword inventory, gap analysis |
| `site-explorer-organic-competitors` | Domains sharing organic keywords | Find the real competitor set |
| `site-explorer-keywords-history` | Keyword count over time | Visibility expansion / contraction |
| `site-explorer-total-search-volume-history` | Total addressable search demand | Market size trajectory |
| `site-explorer-top-pages` | Top pages by traffic | What's actually working |
| `site-explorer-pages-by-traffic` | Page traffic distribution | Long tail vs head analysis |
| `site-explorer-pages-by-backlinks` | Pages ranked by inbound links | Best linkable assets |
| `site-explorer-pages-by-internal-links` | Internal link distribution | Find under-linked important pages |
| `site-explorer-pages-history` | Pages indexed over time | Content velocity |
| `site-explorer-paid-pages` | Pages running paid ads | Competitive ad intel |
| `site-explorer-crawled-pages` | Pages Ahrefs has crawled | Coverage check |
| `site-explorer-all-backlinks` | Every backlink to a target | Link profile audit |
| `site-explorer-broken-backlinks` | Backlinks pointing to 404s on the target | Broken link building / reclamation |
| `site-explorer-backlinks-stats` | Backlink counts and types | Quick link health check |
| `site-explorer-anchors` | Anchor text distribution | Anchor text audit, over-optimization check |
| `site-explorer-linked-anchors-internal` | Internal anchor text data | Internal linking audit |
| `site-explorer-linked-anchors-external` | External outbound anchor data | Outbound link review |
| `site-explorer-linked-domains` | Domains the target links out to | Outbound profile, link sculpting |
| `site-explorer-outlinks-stats` | Outbound link statistics | Outbound link health |
| `site-explorer-referring-domains` | Domains linking to the target | Referring domain inventory |
| `site-explorer-refdomains-history` | Referring domains over time | Link velocity, link loss |

**Parameter pattern.** Most Site Explorer tools accept `target` (domain or URL), `mode` (`domain`, `subdomain`, `prefix`, `exact`), `country`, and a date range. Pick `mode` carefully: `domain` covers the whole site; `prefix` is right for sections like `/blog/`; `exact` is right for a single URL.

---

## Keywords Explorer — keyword research

| Tool | Returns | Use when |
| --- | --- | --- |
| `keywords-explorer-overview` | Search volume, KD, CPC, SERP overview for one or many keywords | First call for any keyword question |
| `keywords-explorer-matching-terms` | Keyword ideas that contain the seed | Topic expansion, long-tail discovery |
| `keywords-explorer-related-terms` | "Also rank for" and semantically related | Topic clustering |
| `keywords-explorer-search-suggestions` | Autocomplete-style suggestions | Question intent, conversational queries |
| `keywords-explorer-volume-by-country` | Volume of one keyword across countries | International prioritization |
| `keywords-explorer-volume-history` | Search volume time series | Seasonality, trend confirmation |

---

## Content Explorer — content discovery (mentioned via Brand Radar siblings)

Used heavily inside content gap, decay, linkbait, community research, and authority workflows. If a Content Explorer-specific MCP endpoint is available in the current client, use it. If not, compose from Site Explorer, Keywords Explorer, Brand Radar, SERP overview, and social tools where available.

---

## Site Audit — technical SEO

| Tool | Returns | Use when |
| --- | --- | --- |
| `site-audit-projects` | List of Site Audit projects and summary stats | Discover which project to audit |
| `site-audit-issues` | All issues from the latest crawl, with severity | Generate the issue triage list |
| `site-audit-page-explorer` | Per-page audit data (status, indexability, performance) | Page-level diagnosis |
| `site-audit-page-content` | Extracted HTML/text of a crawled page | Content-level review |

Site Audit requires a project to already exist in the user's Ahrefs account. If no project exists for the target domain, tell the user — Agent AHHOG can't create projects through MCP.

---

## Rank Tracker — position monitoring

| Tool | Returns | Use when |
| --- | --- | --- |
| `rank-tracker-overview` | Tracked keyword rankings overview | Daily/weekly rank check |
| `rank-tracker-competitors-overview` | Tracked competitors' ranking summary | Competitive rank comparison |
| `rank-tracker-competitors-pages` | Competitor pages for tracked keywords | What page is winning a keyword |
| `rank-tracker-competitors-stats` | Aggregate competitor stats | Share of visibility |
| `rank-tracker-serp-overview` | SERP snapshot for a tracked keyword | SERP feature opportunities |

Like Site Audit, Rank Tracker needs a project. Confirm before calling.

---

## Brand Radar — AI search visibility & brand monitoring

This is the newest layer and the one Agent AHHOG leans on heaviest for AI search work.

| Tool | Returns | Use when |
| --- | --- | --- |
| `brand-radar-mentions-overview` | Brand mention count across AI assistants | "Are we showing up in ChatGPT/Perplexity?" |
| `brand-radar-mentions-history` | Mention count over time | AI visibility trend |
| `brand-radar-impressions-overview` | Impressions in AI responses | Reach in AI answers |
| `brand-radar-impressions-history` | Impressions trend | AI reach trajectory |
| `brand-radar-sov-overview` | Share of voice vs competitors in AI | Competitive AI positioning |
| `brand-radar-sov-history` | SoV over time | AI positioning trend |
| `brand-radar-ai-responses` | The actual AI responses mentioning the brand | Read the citations |
| `brand-radar-cited-pages` | Which of your pages get cited by AI | AI-citation page inventory |
| `brand-radar-cited-domains` | Which domains AI cites alongside yours | AI co-citation network |

Each of the above also has an `-entities` variant for tracking specific entity strings rather than brand names. Use the `-entities` variants when comparing product names, people, or topics that aren't the brand itself.

---

## Web Analytics — first-party traffic data

Available when the user has connected their site to Ahrefs Web Analytics.

| Tool | Returns | Use when |
| --- | --- | --- |
| `web-analytics-stats` | Aggregate visits / pageviews | Top-line traffic |
| `web-analytics-chart` | Time series | Trend |
| `web-analytics-top-pages` | Most visited pages | Top performers |
| `web-analytics-entry-pages` | Where users enter | Funnel top |
| `web-analytics-exit-pages` | Where users leave | Friction diagnosis |
| `web-analytics-sources` | Traffic source breakdown | Channel mix |
| `web-analytics-source-channels` | Grouped channels (organic, paid, social, direct) | Channel attribution |
| `web-analytics-referrers` | Specific referrer breakdown | Referrer-level traffic |
| `web-analytics-countries` | Visits by country | Geo split |
| `web-analytics-cities` | Visits by city | City-level demand |
| `web-analytics-continents` | Visits by continent | Continent split |
| `web-analytics-devices` | Visits by device type | Mobile vs desktop |
| `web-analytics-browsers` | Browser breakdown | Browser support priorities |
| `web-analytics-browser-versions` | Browser version breakdown | Compatibility planning |
| `web-analytics-operating-systems` | OS breakdown | OS audience |
| `web-analytics-operating-systems-versions` | OS version breakdown | Version-specific issues |
| `web-analytics-languages` | Visitor language | Localization signal |
| `web-analytics-utm-params` | UTM-tagged traffic | Campaign attribution |

Each of these also has a `-chart` variant that returns the same dimension as a time series.

---

## Google Search Console insights (via Ahrefs GSC integration)

Available when the user has connected GSC to Ahrefs.

> **Priority note:** if the GSC MCP server (`gscServer:`) is also connected, **use that instead** — it provides more accurate first-party data, URL inspection, indexing tools, and finer filters. Use these Ahrefs `gsc-*` tools only as a fallback when the GSC MCP is not available. See `references/gsc-mcp-tools.md`.

| Tool | Returns | Use when |
| --- | --- | --- |
| `gsc-keywords` | GSC keyword table (clicks, impressions, CTR, position) | First-party query data |
| `gsc-keyword-history` | Time series for a keyword | Query-level trend |
| `gsc-pages` | GSC page table | First-party page performance |
| `gsc-page-history` | Page time series | Page-level trend |
| `gsc-pages-history` | Indexed-page count over time | Coverage trend |
| `gsc-anonymous-queries` | Keywords ranking but not reported by GSC | Hidden query discovery |
| `gsc-ctr-by-position` | CTR curve by position | Snippet optimization sizing |
| `gsc-performance-history` | Clicks/impressions/CTR over time | Overall search performance |
| `gsc-performance-by-position` | Performance grouped by rank | Position-tier analysis |
| `gsc-performance-by-device` | Performance by device | Device-tier diagnosis |
| `gsc-metrics-by-country` | Clicks by country | Geo demand vs delivery |
| `gsc-positions-history` | Keyword count per position range | Visibility band trend |

---

## SERP

| Tool | Returns | Use when |
| --- | --- | --- |
| `serp-overview` | Top results for a keyword in a country | SERP analysis, ranking opportunity |

---

## Batch analysis

| Tool | Returns | Use when |
| --- | --- | --- |
| `batch-analysis` | Metrics for many URLs/domains at once | Bulk competitive scans, link prospect qualification |

Use `batch-analysis` when you would otherwise call `site-explorer-metrics` more than ~5 times in a row.

---

## Management — what's configured in the user's account

| Tool | Returns | Use when |
| --- | --- | --- |
| `management-projects` | All projects in the user's account | Discover what's available before asking the user |
| `management-project-keywords` | Tracked keywords for a project | Confirm what's being tracked |
| `management-project-competitors` | Tracked competitors for a project | Know who is in the competitor set |
| `management-keyword-list-keywords` | A saved keyword list | Reuse the user's lists |
| `management-locations` | Available geo targets | Confirm country codes |
| `management-brand-radar-reports` | Brand Radar reports | Check existing AI monitoring |
| `management-brand-radar-prompts` | Custom Brand Radar prompts | Reuse AI tracking prompts |

**Pro tip:** Call `management-projects` early in any session where you don't know the user's setup. It tells you which domains they care about, which competitors they track, and which markets they monitor — most of the clarifying questions become unnecessary.

---

## Social media (via Ahrefs Social integration)

| Tool | Returns | Use when |
| --- | --- | --- |
| `social-media-channels` | Connected social channels | Inventory |
| `social-media-channel-metrics` | Follower history | Channel growth |
| `social-media-posts` | Posts with filtering | Content audit |
| `social-media-post-metrics` | Engagement per post | Performance breakdown |
| `social-media-authors` | Who posts | Author audit |
| `social-media-activity-history` | Activity log | Cadence audit |

---

## Subscription & limits

| Tool | Returns | Use when |
| --- | --- | --- |
| `subscription-info-limits-and-usage` | Plan limits and current usage | Before running a huge job — confirm you won't blow the budget |

Always check this if a workflow requires >50 MCP calls.

---

## Choosing tools — decision tree

When deciding which tool to call:

1. **Is this about a known competitor / your own domain?** → Site Explorer
2. **Is this a keyword question (volume, KD, ideas)?** → Keywords Explorer
3. **Is this about technical issues on the user's site?** → Site Audit
4. **Is this about tracked rankings?** → Rank Tracker
5. **Is this about AI search / brand mentions in AI?** → Brand Radar
6. **Is this about first-party site traffic or GSC performance?** → GSC MCP (`gscServer:`) if connected; otherwise Web Analytics or Ahrefs `gsc-*` as fallback
7. **Is this a list of URLs/domains to qualify in bulk?** → Batch Analysis
8. **Don't know what the user has set up?** → Management

If nothing fits, the request probably isn't an Ahrefs task. Say so rather than forcing a tool.

---

## Full documentation

For exhaustive parameter schemas and edge cases, the Ahrefs MCP server exposes a `doc` tool that returns the full API v3 documentation. Call it when you need a parameter you're not sure about — but don't load it preemptively, it's large.
