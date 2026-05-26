# Agent AHHOG — Combined Workflows (Ahrefs × PostHog)

These workflows are the reason for connecting both data sources. Ahrefs answers "who arrives and from where." PostHog answers "what they do once here." Joined, they answer the questions marketing actually cares about: **which SEO investment produces revenue, and where the journey breaks.**

Same recipe format as `workflows.md`. When a request needs both acquisition and behavior data, use these. When it needs only one source, use the single-source recipes in `workflows.md` (Ahrefs) or compose directly from `posthog-mcp-tools.md` (PostHog).

A note on joining the two: Ahrefs and PostHog don't share IDs. You connect them at the **dimension** level — usually landing-page URL, traffic source/channel, or UTM parameters — not at the user level. Match on the URL path or the source attribute that exists in both systems.

---

## seo-to-conversion (closed-loop attribution)

**Triggers:** "which keywords actually convert", "does our SEO traffic make money", "ROI of organic", "from ranking to revenue", "close the loop on SEO"

**Inputs needed:** target domain, the conversion event name in PostHog (discover via schema), country, time window.

**Steps:**
1. **PostHog schema first.** `read-data-schema` to find the real conversion event name(s) and the property that holds traffic source / referrer / UTM / landing page.
2. **Acquisition side (Ahrefs).** `site-explorer-top-pages` and `site-explorer-organic-keywords` → which pages and keywords drive organic traffic, with estimated volume.
3. **Behavior side (PostHog).** `query-funnel` from landing (filtered to organic source or the SEO landing pages) → conversion event. Break down by landing-page path.
4. **Join on landing page.** For each top organic landing page from Ahrefs, attach its PostHog conversion rate and conversion count.

**Synthesis:**
- Rank organic landing pages by **conversions**, not just traffic. A page with 500 visits at 8% conversion beats a page with 5,000 visits at 0.2%.
- Identify "traffic-rich, conversion-poor" pages (high Ahrefs traffic, low PostHog conversion) → CRO opportunities.
- Identify "conversion-rich, traffic-poor" pages (low traffic, high conversion) → SEO scaling opportunities; these deserve more keyword targeting.
- Attach the keywords (from Ahrefs) that drive the high-converting pages → these are the keywords worth ranking harder for.

**Output:** Markdown report. TL;DR ranks keywords/pages by estimated converting value. Two highlighted lists: "scale these (convert well, need more traffic)" and "fix these (lots of traffic, poor conversion)." Action list ties each to a specific next step.

---

## landing-page-cro (traffic-rich, conversion-poor diagnosis)

**Triggers:** "this page gets traffic but doesn't convert", "improve landing page conversion", "why isn't this page converting", "CRO audit"

**Inputs needed:** the landing page URL, conversion event, time window.

**Steps:**
1. **Ahrefs:** confirm the page's organic traffic, top keywords, and SERP intent — `site-explorer-organic-keywords` filtered to the URL. Intent mismatch (page ranks for informational queries but asks for purchase) is a common root cause.
2. **PostHog funnel:** `query-funnel` for the on-page journey (pageview → key action → conversion). Find the step with the biggest drop.
3. **PostHog paths:** `query-paths` with the page as start point → where do users go instead of converting?
4. **PostHog session replays:** `query-session-recordings-list` filtered to the page, then `session-recording-summarize` on a few → qualitative friction.
5. **PostHog errors:** `query-error-tracking-issues-list` → is a JS error breaking the page?

**Synthesis:**
- Diagnose the drop: intent mismatch (Ahrefs), funnel step friction (PostHog funnel), distraction/exit (paths), UX friction (replays), or technical breakage (errors).
- Prioritize fixes by expected conversion lift × traffic volume.

**Output:** CRO diagnosis report. Root-cause section, prioritized fix list, and the specific evidence (which funnel step, which error, which replay pattern) behind each recommendation.

---

## content-roi (prioritize content by downstream value)

**Triggers:** "which content is worth keeping", "content ROI", "what content drives signups", "prune or invest content decisions"

