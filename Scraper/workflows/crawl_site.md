# Workflow: Crawl a Site

**Objective:** Collect content from many related pages of one site.
**Trigger:** "Crawl X / get all the blog posts / scrape the whole docs section."

> Crawl is **async** and responses can be large. Keep `limit` and `maxDiscoveryDepth`
> modest. For big sites, prefer `map_site` + targeted `scrape` instead.

## Inputs
| Input | Required | Default |
|---|---|---|
| url | yes | — |
| scope (paths/section) | no | the given URL's section |
| limit | no | start small (~20) and raise if needed |
| max depth | no | 2 |

## Steps
1. Load the tools: ToolSearch
   `select:mcp__firecrawl__firecrawl_crawl,mcp__firecrawl__firecrawl_check_crawl_status`.
2. Start the crawl with a tight scope:
   - `limit` (cap pages), `maxDiscoveryDepth` (2 is usually enough).
   - `includePaths` / `excludePaths` to stay in the right section.
   - `deduplicateSimilarURLs:true`, `sitemap:"include"`.
   - `scrapeOptions:{formats:["markdown"], onlyMainContent:true}` for page content.
   This returns a **job id**.
3. Poll `firecrawl_check_crawl_status` with the id until `completed` (or `failed`).
4. Assemble results and deliver per `../CLAUDE.md` (large/structured -> file in
   `output/` or cloud, not a giant inline dump).

## Quality checklist
- [ ] Scope was constrained (paths/limit/depth) — no runaway crawl.
- [ ] Polled to completion before reporting.
- [ ] Output saved sensibly rather than dumped inline if large.

## Edge cases
- **Token overflow / huge response:** lower `limit`/`depth`, or switch to
  `map_site` + selective scrape.
- **Crawl stalls or fails:** re-check status once; if `failed`, narrow scope and
  retry. Don't burn credits re-running a too-broad crawl.

## Notes / learnings
_(append discoveries here — good limit/depth per site, path patterns, etc.)_
