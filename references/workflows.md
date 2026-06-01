# Agent AHHOG — Workflow Recipes

Each recipe below is a named workflow that mirrors one of the skills in Agent AHHOG's library, including the newer public Ahrefs' Agent skill/app surface where MCP access allows it. The format is the same throughout:

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

## content-calendar

**Triggers:** "build a content calendar", "editorial roadmap", "what should we publish this quarter", "content plan"

**Inputs needed:** target domain, country, planning horizon (default: next 90 days), competitors or topic focus.

**MCP calls:**
1. `management-projects` to discover tracked domain/competitors if not provided.
2. `site-explorer-top-pages` and `site-explorer-organic-keywords` for the target → current strengths.
3. `site-explorer-organic-keywords` for competitors → gaps.
4. `keywords-explorer-matching-terms`, `related-terms`, and `search-suggestions` for seed expansion.
5. (Optional) PostHog conversion data for existing content → prioritize by downstream value.

**Synthesis:**
- Cluster topics by intent and funnel stage.
- Balance refreshes, new content, comparison pages, programmatic pages, and linkable assets.
- Prioritize by traffic potential × ranking feasibility × business value; use PostHog conversions where available.
- Assign each item a suggested publish week, page type, primary keyword, secondary keywords, owner role, and CTA.

**Output:** Markdown calendar strategy plus CSV columns: `week,publish_date,page_type,title,primary_keyword,cluster,volume,kd,intent,cta,priority,notes`.

---

## link-building-strategy

**Triggers:** "create link building strategy", "link plan", "build links", "off-page strategy"

**Inputs needed:** target domain, 2-4 competitors, country or market, outreach constraints.

**MCP calls:**
1. `site-explorer-backlinks-stats`, `referring-domains`, and `anchors` for target → baseline.
2. `site-explorer-referring-domains` for competitors → link intersect opportunities.
3. `site-explorer-broken-backlinks` for target and/or competitors → reclamation and broken-link prospects.
4. `brand-radar-mentions-overview` / `cited-pages` plus target refdomains → unlinked mentions.
5. `site-explorer-pages-by-backlinks` for competitors → linkbait patterns.

**Synthesis:**
- Split strategy into lanes: reclaim, intersect, unlinked mentions, linkable assets, digital PR, partner/vendor links.
- Score each lane by expected links, DR/traffic quality, effort, and time-to-impact.
- Recommend outreach angles and assets to create before outreach.

**Output:** Link building plan with a 30/60/90-day roadmap and CSV prospect list where useful.

---

## ai-mention-gap

**Triggers:** "AI mention gap", "competitors show up in AI but we don't", "LLM visibility gap", "AI answer gap"

**Inputs needed:** brand/entity, competitor brands/entities, market, assistant/report scope if known.

**MCP calls:**
1. `brand-radar-mentions-overview` and `mentions-history` for brand and competitors.
2. `brand-radar-sov-overview` and `sov-history` with competitors.
3. `brand-radar-ai-responses` for sample prompts/responses where competitors appear.
4. `brand-radar-cited-domains` and `cited-pages` to find sources AI assistants trust.

**Synthesis:**
- Identify prompts/topics where competitors appear and target does not.
- Extract common cited domains/pages that support competitor visibility.
- Translate gaps into content, PR, digital authority, and citation-building actions.

**Output:** AI mention gap report with prompt/topic gaps, cited-source gaps, competitor patterns, and prioritized actions.

---

## anchor-text-analysis

**Triggers:** "anchor text analysis", "anchor text audit", "over-optimized anchors", "internal anchor text"

**Inputs needed:** target domain or URL, external vs internal scope, country if organic impact matters.

**MCP calls:**
1. `site-explorer-anchors` → external anchor distribution.
2. `site-explorer-linked-anchors-internal` → internal anchor distribution.
3. `site-explorer-all-backlinks` sample → source quality/context.
4. `site-explorer-organic-keywords` for commercial keyword overlap.

**Synthesis:**
- Classify anchors as branded, URL, generic, topical, partial-match, exact-match, or irrelevant.
- Flag risk where exact commercial anchors are unusually concentrated or from weak/referring domains.
- Find missed internal-link opportunities where high-value pages lack descriptive internal anchors.

