# Workflow: Map a Site

**Objective:** Discover the URLs on a site — to scope a scrape/crawl, or to locate
the right page when a direct scrape comes back empty.
**Trigger:** "What pages are on X / find the page about Y on this site / list the
URLs."

## Inputs
| Input | Required | Default |
|---|---|---|
| url | yes | — |
| search filter | no | none (return all discovered URLs) |
| include subdomains | no | false |
| limit | no | tool default |

## Steps
1. Load the tool: ToolSearch `select:mcp__firecrawl__firecrawl_map`.
2. Call with `url`. Add `search:"<topic>"` to filter URLs to a subject — this is the
   recommended, cheap way to find a specific page before scraping.
3. Options as needed: `includeSubdomains`, `sitemap` (`include`|`skip`|`only`),
   `limit`, `ignoreQueryParameters`.
4. Hand the resulting URLs to `scrape_single_site` / `extract_structured` /
   `crawl_site` as the next step, or return the list per `../CLAUDE.md` conventions.

## Quality checklist
- [ ] URLs returned are on-target (used `search` if the user wanted a topic).
- [ ] Picked the cheapest path: map+scrape before reaching for `firecrawl_agent`.

## Edge cases
- **Sparse/empty map:** site may lack a sitemap — try `sitemap:"skip"` to force
  discovery, or fall back to `firecrawl_crawl` with a small `limit`.
- **Too many URLs:** tighten with `search` or `limit` before scraping downstream.

## Notes / learnings
_(append discoveries here)_
