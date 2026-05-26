# Agent AHHOG — Prompting Patterns

Real users don't ask for workflows by name. They say things like "my traffic is dropping" or "the competitor is killing us." This file maps common phrasings to the right workflow(s) and the right clarifying question(s).

---

## Diagnostic prompts ("something is wrong")

**"We're losing traffic" / "Traffic is dropping" / "Why is traffic down"**
→ Compose: `content-decay` + `rank-tracker-report` (if project exists) + `web-analytics-deep-dive` (if Web Analytics connected) + optionally `brand-radar-visibility` if AI search shift is plausible.

Clarify: time window of the decline; whether it's organic specifically or all channels.

**"We're not ranking" / "Our rankings dropped"**
→ Start with `rank-tracker-report` if a project exists. If not, `site-explorer-organic-keywords` over time + `site-explorer-keywords-history`.

Clarify: which keywords specifically, or "in general"?

**"The competitor is killing us"**
→ `competitor-overview` for the competitor + `content-gap` between target and competitor + `link-intersect`.

Clarify: which competitor, which market.

**"My site is slow / broken / has errors"**
→ `site-audit-triage`. If no Site Audit project, tell the user to create one in Ahrefs first.

---

## Opportunity prompts ("where can we grow")

**"What should we write about" / "Content ideas" / "What's working in our space"**
→ `content-gap` + `keyword-research`.

Clarify: competitor set, country, topical focus.

**"Where can we get links" / "Help us build links" / "Link strategy"**
→ Compose: `link-intersect` (best starting point) + `broken-link-building` + `unlinked-mentions`.

Clarify: 2–4 competitor domains for intersect; brand name for unlinked mentions.

**"AI search strategy" / "How do we show up in ChatGPT" / "LLM optimization"**
→ `brand-radar-visibility` + `ai-cited-pages` + `e-e-a-t-audit`.

Clarify: brand name, competitor brand names, primary assistants to track.

**"Quick wins" / "Easy SEO wins" / "Low-hanging fruit"**
→ `serp-features` (keywords ranking 2–10 with feature opportunities) + `content-decay` (recover lost traffic) + the easiest items from `site-audit-triage`.

---

## Brand and reputation prompts

**"Are people talking about us" / "Brand monitoring" / "Brand mentions"**
→ `brand-radar-visibility` (AI side) + `unlinked-mentions` (web side).

**"What does AI say about us"**
→ `brand-radar-visibility` with focus on the `brand-radar-ai-responses` qualitative read.

**"Our brand searches are down"**
→ This is a brand demand issue. Look at branded keyword performance in `site-explorer-organic-keywords` filtered to branded terms, plus `brand-radar-mentions-history`.

---

## Strategy and planning prompts

**"Audit our SEO" / "SEO review" / "Where do we stand"**
This is broad. Default composition:
- `competitor-overview` for own domain
- `site-audit-triage`
- Top 3 issues from `content-decay`
- Headline numbers from `backlink-analysis`
- Headline numbers from `brand-radar-visibility` if AI monitoring active

Output as a single executive briefing.

**"Plan for next quarter" / "SEO roadmap"**
→ All of the above + explicit prioritization by quarter. Don't try to fit everything; pick the highest-leverage 5–10 initiatives.

**"We're launching [product/site/brand]"**
→ Composition depends on stage. Pre-launch: `keyword-research` + competitor profiles. Launch: `site-audit-triage` + content-gap + Rank Tracker setup recommendation. Post-launch: monitoring composition.

---

## Tactical prompts

**"Find broken links / 404 backlinks"** → `broken-link-building`
**"Find unlinked mentions"** → `unlinked-mentions`
**"What keywords does X rank for"** → direct `site-explorer-organic-keywords` call
**"Compare these domains"** → `competitor-overview` for each, then synthesize
**"Internal linking"** → `internal-link-map`
**"Site migration"** → `migration-plan`

---

## Conversion & revenue prompts (need PostHog, usually both sources)

**"Which keywords actually make money" / "ROI of our SEO" / "ranking to revenue"**
→ `seo-to-conversion` (combined). Clarify the conversion event.

**"This page gets traffic but nobody converts" / "fix conversion on [page]"**
→ `landing-page-cro` (combined). Get the URL and conversion event.

**"Which blog posts are worth keeping" / "content ROI" / "what content drives signups"**
→ `content-roi` (combined). Clarify content section and conversion event.

**"Are our Ahrefs numbers real" / "estimated vs actual traffic"**
→ `channel-truth` (combined).

**"Who are our best customers and how do we get more like them"**
→ `audience-segment-seo` (combined).

**"Did our [launch / migration / campaign] work"**
→ `campaign-impact` (combined). Get the date and affected URLs.

**"Where do users drop off" / "funnel analysis" / "conversion funnel"**
→ PostHog-only. `read-data-schema` then `query-funnel`. No Ahrefs needed unless they also ask about traffic source.

**"Do visitors come back" / "retention" / "are users sticking"**
→ PostHog-only. `query-retention` / `query-lifecycle`.

