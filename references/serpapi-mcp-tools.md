# SerpApi MCP — Tool Reference

SerpApi brings something Ahrefs and PostHog cannot: **live, real-time SERP data from the actual search engines** — Google, Bing, YouTube, and dozens more — at the moment of query, not as historical estimates.

## Positioning within Agent AHHOG

| Data source | What it answers | Freshness |
| --- | --- | --- |
| Ahrefs | Who ranks, how many links, estimated traffic, AI visibility trends | Updated daily/weekly (historical) |
| PostHog | What users do on your site after they land | Real-time first-party events |
| **SerpApi** | **What the SERP actually looks like right now** | Live, real-time |

The gap SerpApi fills: Ahrefs' SERP data is excellent for trend and competitive analysis, but it's not a live snapshot. If you need to know **exactly what appears on Google right now** — which featured snippet is showing, which ads are running, what the People Also Ask box says today, what's trending on Google Trends this hour — SerpApi is the only way to get it.

---

## This MCP is optional and gated — read this before calling it

SerpApi MCP is **opt-in** and should be used **only when Ahrefs (or PostHog) genuinely cannot answer the question** and the data is time-sensitive enough that a historical estimate won't do.

### Three rules for using SerpApi

**Rule 1 — Try Ahrefs first.**
Most SERP questions Ahrefs can answer sufficiently: who ranks, what the top pages are, what SERP features exist for a keyword, what competitors' pages look like. Reach for SerpApi only when Ahrefs' answer is "I have data from N days ago and the user needs it right now."

**Rule 2 — Ask the user before calling.**
SerpApi consumes search credits from the user's SerpApi plan. Before making any SerpApi call, state explicitly:
- What you're about to search for
- Why Ahrefs/PostHog can't answer it
- Approximately how many searches it will consume

Then wait for explicit approval ("yes, go ahead") before proceeding. This is not optional — even for a single search.

**Rule 3 — Minimum searches, maximum value.**
Once approved, use the minimum number of searches to answer the question. If you need 5 keywords checked, do them in one batch where possible, not one-by-one.

### Approval prompt template

When SerpApi is needed, say something like:

> "To get the live [Google SERP / trending data / news coverage] for this, I'd need to call SerpApi — that's the only way to see what's actually showing right now (Ahrefs has data from ~[N days] ago). This would use approximately [N] SerpApi search credits from your plan. Want me to proceed?"

If the user says no or is unsure, fall back to Ahrefs data and note it may not reflect the current live state.

---

## Setup (Claude Code)

```bash
claude mcp add --transport http serpapi https://mcp.serpapi.com/YOUR_API_KEY/mcp
```

Replace `YOUR_API_KEY` with your SerpApi API key from https://serpapi.com/dashboard.

Free plan: 250 searches/month. Paid plans from $50/month for 5,000 searches.

The user in claude.ai has a pre-connected SerpApi integration — if you see `serpapi:search` in available tools, you can propose using it (still subject to the approval rules above).

---

## The tool: `search`

SerpApi MCP exposes a single, flexible `search` tool. All engine-specific queries go through the same interface.

**Core parameters:**

| Parameter | Values | Notes |
| --- | --- | --- |
| `engine` | `google`, `bing`, `youtube`, `google_trends`, `google_news`, `google_images`, `google_shopping`, `duckduckgo`, `yahoo`, `yandex`, `baidu`, and many more | Required. Defaults to `google` if omitted. |
| `q` | search query string | Required for most engines |
| `location` | e.g. `"Jakarta, Indonesia"`, `"United States"` | Important for geo-specific SERPs |
| `hl` | language code, e.g. `"id"`, `"en"` | Interface language |
| `gl` | country code, e.g. `"id"`, `"us"` | Geographic context for results |
| `num` | integer | Number of results (default 10, max 100 for Google) |
| `output` | `"json"` (default) or `"clean"` | `clean` returns simplified fields: title, snippet, link |

**Engine-specific parameters** (only needed for specialty engines):

| Engine | Extra params |
| --- | --- |
| `google_trends` | `data_type` (`TIMESERIES`, `GEO_MAP`, `RELATED_QUERIES`, `RELATED_TOPICS`), `date` range |
| `google_news` | `topic_token` or just `q` for news about a topic |
| `google_shopping` | `q`, standard shopping params |
| `youtube` | `q`, `sp` for sorting |
| `google_images` | `q`, `tbs` for time/type filters |

---

## When SerpApi is genuinely needed (decision guide)