**Inputs needed:** target domain (or `/blog/` section), conversion event, time window.

**Steps:**
1. **Ahrefs:** `site-explorer-top-pages` with `mode: prefix` on the content section → traffic per article.
2. **PostHog:** for each top article path, `query-funnel` or `query-trends` measuring how many visitors to that path later hit the conversion event (use the path as an entry filter; or `execute-sql` for a path → conversion join).
3. Combine into a per-article table: organic traffic, downstream conversions, assisted-conversion rate.

**Synthesis:**
- Classify each article: **invest** (traffic + conversions), **optimize** (traffic, no conversions — add CTAs/intent), **prune or merge** (no traffic, no conversions), **scale** (conversions, low traffic — needs SEO push).
- Cross-reference with `content-decay` from `workflows.md` to catch declining-but-converting articles that urgently need refreshing.

**Output:** Content portfolio report with the four-quadrant classification and a per-article action list.

---

## channel-truth (validate Ahrefs estimates against first-party data)

**Triggers:** "are Ahrefs numbers accurate", "real traffic vs estimated", "ground truth our SEO data", "reconcile analytics"

**Inputs needed:** target domain, time window.

**Steps:**
1. **Ahrefs:** `site-explorer-metrics-history` → estimated organic traffic over time.
2. **PostHog:** `query-trends` of pageviews/visitors filtered to organic source, same window, same granularity. Or `web-analytics-weekly-digest` for a quick read.
3. Plot both series; compute the ratio.

**Synthesis:**
- Ahrefs estimates and first-party data will differ — that's expected. The useful output is the **consistent ratio** (so future Ahrefs-only estimates can be calibrated) and any **divergence points** (where the trend lines split, indicating tracking gaps, algorithm shifts, or attribution changes).

**Output:** Reconciliation report with the dual-series chart, the calibration ratio, and notes on any divergence.

---

## audience-segment-seo (target SEO at your best-converting segments)

**Triggers:** "who are our best customers and how do we reach more", "SEO for our ICP", "target content at converting segments"

**Inputs needed:** conversion event, optionally an existing high-value cohort.

**Steps:**
1. **PostHog:** identify the highest-value converting segment — `cohorts-list` for existing cohorts, or `query-trends-actors` on the conversion to inspect who converts, then `persons-retrieve` / `persons-values-retrieve` for shared attributes (industry, use case, geography, referrer language).
2. Extract the segment's defining characteristics and the **landing pages / search intents** that brought them in (the source/landing-page property on those persons).
3. **Ahrefs:** take those winning intents and run `content-gap` / `keyword-research` to find more keywords matching that segment's intent.

**Synthesis:**
- Profile the best-converting segment.
- Map the keywords and content that attract that segment.
- Recommend new content/keyword targets that look like the proven winners.

**Output:** Segment-targeted SEO plan. Segment profile, the intents that win them, and a prioritized keyword/content target list to attract more of them.

---

## campaign-impact (measure a content launch or migration end-to-end)

**Triggers:** "did our content launch work", "impact of the migration", "before and after", "did the campaign move the needle"

**Inputs needed:** the launch/migration date, affected URLs, conversion event.

**Steps:**
1. **PostHog annotation:** if not already marked, offer to `annotation-create` for the launch date (write — confirm first) so every chart shows the boundary.
2. **Ahrefs:** `site-explorer-metrics-history` and `site-explorer-organic-keywords` before vs after → ranking and traffic change.
3. **PostHog:** `query-trends` of conversions for the affected pages before vs after; `query-funnel` to see if conversion rate (not just volume) changed.

**Synthesis:**
- Separate traffic change (Ahrefs) from conversion change (PostHog). A launch can win traffic but lose conversion, or vice versa — report both.
- Attribute the net business impact (converting visits gained or lost).

**Output:** Before/after impact report with both acquisition and conversion deltas, and a verdict on net impact.

---

## Composing with single-source workflows

Combined workflows often start from a single-source one and extend it:

