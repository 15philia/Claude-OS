# Firecrawl MCP Cheat Sheet

A reference for the Firecrawl MCP server tools available in this workspace. Use this whenever the user asks to gather data from websites.

> **Tools are deferred.** Each Firecrawl tool's schema must be loaded with `ToolSearch` before it can be called, e.g. `select:mcp__firecrawl__firecrawl_scrape`. Calling before loading fails with `InputValidationError`.

---

## Decision guide — which tool?

| Situation | Tool |
|-----------|------|
| I have ONE URL and want its content | `firecrawl_scrape` |
| I want SPECIFIC data points from one or more known URLs | `firecrawl_scrape` / `firecrawl_extract` (JSON format) |
| I don't know which site has the info; open-ended question | `firecrawl_search` |
| I know the site, want to discover its URLs first | `firecrawl_map` |
| I want content from MANY pages of one site | `firecrawl_crawl` (async → poll status) |
| A local file on disk (PDF, docx, xlsx, html) | `firecrawl_parse` |
| Scrape returned empty/minimal (JS-heavy SPA) | retry `scrape` w/ `waitFor`, then `map`+scrape, then `agent` |
| Multi-step page interaction (click, fill, login) | `firecrawl_scrape` → `firecrawl_interact` |
| Complex research across unknown sites | `firecrawl_agent` (async → poll status) |
| Watch a page for changes over time | `firecrawl_monitor_*` |

**Golden rule:** specific data points → use **JSON format with a schema**. Entire page for reading/summarizing → use **markdown**.

---

## Core tools

### `firecrawl_scrape` — single page (default workhorse)
Fastest, most reliable. Use for one known URL.

Key params:
- `url` (required)
- `formats`: array — `markdown`, `html`, `rawHtml`, `screenshot`, `links`, `summary`, `json`, `branding`, `changeTracking`, `query`, `audio`
- `onlyMainContent`: strip nav/footer/chrome (default behavior is main content)
- `includeTags` / `excludeTags`: CSS selectors to keep/drop
- `waitFor`: ms to wait for JS render (try `5000`–`10000` if content is empty)
- `maxAge`: serve from cache if younger than N ms (faster/cheaper)
- `proxy`: `basic` | `stealth` | `enhanced` | `auto` (use stealth for anti-bot sites)
- `mobile`, `location` (`{country, languages}`), `parsers: ["pdf"]`, `removeBase64Images`, `redactPII`
- `actions`: scripted browser steps (`wait`, `click`, `scroll`, `write`, `press`, `screenshot`, `executeJavascript`, `scrape`)
- `jsonOptions`: `{ prompt, schema }` when `formats:["json"]`

General markdown scrape (what "just scrape this site" usually means):
```json
{ "url": "https://example.com", "formats": ["markdown"] }
```

Structured extraction from one page:
```json
{
  "url": "https://example.com/pricing",
  "formats": ["json"],
  "jsonOptions": {
    "prompt": "Extract each plan name and price",
    "schema": { "type":"object", "properties": { "plans": { "type":"array" } } }
  }
}
```
The response `metadata.scrapeId` is needed for `firecrawl_interact`.

### `firecrawl_search` — web search (preferred over built-in web search)
Returns rich results and can scrape each hit inline.