**Output:** Anchor text scorecard plus recommended internal anchor changes and external-risk notes.

---

## linkbait-opportunities

**Triggers:** "linkbait opportunities", "linkable assets", "what content earns links", "digital PR ideas"

**Inputs needed:** target domain, competitors, market/topic.

**MCP calls:**
1. `site-explorer-pages-by-backlinks` for competitors → their most-linked assets.
2. `site-explorer-referring-domains` for those competitor assets → link source quality.
3. `keywords-explorer-matching-terms` / `related-terms` for topic demand.
4. (Optional) `site-explorer-top-pages` for target → assets that already have traction.

**Synthesis:**
- Identify repeatable asset patterns: data studies, calculators, templates, original research, statistics pages, benchmarks, glossaries.
- Score ideas by linkability, search demand, production effort, and fit with target authority.
- Propose outreach targets and hooks for each asset.

**Output:** Linkable-asset roadmap with 5-15 ideas, proof from competitor links, and promotion plan.

---

## site-audit-discovery

**Triggers:** "site audit discovery", "what Ahrefs projects do we have", "crawl coverage", "find site audit setup"

**Inputs needed:** target domain if known.

**MCP calls:**
1. `management-projects` → project inventory and tracked domains.
2. `site-audit-projects` → crawl projects, health, last crawl.
3. `site-audit-issues` for the target project → current issue inventory.
4. `site-audit-page-explorer` → crawl coverage and affected page types.

**Synthesis:**
- Confirm whether the target has a project and recent crawl.
- Summarize crawl freshness, scope, health, and major issue categories.
- Identify setup gaps: missing project, stale crawl, wrong scope, ignored subdomains, missing key templates.

**Output:** Site Audit setup and discovery report with next crawl/configuration actions.

---

## site-audit-issue-fixer

**Triggers:** "fix site audit issues", "site audit issue fixer", "create tickets for these SEO issues", "implement technical SEO fixes"

**Inputs needed:** target project/domain, issue scope, whether to create tickets or edit local code if the website repo is present.

**MCP calls:**
1. `management-projects` and `site-audit-projects` → confirm project.
2. `site-audit-issues` → issue list and severity.
3. `site-audit-page-explorer` and `site-audit-page-content` for representative affected pages.

**Synthesis:**
- Deduplicate issues by underlying template/root cause, not by URL.
- For each issue group, produce: cause, affected template/pages, fix owner, implementation steps, validation check.
- If the user's local workspace contains the affected website code and the request implies implementation, inspect the code and patch clear fixes. If mapping from crawl URL to local code is ambiguous, create tickets/specs instead of guessing.

**Output:** Fix plan, GitHub/Linear tickets if requested, and local code patches only when the repo mapping is clear.

---

## trending-keyword-research

**Triggers:** "trending keywords", "rising keywords", "what topics are growing", "trend research"

**Inputs needed:** seed topic(s), country, time window, whether live trend data is required.

**MCP calls:**
1. `keywords-explorer-matching-terms` / `related-terms` / `search-suggestions` → candidate set.
2. `keywords-explorer-volume-history` for candidates → historical trend.
3. `keywords-explorer-volume-by-country` if international demand matters.
4. If the user explicitly needs live/current trending data, propose SerpApi `google_trends` with credit cost and wait for approval.

**Synthesis:**
- Separate evergreen high-volume topics from breakout/rising topics.
- Flag seasonality vs genuine growth.
- Prioritize rising topics that have low competition and business fit.

**Output:** Trending keyword report with trend classification and content recommendations.

---

## seo-to-ai-query-converter

**Triggers:** "SEO to AI query converter", "turn keywords into AI prompts", "what AI questions should we monitor", "AI search prompts"

**Inputs needed:** keyword set or topic, brand/entity, competitors, market.

**MCP calls:**
1. `keywords-explorer-overview` and keyword expansion tools if the keyword set is not supplied.
2. `brand-radar-ai-responses` for existing prompt/response patterns.
3. `management-brand-radar-prompts` if available → reuse existing monitored prompts.