### ✅ Use SerpApi — Ahrefs can't do this

**1. Live SERP snapshot for a specific keyword right now**
Ahrefs shows *historical ranking data* for a keyword. SerpApi shows *what's actually on the SERP at this moment* — including the exact featured snippet text, People Also Ask questions, ad copy, local pack results, current sitelinks.

Use case: "What does the SERP look like for [keyword] in Indonesia right now?" or "Is our featured snippet still showing?"

```json
{ "engine": "google", "q": "your keyword", "location": "Jakarta, Indonesia", "hl": "id", "gl": "id" }
```

**2. Real-time Google Trends**
Ahrefs shows search volume as a historical estimate. Google Trends via SerpApi shows *right now*: is this term trending up this week? What are the related rising queries? What's the geographic demand split today?

Use case: "Is [topic] trending right now?", "What's trending in Indonesia this week?", "What are people searching related to [event]?"

```json
{ "engine": "google_trends", "q": "keyword", "data_type": "TIMESERIES", "date": "today 3-m" }
```

**3. Live Google News coverage**
Ahrefs doesn't index news articles in real-time. SerpApi's Google News gives current coverage of a topic — useful for brand monitoring (what's being written about us today?), competitive intel (what news is boosting a competitor?), or content timing (is this topic in the news right now?).

Use case: "What's being written about [brand/topic] today?", "Is there news about [competitor] that might explain their traffic spike?"

```json
{ "engine": "google_news", "q": "brand or topic", "hl": "id", "gl": "id" }
```

**4. Multi-engine SERP comparison**
Sometimes a keyword behaves very differently on Google vs Bing vs DuckDuckGo — especially for local or commercial intent. Ahrefs is primarily Google. SerpApi can check all.

Use case: "How does our site rank on Bing vs Google for [keyword]?"

**5. Live ad intelligence (Google Shopping / paid)**
Ahrefs' paid data is useful but not real-time. SerpApi shows exactly which ads are running right now, at what position, with what copy — useful for paid competitive research that needs freshness.

**6. YouTube SERP**
Ahrefs doesn't cover YouTube rankings. SerpApi does.

Use case: "Who's ranking on YouTube for [keyword]?", "What's the top YouTube content for our topic?"

**7. Google People Also Ask — live**
PAA questions change frequently. Ahrefs captures them at crawl time. SerpApi gets the current live set.

Use case: "What are people asking about [topic] on Google right now?"

---

### ❌ Don't use SerpApi — Ahrefs is better

| Request | Use instead |
| --- | --- |
| "Who ranks for [keyword]?" (general ranking data) | `keywords-explorer-overview` + `serp-overview` |
| "What keywords does [site] rank for?" | `site-explorer-organic-keywords` |
| "How much traffic does [site] get?" | `site-explorer-metrics` |
| "What SERP features are there for [keyword]?" (historical) | `rank-tracker-serp-overview` |
| "How has [keyword] volume changed?" | `keywords-explorer-volume-history` |
| "What backlinks does [site] have?" | `site-explorer-all-backlinks` |

The rule of thumb: if the question can be answered with Ahrefs data from the past week, that's good enough. Call SerpApi only when "right now" is truly the requirement.

---

## Cost awareness

SerpApi is a paid API with credit-based pricing:

- Free plan: 250 searches/month
- Paid plans: from $50/month for 5,000 searches

One `search` call = 1 credit, regardless of how many results it returns.

High-consumption scenarios (always confirm before):
- Checking 50 keywords live = 50 credits
- Trends for 10 keywords × 3 data types = 30 credits
- Multi-engine comparison for 5 keywords × 4 engines = 20 credits

Always state the estimated credit cost before proceeding. If the user is on the free plan (250/month), a single multi-keyword check could use a meaningful fraction of their monthly budget.

---

## Output handling

SerpApi returns structured JSON. For marketing purposes, the most useful fields in a Google SERP response:

- `organic_results[]` — title, link, snippet, position, displayed_link, sitelinks, rich_snippet
- `answer_box` — featured snippet content (if present)
- `people_also_ask[]` — PAA questions and answers
- `ads[]` — paid results at top/bottom
- `local_results[]` — map pack results (for local queries)
- `related_searches[]` — related query suggestions
- `knowledge_graph` — entity panel data

For `output: "clean"`, the response is simplified to title/snippet/link per result — faster to process, use for bulk keyword checks.

Always note in the report that this data is a live SERP snapshot taken at a specific date and time — it will change.