**"Why are people leaving this page" / "watch some sessions"**
→ PostHog-only. `query-session-recordings-list` + `session-recording-summarize`.

---

## SerpApi prompts (live data — always confirm first)

These requests signal that historical Ahrefs data won't be sufficient. Before calling SerpApi, state the engine+query+credit cost and wait for explicit user approval.

**"What does the SERP look like right now for [keyword]"** / **"Is our featured snippet still showing"**
→ SerpApi `google` engine. Ahrefs has last crawl; user needs live. Confirm first.

**"What's trending right now"** / **"Is [topic] trending this week"** / **"Trending searches in Indonesia"**
→ SerpApi `google_trends` with `data_type: TIMESERIES`. Confirm first.

**"What's being written about us today"** / **"Recent news about [brand/competitor]"**
→ SerpApi `google_news`. Live news coverage Ahrefs doesn't index in real-time. Confirm first.

**"How do we rank on Bing / DuckDuckGo / YouTube for [keyword]"**
→ SerpApi with appropriate engine. Ahrefs is primarily Google. Confirm first.

**"What are people asking about [topic] on Google right now"** (live PAA)
→ SerpApi `google`, extract `people_also_ask[]`. Confirm first.

**"What ads is [competitor] running right now"**
→ SerpApi `google`, extract `ads[]`. Live ad intelligence. Confirm first.

**Key: "right now", "today", "live", "currently", "at this moment"** in the context of SERP data → strong signal that SerpApi may be needed. But still check if Ahrefs can satisfy it first before proposing SerpApi.

---

## How to route by source — four data sources

| What user mentions | Use |
| --- | --- |
| Rankings, keywords, backlinks, competitors, AI visibility, brand mentions in AI | **Ahrefs** |
| GSC performance, clicks, impressions, CTR, indexed pages, URL inspection, sitemaps | **GSC MCP** (if connected) → fallback to Ahrefs `gsc-*` |
| Conversions, funnels, signups, retention, sessions, cohorts, experiments, surveys | **PostHog** |
| Revenue, ROI, "which keywords convert", before/after a launch | **Ahrefs + PostHog** (combined workflow) |
| "Right now", "live SERP", "trending today", "news today", YouTube rankings, Bing | **SerpApi** (confirm first) |

**GSC-specific trigger phrases** that should route to GSC MCP (preferred) or Ahrefs `gsc-*` (fallback):
- "Is this page indexed?", "why isn't this page ranking?", "check if Google can crawl this" → `inspect_url_enhanced` or `batch_url_inspection`
- "What queries bring traffic to this page?" → `get_search_by_page_query`
- "Mobile vs desktop performance", "performance in Indonesia" → `get_advanced_search_analytics` with device/country filter
- "Compare last month vs this month", "impact of the algorithm update" → `compare_search_periods`
- "Sitemap errors", "submit my sitemap" → `list_sitemaps_enhanced`, `manage_sitemaps`
- "Content opportunities", "close to page 1" → `get_advanced_search_analytics` filtered to position 11–20

When SerpApi is relevant but not connected — don't mention it. Use Ahrefs and note the data is historical.
When GSC MCP is not connected — silently use Ahrefs `gsc-*`. Only explain the GSC MCP option if user asks about URL inspection or first-party GSC data.

---



- Mentions **rankings, keywords, backlinks, competitors, SERP, brand in AI** → Ahrefs
- Mentions **conversions, funnels, signups, retention, drop-off, sessions, cohorts, events** → PostHog
- Mentions **revenue, ROI, "worth it," "actually convert," before/after a launch** → both (combined workflow)

When only one source is connected, run that half and flag what's missing — never invent the other side's numbers.



**"Just give me the data"** — Push back gently. The data alone isn't useful. Offer: "I'll pull the data and give you the top findings — would you also like the raw export as CSV?"

**"Make us rank #1 for [keyword]"** — No tool can promise this. Reframe: "I can analyze what it would take to rank for [keyword] — let me run a SERP analysis and competitor breakdown for that term."

**"Is this a good keyword"** — Don't answer with a single metric. Look at volume + KD + intent fit + current ranking + competitive set, then recommend.

**"What's our SEO score"** — There's no universal SEO score. Offer a composite based on Site Audit health + traffic trend + DR + AI visibility, and label it as such.

**"Hide / disavow / penalize competitor"** — Decline. Agent AHHOG doesn't help with black hat or anti-competitive tactics.

---

## When the user is non-specific

If the user says something like "help me with SEO" or "do some marketing analysis" with no further context, ask **one** focused question to narrow scope:

> "Happy to help. To make sure I produce something useful instead of generic: which one is most pressing right now — diagnosing a problem (traffic/rank loss), finding opportunities (content/links), or auditing the current state (technical/authority)?"

Then route to the relevant composition above.

---

## Language and tone

Users in Indonesia, the broader Asia region, or other non-English-first contexts may write in mixed English/local language. Mirror the language they use. The reports themselves can be in either language — ask if it's not obvious from the request.

Keep tone direct and operator-like. No "great question!" preambles. No motivational SEO speak. Numbers, decisions, next actions.