**Synthesis:**
- Convert keywords into natural AI questions across intent types: recommendations, comparisons, how-to, alternatives, best-for, pricing, troubleshooting.
- Include brand-present, competitor-present, and category-generic prompts.
- Group prompts by funnel stage and expected response type.

**Output:** AI query/prompt library suitable for Brand Radar monitoring, with priority and expected citation targets.

---

## ai-brand-sentiment

**Triggers:** "AI brand sentiment", "what does AI think of us", "brand perception in AI", "sentiment in ChatGPT/Perplexity"

**Inputs needed:** brand/entity, competitors if benchmarking, market/report scope.

**MCP calls:**
1. `brand-radar-ai-responses` → qualitative response sample.
2. `brand-radar-mentions-overview` and `sov-overview` → quantitative context.
3. `brand-radar-cited-domains` and `cited-pages` → evidence sources shaping perception.

**Synthesis:**
- Classify responses as positive, neutral, mixed, negative, outdated, or incorrect.
- Extract recurring descriptors, claimed strengths/weaknesses, missing differentiators, and misinformation.
- Connect sentiment problems to citation/source problems and content fixes.

**Output:** AI sentiment report with message corrections, citation targets, and content/PR actions.

---

## ai-bot-web-analytics

**Triggers:** "AI bot analytics", "AI crawler traffic", "ChatGPT referrals", "AI search web analytics", "AI bots and web analytics"

**Inputs needed:** target site/project, time window, whether Ahrefs Web Analytics, PostHog, or server logs are available.

**MCP calls:**
1. Ahrefs `web-analytics-referrers`, `source-channels`, `top-pages`, and charts → AI referrers and landing pages.
2. PostHog `read-data-schema` then `query-trends` / HogQL if user-agent, referrer, or UTM properties capture AI traffic.
3. `brand-radar-cited-pages` to compare AI-cited pages with AI-referred landing pages.

**Synthesis:**
- Separate AI assistant referrals, AI search visibility, and AI crawler/bot visits if the data supports it.
- Identify pages that get cited but receive little referral traffic, and pages receiving AI traffic that under-convert.
- Note tracking limitations explicitly; many analytics tools filter bots differently.

**Output:** AI traffic and bot visibility report with tracking recommendations and page-level actions.

---

## ai-citation-freshness

**Triggers:** "AI citation freshness", "stale AI citations", "outdated pages AI cites", "AI cites old content"

**Inputs needed:** brand/domain, market, freshness threshold (default: older than 12 months or materially outdated).

**MCP calls:**
1. `brand-radar-cited-pages` → pages AI assistants cite.
2. `brand-radar-ai-responses` → context and claims in AI answers.
3. `site-audit-page-content` or crawled page data when available → visible dates/content cues.
4. `site-explorer-top-pages` / `organic-keywords` → business value of cited pages.

**Synthesis:**
- Flag cited pages that are old, thin, missing current stats, or mismatched to current positioning.
- Prioritize updates by citation frequency × organic value × business importance.
- Recommend exact content freshness actions: update date, replace stats, add examples, improve schema, add author/reviewer.

**Output:** Freshness audit table and refresh brief for each high-priority cited page.

---

## community-content-research

**Triggers:** "community content research", "what are people asking in communities", "Reddit topics", "forum research"

**Inputs needed:** topic/market, target audience, country/language, whether live community SERP data is required.

**MCP calls:**
1. `keywords-explorer-search-suggestions` and `related-terms` → question language and long-tail needs.
2. `serp-overview` for community-heavy queries → identify forums/communities ranking.
3. Social tools if connected (`social-media-posts`, post metrics) → owned social/community evidence.
4. If the user needs current Reddit/forum/news results, propose SerpApi with engine/query/credit cost and wait for approval.

**Synthesis:**
- Cluster community questions by pain, comparison, objection, workaround, and buying trigger.
- Map each cluster to content formats: FAQ, comparison, troubleshooting, template, case study, product-led tutorial.
- Preserve exact user language where available, but do not over-quote external content.

