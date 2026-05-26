# Agent AHHOG — Workflow Recipes

Each recipe below is a named workflow that mirrors one of the skills in Agent AHHOG's library. The format is the same throughout:

- **Triggers** — phrases that should activate this workflow
- **Inputs needed** — what to confirm with the user
- **MCP calls** — the sequence (or parallel set) of Ahrefs MCP tools to call
- **Synthesis** — how to combine the data
- **Output** — the structure of the finished artifact

When a user request maps clearly to one workflow, follow it. When it spans two or three, compose them — but keep the output as a single coherent report.

---

## content-gap

**Triggers:** "content gap", "keywords competitors rank for", "topics we're missing", "what should we write about", "where competitors beat us"

**Inputs needed:** target domain, 2–5 competitor domains, country.

**MCP calls (in parallel):**
1. `site-explorer-organic-keywords` for target → keyword inventory
2. `site-explorer-organic-keywords` for each competitor → competitor inventories
3. `site-explorer-organic-competitors` for target → confirm the right competitor set (if user-provided list looks off)

**Synthesis:**
- Find keywords where every competitor ranks (top 20) and target does not, or ranks beyond position 30.
- Filter by minimum search volume (e.g., ≥ 100/mo) and reasonable KD (e.g., ≤ 40 unless target's DR can support more).
- Group keywords into topical clusters — don't deliver 500 isolated keywords, deliver 15–25 clusters.
- For each cluster: total volume, avg KD, example SERP, suggested page format (guide, comparison, calculator, etc.).

**Output:** Markdown report. TL;DR with top 5 clusters by total volume. Findings section with one cluster per subsection. Prioritized action list = "write these N pages, in this order."

---

## keyword-cannibalization

**Triggers:** "cannibalization", "pages competing for the same keyword", "we have multiple pages ranking", "fix cannibalization"

**Inputs needed:** target domain, country.

**MCP calls:**
1. `site-explorer-organic-keywords` for target with `mode: domain` → full keyword list with the ranking URL per keyword

**Synthesis:**
- For each keyword that appears in the results with **more than one ranking URL on the target**, flag it.
- Rank cannibalization severity by: keyword search volume × number of cannibalizing pages × difference between best and worst rank.
- Cluster cannibalizations by topic — often the same set of pages is fighting over a topic cluster, not isolated keywords.
- Recommend a resolution for each cluster: consolidate to canonical, redirect, differentiate intent, or delete.

**Output:** Markdown report with a table of cannibalization clusters (keyword group, cannibalizing URLs, volume, recommended action) and a prioritized fix list.

---

## content-decay

**Triggers:** "declining pages", "traffic loss", "what content needs a refresh", "find pages losing traffic"

**Inputs needed:** target domain, comparison window (default: last 6 months vs prior 6 months).

**MCP calls:**
1. `site-explorer-top-pages` for target, current window
2. `site-explorer-top-pages` for target, prior window
3. (Optional) `gsc-pages` and `gsc-page-history` for first-party confirmation if GSC is connected

**Synthesis:**
- Match URLs across the two windows. Compute % change in traffic.
- Filter to pages with meaningful prior traffic (e.g., ≥ 500/mo) that declined ≥ 30%.
- For top decliners, optionally pull `site-explorer-organic-keywords` filtered to that URL to see which keywords slipped.
- Diagnose pattern: rank loss, SERP feature steal, CTR drop, declining search volume — call out which.

**Output:** Markdown report with ranked decay list (URL, prior traffic, current traffic, % change, suspected cause, recommended refresh action).

---

## site-audit-triage

**Triggers:** "technical SEO audit", "site audit", "fix list", "health score", "what's broken on my site"

**Inputs needed:** Ahrefs project for the target domain (confirm via `management-projects`).

**MCP calls:**
1. `management-projects` → confirm project exists
2. `site-audit-projects` for project details
3. `site-audit-issues` → all issues with severity
4. (Optional) `site-audit-page-explorer` for the worst-affected pages

**Synthesis:**
- Group issues by category (crawlability, indexability, performance, content, structured data, mobile, links).
- For each issue: severity, # of pages affected, sample affected URLs, fix difficulty, expected impact.
- Compute a health score: weight critical issues 5x, warnings 2x, notices 1x; normalize against site size.
- Pick the top 10 fixes by (severity × # pages × ease) and put them in the prioritized action list.

**Output:** Markdown report. Health score in TL;DR. Findings grouped by category. Prioritized fix list with code/CMS-level recommendations where possible.

---

## backlink-analysis

**Triggers:** "who links to us", "link profile review", "backlink audit", "are our backlinks good"

**Inputs needed:** target domain.

**MCP calls:**
1. `site-explorer-backlinks-stats` → top-line link counts
2. `site-explorer-referring-domains` → unique referring domains
3. `site-explorer-refdomains-history` → link velocity
4. `site-explorer-anchors` → anchor text distribution
5. `site-explorer-all-backlinks` with `limit` reasonable (100–500) sorted by DR → quality sample

**Synthesis:**
- Profile the link graph: total RDs, DR distribution, dofollow/nofollow split, country distribution.
- Anchor text health: branded vs exact-match vs partial vs generic. Flag over-optimization (>10% exact-match for commercial terms is usually risky).
- Velocity: is link acquisition accelerating, flat, or declining?
- Quality flags: spike of low-DR links, link networks, mass-produced anchors.

**Output:** Markdown report with link profile dashboard (counts and distributions), risk flags section, and prioritized action list (disavow candidates, anchor diversification, gaps to fill).

---

## broken-link-building

**Triggers:** "broken backlinks", "link reclamation", "404 link opportunities", "broken link building"

**Inputs needed:** target domain (your own, or a competitor whose broken links you'd target).

**MCP calls:**
1. `site-explorer-broken-backlinks` for target → backlinks pointing to 404s

**Synthesis:**
- For each broken backlink: source page DR, source page traffic, anchor text, target URL (404), suggested replacement URL on the user's site (if doing competitor broken link building) or a redirect candidate (if doing own-site reclamation).
- Rank by linking domain DR × source page traffic.
- For competitor broken link building, propose outreach template per cluster of similar broken targets.

**Output:** Table-heavy report. Columns: linking domain, source URL, source DR, broken target, suggested replacement, outreach template hook.

---

## link-intersect

**Triggers:** "who links to competitors but not us", "link prospecting", "link intersect", "find link prospects"

**Inputs needed:** target domain, 2–4 competitor domains.

**MCP calls (in parallel):**
1. `site-explorer-referring-domains` for target
2. `site-explorer-referring-domains` for each competitor

**Synthesis:**
- Compute the intersection of competitors' referring domains minus target's referring domains.
- Filter by DR (typically ≥ 30) and exclude obvious non-targets (social, search engines, irrelevant niches).
- Score each prospect by: # of competitors linking, max DR of linking pages, traffic of linking pages.
- For top 20–50, draft an outreach angle: what content gap on the prospect's site you fill that the competitors do.

**Output:** Prospects table (domain, DR, # competitors linking, sample linking pages, outreach angle). Prioritized list of the top 20.

---

## unlinked-mentions

**Triggers:** "brand mentions without links", "mention reclamation", "unlinked mentions", "people talking about us without linking"

**Inputs needed:** brand name(s), target domain.

**MCP calls:**
1. `brand-radar-mentions-overview` and `brand-radar-cited-pages` for the brand
2. Cross-reference against `site-explorer-referring-domains` to identify which mentioning domains don't already link

**Synthesis:**
- For each unlinked mentioning page: source DR, page traffic, mention context, suggested anchor text and target URL.
- Rank by source DR × page traffic.
- Group by outreach approach: editorial mentions (request a link), forum/community mentions (lower-effort comment), press mentions (PR follow-up).

**Output:** Outreach-ready list with mention URL, anchor suggestion, target page suggestion, outreach template.

---

## serp-features

**Triggers:** "featured snippet opportunities", "SERP feature wins", "people also ask", "FAQ opportunities"

**Inputs needed:** target domain, country.

**MCP calls:**
1. `site-explorer-organic-keywords` for target where position is 2–10 (close to top)
2. For top candidates, `serp-overview` to see which features are present on those SERPs

**Synthesis:**
- Identify keywords where the target ranks 2–10 AND a SERP feature exists (featured snippet, PAA, video, image pack).
- For each: what feature is present, what page currently owns it, what format would let the target win it.
- Cluster by content type required (definition snippet, list snippet, table, video).

**Output:** Opportunity table with recommended on-page changes per cluster. Top 10 prioritized.

---

## brand-radar-visibility

**Triggers:** "AI search visibility", "ChatGPT mentions", "Perplexity citations", "are we showing up in AI", "LLM visibility"

**Inputs needed:** brand name, competitor brand names, AI assistants to track (default: all).

**MCP calls:**
1. `brand-radar-mentions-overview` for the brand
2. `brand-radar-mentions-history` for trend
3. `brand-radar-impressions-overview` and `-history`
4. `brand-radar-sov-overview` and `-history` (with competitors)
5. `brand-radar-ai-responses` (sample of actual responses for qualitative read)

**Synthesis:**
- Mention count, impression count, SoV — current and trend.
- Compare across AI assistants — where is visibility strongest/weakest?
- Read the AI responses qualitatively: is the brand positioned accurately? Are competitors mentioned alongside? Are AI assistants citing outdated info?

**Output:** Dashboard-style report. TL;DR with the SoV number and trend direction. Findings include per-assistant breakdown and a "what AI says about us" qualitative section. Action list = content/PR actions to improve AI visibility.

---

## share-of-voice

**Triggers:** "share of voice", "brand benchmark", "competitive brand visibility"

**Inputs needed:** brand, competitor brands, market.

**MCP calls:**
1. `brand-radar-sov-overview` and `brand-radar-sov-history` (with all competitors)
2. (Optional) `site-explorer-metrics` for each competitor for traditional search SoV cross-check
3. (Optional) `rank-tracker-competitors-overview` if Rank Tracker project exists

**Synthesis:**
- AI SoV current state and trend.
- Organic SoV via shared keywords / total search demand captured.
- Identify movers — which competitor gained SoV in the last period, which lost.

**Output:** SoV scorecard with current %, change vs prior period, leader and laggard analysis. Recommendation list per competitor.

---

## ai-cited-pages

**Triggers:** "which pages get cited by AI", "AI citation audit", "what content is AI quoting"

**Inputs needed:** target domain.

**MCP calls:**
1. `brand-radar-cited-pages` for target
2. (Optional) `brand-radar-ai-responses` to see context of citations

**Synthesis:**
- Which pages get cited, how often, by which assistants.
- What characteristics do cited pages share — structure, recency, depth, schema?
- What pages should be cited but aren't (compare to top-traffic pages)?

**Output:** Cited-page inventory + recommendations for non-cited but should-be-cited pages (often: refresh date, add structured data, expand answers to common questions).

---

## competitor-overview

**Triggers:** "competitor analysis", "what's working for [competitor]", "study this competitor"

**Inputs needed:** competitor domain, country.

**MCP calls (in parallel):**
1. `site-explorer-metrics` → headline
2. `site-explorer-metrics-history` → trend
3. `site-explorer-top-pages` → what's working
4. `site-explorer-organic-keywords` (top 100) → ranking inventory
5. `site-explorer-referring-domains` (top 100) → link profile
6. `site-explorer-organic-competitors` → who they compete with

**Synthesis:**
- Headline: DR, traffic, traffic value, keyword count.
- Top-performing pages with traffic, top keywords, and inbound link counts.
- Trend: gaining or losing? In which keyword segments?
- Link profile snapshot.

**Output:** Competitor briefing document. Useful as a deliverable to the user's marketing team or as input to other workflows (content gap, link intersect).

---

## keyword-research

**Triggers:** "keyword ideas", "find keywords for", "search volume", "is this a good keyword", "KD analysis"

**Inputs needed:** seed keyword(s), country.

**MCP calls:**
1. `keywords-explorer-overview` for the seed(s)
2. `keywords-explorer-matching-terms` and `keywords-explorer-related-terms` for expansion
3. `keywords-explorer-search-suggestions` for question-form intent

**Synthesis:**
- Cluster suggested keywords by topic / intent (informational, navigational, commercial, transactional).
- For each cluster: total volume, avg KD, sample SERP, recommended page type.
- Filter out: zero-volume, super-high-KD without business justification, irrelevant by manual review.

**Output:** Keyword research deliverable — cluster table, top picks list, page format recommendations.

---

## programmatic-seo

**Triggers:** "programmatic SEO", "template pages", "pSEO", "modifier keywords"

**Inputs needed:** core entity (e.g., "best CRM for", "[city] dentist"), modifier set or seed pattern.

**MCP calls:**
1. `keywords-explorer-matching-terms` with the core pattern as seed
2. `keywords-explorer-overview` for the result set in bulk to attach volume + KD
3. For top variants, `serp-overview` to confirm the SERP supports a programmatic page

**Synthesis:**
- Build the modifier dictionary: every modifier value that has search volume (cities, products, categories, attributes).
- Group into a matrix: rows = entities, columns = modifiers, cells = projected target keyword + volume.
- Identify the highest-volume rows/columns to prioritize.
- Recommend page template structure (H1 pattern, sections, schema).

**Output:** Programmatic keyword matrix (often a CSV) + page template specification.

---

## web-analytics-deep-dive

**Triggers:** "where is traffic coming from", "country breakdown", "device split", "traffic sources"

**Inputs needed:** Ahrefs Web Analytics connected for the target.

**MCP calls:**
1. `web-analytics-stats` and `web-analytics-chart` → top-line
2. `web-analytics-source-channels` → channel mix
3. `web-analytics-countries` and `web-analytics-devices` → audience
4. `web-analytics-top-pages` and `web-analytics-entry-pages` → page performance

**Synthesis:**
- Traffic mix and trend.
- Where the growth (or loss) is concentrated.
- Audience composition.
- Top performing pages.

**Output:** Traffic dashboard report.

---

## rank-tracker-report

**Triggers:** "how are we ranking", "position changes", "tracked keywords", "ranking report"

**Inputs needed:** Rank Tracker project for target.

**MCP calls:**
1. `management-projects` to confirm
2. `rank-tracker-overview`
3. `rank-tracker-competitors-overview` if competitors are tracked

**Synthesis:**
- Visibility score, avg position, top movers (winners and losers).
- Competitor comparison if available.
- Keyword groups by performance tier.

**Output:** Ranking report with TL;DR (visibility score + biggest movers), keyword table, competitor comparison.

---

## e-e-a-t-audit

**Triggers:** "authority audit", "trust signals", "E-E-A-T", "expertise authority trust review"

**Inputs needed:** target domain, topical area.

**MCP calls:**
1. `site-explorer-metrics` and `site-explorer-domain-rating-history` → authority baseline
2. `site-explorer-referring-domains` → who vouches for us
3. `brand-radar-mentions-overview` and `brand-radar-cited-domains` → external recognition
4. `site-audit-page-content` for sample pages to inspect author info, citations, dates

**Synthesis:**
- Experience signals: first-hand language, original data, author bio depth.
- Expertise signals: credentials, citations to authoritative sources.
- Authoritativeness: external mentions, link sources, AI citations.
- Trust: HTTPS, contact info, transparency, reviews.

**Output:** E-E-A-T scorecard with each pillar rated (color-coded), specific page-level findings, and a prioritized improvement list.

---

## migration-plan

**Triggers:** "site migration", "URL change plan", "redirect mapping", "moving to new domain"

**Inputs needed:** source domain/structure, target domain/structure, migration scope.

**MCP calls:**
1. `site-explorer-top-pages` for source → pages that must not be broken
2. `site-explorer-organic-keywords` for source → keywords that must not be lost
3. `site-explorer-all-backlinks` for source → link equity that must be preserved
4. (Optional) `site-audit-page-explorer` for technical baseline

**Synthesis:**
- Inventory of must-preserve URLs (by traffic, by inbound links, by keyword value).
- Recommended redirect map (1-to-1 where possible).
- Risk register: pages where the equivalent on the new site is unclear.
- Pre/post checks to run.

**Output:** Migration plan document with the URL inventory, redirect map (often as CSV), and risk register.

---

## internal-link-map

**Triggers:** "internal linking", "link equity flow", "internal link opportunities", "structure my site links"

**Inputs needed:** target domain.

**MCP calls:**
1. `site-explorer-pages-by-internal-links` → distribution
2. `site-explorer-linked-anchors-internal` → anchor patterns
3. `site-explorer-top-pages` → traffic destinations
4. `site-explorer-pages-by-traffic` → all valuable pages

**Synthesis:**
- Find pages with high traffic potential but low internal links pointing at them.
- Find orphan pages (very low internal links).
- Identify over-linked pages that could share authority.
- Anchor text patterns — too repetitive? Too generic?

**Output:** Internal linking opportunity report. For each opportunity: target page, recommended source pages to link from, suggested anchor text.

---

## Composing workflows

When a request spans multiple workflows, run them in sequence and merge:

- "Audit my site and tell me what to fix" → `site-audit-triage` + `content-decay` + `internal-link-map`
- "Why are we losing traffic" → `content-decay` + `rank-tracker-report` + `web-analytics-deep-dive` + (if available) `brand-radar-visibility` for the AI search loss factor
- "Beat competitor X" → `competitor-overview` + `content-gap` + `link-intersect`
- "AI search strategy" → `brand-radar-visibility` + `ai-cited-pages` + `e-e-a-t-audit`

Compose, don't pad — the user wants the merged conclusion, not three reports stapled together.