- `content-decay` (Ahrefs) + PostHog conversion data → `content-roi` (don't just refresh declining pages; refresh declining pages *that convert*).
- `competitor-overview` (Ahrefs) + `audience-segment-seo` (PostHog) → find what attracts your converters, then see which competitor also targets them.
- `migration-plan` (Ahrefs) + `campaign-impact` (PostHog) → plan the migration, then measure it end-to-end.

Always lead with the business question. If the user cares about revenue/conversions, a combined workflow beats either source alone. If they only care about rankings or only about on-site behavior, don't over-engineer — use the single source.

---

## When PostHog isn't connected

If only Ahrefs MCP is available, these combined workflows degrade gracefully: run the Ahrefs half, and clearly flag the missing conversion side ("I can show traffic and rankings, but to tie this to actual conversions I'd need the PostHog MCP connected"). Never fabricate conversion numbers. The same applies in reverse if only PostHog is connected.

---

# PostHog-only marketing workflows

These don't need Ahrefs. They're the product-analytics half of marketing — and several involve writes (Tier 2–3 per the permission tiers in SKILL.md). Compose them from `posthog-mcp-tools.md`.

## funnel-diagnosis

**Triggers:** "where do users drop off", "funnel analysis", "conversion funnel", "biggest leak in the funnel"

**Steps:** `read-data-schema` → `query-funnel` across the key event sequence → for the worst-dropping step, `query-paths` (where do they go instead) and `query-session-recordings-list` + `session-recording-summarize` (why).

**Output:** Funnel report with per-step conversion, the biggest leak highlighted, root-cause evidence, and prioritized fixes.

## audience-cohort-build (Tier 2 write)

**Triggers:** "segment our users", "create a cohort of", "group users who", "build an audience"

**Steps:** `read-data-schema` → confirm the behavioral/property criteria → `cohorts-create` (dynamic for behavior-based, static for a fixed list). Report the cohort ID and size. Then optionally analyze it (`query-trends` broken down by the cohort).

**Output:** The created cohort + a short profile of who's in it and how they behave.

## cro-experiment-setup (Tier 2 create, Tier 3 launch)

**Triggers:** "set up an A/B test", "test this headline/CTA", "experiment on the landing page"

**Steps:** `read-data-schema` to find the conversion metric → `experiment-create` in draft with control/variant and the funnel/trend metric → present the draft to the user → on their go-ahead, `experiment-launch` (Tier 3; if it touches a live flag, that's the confirm point). Later, `experiment-results-get` / `experiment-timeseries-results` to read the winner, and `experiment-ship-variant` to roll out.

**Output:** The configured experiment (draft first), then results analysis when data is in.

## voice-of-customer-survey (Tier 2 create, Tier 3 go-live)

**Triggers:** "ask users why", "run a survey", "get feedback on", "what do customers think about"

**Steps:** `survey-create` in draft with the questions → present for review → on go-ahead, take it live. Later, `survey-stats` / `surveys-global-stats` to read responses, and feed the language customers use back into keyword/messaging work (hand off to Ahrefs `keyword-research` if relevant).

**Output:** The survey (draft first), then a themed analysis of responses.

## retention-analysis

**Triggers:** "do users come back", "retention", "are we sticky", "churn signal"

**Steps:** `read-data-schema` → `query-retention` (cohort grid) and `query-lifecycle` (new/returning/dormant/resurrecting) → optionally break down by acquisition source to see which channels bring back loyal users.

**Output:** Retention report; if source breakdown is included, a note on which acquisition channels produce the stickiest users (actionable for where to invest).

## marketing-dashboard-build (Tier 2 write)

**Triggers:** "build a dashboard", "set up a marketing dashboard in PostHog", "I want a recurring view of"

**Steps:** agree the metrics → `insight-create` for each (trends, funnels) → `dashboard-create` and attach them → report the dashboard link. This lives natively in PostHog so the team sees it without Claude.

**Output:** A live PostHog dashboard + the link, plus a one-paragraph summary of what it tracks.