**Output:** Community insight report with topic clusters, user-language snippets, and content briefs.

---

## page-traffic-opportunities

**Triggers:** "page traffic opportunities", "pages close to page one", "increase traffic to existing pages", "CTR opportunities"

**Inputs needed:** target domain, country, time window.

**MCP calls:**
1. Prefer GSC MCP `get_advanced_search_analytics` / page+query tools if connected → clicks, impressions, CTR, position.
2. Otherwise Ahrefs `gsc-pages`, `gsc-keywords`, `gsc-ctr-by-position` if available.
3. Fallback to `site-explorer-organic-keywords` and `top-pages` → estimated ranking opportunities.
4. `serp-overview` for top candidates → SERP intent/features.

**Synthesis:**
- Find pages/queries with high impressions or volume at positions 4-20.
- Estimate lift from moving to target positions using GSC CTR curve when available.
- Separate title/meta CTR work from content expansion, internal linking, and link acquisition needs.

**Output:** Opportunity table with page, query, current position/CTR, estimated upside, recommended action, and effort.

---

## traffic-forecast

**Triggers:** "forecast traffic", "traffic forecast", "what traffic could this plan drive", "project SEO growth"

**Inputs needed:** target domain, keyword/page set or proposed plan, country, forecast horizon, assumptions.

**MCP calls:**
1. `keywords-explorer-overview` for target keywords → volume/KD.
2. `site-explorer-organic-keywords` and `top-pages` → current baseline.
3. GSC CTR curve (`gsc-ctr-by-position` or GSC MCP equivalent) when available; otherwise use clearly labeled scenario assumptions.
4. `site-explorer-metrics-history` → historical growth/seasonality context.

**Synthesis:**
- Build conservative/base/upside scenarios by target position and publication/ramp timing.
- Apply CTR assumptions transparently and avoid false precision.
- If PostHog conversions are available, convert traffic scenarios into conversion/revenue scenarios.

**Output:** Forecast report with assumptions, scenario table, risks, and actions needed to reach each scenario.

---

## client-pitch-deck

**Triggers:** "client pitch deck", "agency pitch", "proposal deck", "sell this SEO plan", "freelancer pitch"

**Inputs needed:** prospect domain, market, known competitors, service scope, deck audience.

**MCP calls:**
1. `site-explorer-metrics`, `metrics-history`, `top-pages`, and `organic-keywords` for prospect.
2. `site-audit-issues` if a project exists; otherwise use available crawl/domain evidence and state limitation.
3. `site-explorer-referring-domains` and competitor equivalents.
4. `brand-radar-visibility` / `share-of-voice` workflows if AI search is in scope.
5. Optional PostHog only if the user owns the prospect/client analytics.

**Synthesis:**
- Tell a buyer-ready story: current state, missed upside, competitor pressure, priority initiatives, expected outcomes, next 90 days.
- Keep evidence tight; avoid a raw data dump.
- Include assumptions and what data access would improve after onboarding.

**Output:** Slide-by-slide pitch deck outline or markdown deck content. If the user requests tickets/tasks, also create the implementation backlog.

---

## Composing workflows

When a request spans multiple workflows, run them in sequence and merge:

- "Audit my site and tell me what to fix" → `site-audit-triage` + `content-decay` + `internal-link-map`
- "Why are we losing traffic" → `content-decay` + `rank-tracker-report` + `web-analytics-deep-dive` + (if available) `brand-radar-visibility` for the AI search loss factor
- "Beat competitor X" → `competitor-overview` + `content-gap` + `link-intersect`
- "AI search strategy" → `brand-radar-visibility` + `ai-mention-gap` + `ai-cited-pages` + `ai-citation-freshness` + `e-e-a-t-audit`
- "Build our next quarter plan" → `content-calendar` + `page-traffic-opportunities` + `link-building-strategy`
- "Ahrefs' Agent-style client pitch" → `client-pitch-deck` backed by `competitor-overview`, `content-gap`, `site-audit-discovery`, and `link-building-strategy`

Compose, don't pad — the user wants the merged conclusion, not three reports stapled together.
