# Workflow: Scrape a Single Site

**Objective:** Get the content (or a screenshot) of one known URL.
**Trigger:** "Scrape this page / grab this URL / screenshot this site."

## Inputs
| Input | Required | Default |
|---|---|---|
| url | yes | — |
| what to capture | no | clean markdown of main content |
| output destination | no | inline; save to output/ if large or asked |

## Steps
1. Load the tool: ToolSearch `select:mcp__firecrawl__firecrawl_scrape`.
2. Pick the format for the ask:
   - Readable content -> `formats:["markdown"]`, `onlyMainContent:true`.
   - Specific fields -> `formats:["json"]` + `jsonOptions:{prompt, schema}`.
   - Screenshot -> `formats:["screenshot"]` (+ `screenshotOptions.fullPage:true` for
     the whole page).
3. Empty/minimal result? Add `waitFor:8000`; still empty -> `firecrawl_map` with
   `search` to find the real page URL, then scrape that; anti-bot -> `proxy:"stealth"`.
4. Use `maxAge` to serve from cache when freshness isn't critical (saves credits).
5. Deliver per the output conventions in `../CLAUDE.md`.

## Quality checklist
- [ ] Got real content, not nav/cookie/footer chrome.
- [ ] Right format for the ask (markdown vs JSON vs screenshot).
- [ ] Saved/named clearly if persisted.

## Edge cases
- **SPA returns empty:** `waitFor`, then map+scrape the real page.
- **Anti-bot / 403:** switch `proxy` to `stealth` or `enhanced`.
- **Paywall:** note it; don't fabricate the blocked content.

## Notes / learnings
_(append discoveries here — sites needing stealth, waitFor values that worked, etc.)_