Key params:
- `query` (required) — supports operators: `"exact"`, `-exclude`, `site:`, `inurl:`, `intitle:`, `related:`, `imagesize:`, `larger:`
- `limit`: number of results
- `sources`: `[{type:"web"}]`, `news`, or `images` (default web)
- `includeDomains` / `excludeDomains` (don't use both at once)
- `location`, `tbs` (time filter)
- `scrapeOptions`: same shape as scrape — fetch full content of each result

> After using search results, call `firecrawl_search_feedback` with the `id` from the response (rating + valuableSources/missingContent). This refunds 1 credit (search costs 2) — within ~2 min only.

### `firecrawl_map` — discover URLs
Lists indexed URLs on a site. Use before deciding what to scrape, or to find the right page when scrape comes back empty.

Key params: `url` (required), `search` (filter URLs by topic), `limit`, `includeSubdomains`, `sitemap` (`include`|`skip`|`only`), `ignoreQueryParameters`.

```json
{ "url": "https://docs.example.com", "search": "webhook events" }
```

### `firecrawl_crawl` — many pages (async)
Crawls and scrapes multiple related pages. **Returns a job ID** → poll with `firecrawl_check_crawl_status`.

⚠️ Responses can be huge. Keep `limit` and `maxDiscoveryDepth` modest. For big sites, prefer `map` + targeted `scrape`.

Key params: `url`, `limit`, `maxDiscoveryDepth`, `includePaths`/`excludePaths`, `allowExternalLinks`, `allowSubdomains`, `crawlEntireDomain`, `deduplicateSimilarURLs`, `sitemap`, `delay`, `maxConcurrency`, `scrapeOptions` (same as scrape), `prompt`, `webhook`.

```json
{ "url":"https://example.com/blog/*", "limit":20, "maxDiscoveryDepth":2, "deduplicateSimilarURLs":true }
```

### `firecrawl_check_crawl_status` — poll a crawl
`{ "id": "<crawl-job-id>" }` → status, progress, results when ready.

### `firecrawl_extract` — structured data across URLs (LLM)
Pull specific fields from one or more pages with a prompt + schema.

Key params: `urls` (array, required), `prompt`, `schema`, `allowExternalLinks`, `enableWebSearch`, `includeSubdomains`.

```json
{
  "urls": ["https://example.com/p1","https://example.com/p2"],
  "prompt": "Extract product name, price, description",
  "schema": { "type":"object", "properties": {
    "name":{"type":"string"}, "price":{"type":"number"} }, "required":["name","price"] }
}
```

### `firecrawl_parse` — local files (self-hosted)
Extract content from a file on disk: `.pdf .docx .doc .odt .rtf .xlsx .xls .html .htm .xhtml`.

Key params: `filePath` (required), `formats` (`markdown`/`json`/…), `jsonOptions`, `parsers:["pdf"]`, `pdfOptions.maxPages`, `redactPII`, `onlyMainContent`.
Not supported: actions, screenshots, `waitFor`, `location`, `mobile`, non-basic proxy. For remote URLs use `scrape` instead.

---

## Interactive & autonomous

### `firecrawl_interact` — drive a live page
Requires a `scrapeId` from a prior `firecrawl_scrape`. Click, fill forms, navigate, extract dynamic content.
Params: `scrapeId` (required), `prompt` *or* `code`, `language` (`node`|`python`|`bash`), `timeout` (1–300s).

### `firecrawl_agent` + `firecrawl_agent_status` — autonomous research (async)
Describe what you need; the agent searches, follows links, extracts. **Async** — `firecrawl_agent` returns a job ID, then poll `firecrawl_agent_status` every 15–30s, **for at least 2–3 min** (complex tasks take 5+ min). Stop only on `completed` or `failed`.
Params: `prompt` (required, ≤10k chars), optional `urls`, optional `schema`.
Use as a last resort — prefer scrape/search/map when you know the target.

---

## Monitoring (watch pages for change)

- `firecrawl_monitor_create` — simple path: `page`/`pages` + `goal` (plain-English description of what change matters) + optional `scheduleText` (default "every 30 minutes"), `email`, `webhookUrl`. Use `body` for advanced (crawl targets, JSON change tracking, custom retention).
- `firecrawl_monitor_list` — list monitors (`limit`, `offset`).
- `firecrawl_monitor_get` / `firecrawl_monitor_update` / `firecrawl_monitor_delete` — manage one.
- `firecrawl_monitor_run` — trigger a check now (`{id}`).
- `firecrawl_monitor_check` / `firecrawl_monitor_checks` — inspect check results & diffs; filter `pageStatus` (`same`|`new`|`changed`|`removed`|`error`). Prefer summarizing `diff.json` field paths over raw markdown diffs.

---

## Feedback

- `firecrawl_search_feedback` — search-specific quality feedback (refunds 1 credit). `{searchId, rating, valuableSources, missingContent, querySuggestions}`. ~2 min window.
- `firecrawl_feedback` — generic endpoint feedback for `scrape`/`parse`/`map`/`search` jobs. `{endpoint, jobId, rating, issues, note}`.

---

## Handling JS-heavy / empty results (escalation order)
1. Re-scrape with `waitFor: 5000`–`10000`.
2. Try a different/base URL (drop `#hash` fragments).
3. `firecrawl_map` with `search` to find the real content page, then scrape it.
4. Switch `proxy` to `stealth` or `enhanced` for anti-bot sites.
5. Only then reach for `firecrawl_agent`.

## Cost / etiquette notes
- `scrape` ≈ 1 credit; `search` ≈ 2 (refundable to 1 via feedback).
- Use `maxAge` to hit cache and save credits/time when freshness isn't critical.
- For large sites: `map` to scope → targeted `scrape`/`extract`, rather than a broad `crawl`.
- Save scraped output to files in this `Scraper/` directory when the user wants to keep it.
