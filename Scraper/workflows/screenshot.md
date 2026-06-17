# Workflow: Screenshot a Page

**Objective:** Capture a visual screenshot of a page (full-page or viewport).
**Trigger:** "Screenshot X / show me what this page looks like / grab an image of it."

## Inputs
| Input | Required | Default |
|---|---|---|
| url | yes | — |
| full page vs viewport | no | full page |
| viewport size | no | tool default |
| output destination | no | save image to output/ |

## Steps
1. Load the tool: ToolSearch `select:mcp__firecrawl__firecrawl_scrape`.
2. Call with `formats:["screenshot"]` and:
   - `screenshotOptions:{fullPage:true}` for the entire page; set
     `viewport:{width,height}` for a specific size, `quality` to tune file size.
   - `waitFor` (e.g. 5000–8000) so JS/images render before the shot.
   - To capture a state behind interaction (cookie banner dismissed, tab clicked),
     add `actions:[{type:"click", selector:"..."}, {type:"screenshot"}]`.
3. `proxy:"stealth"` if the site blocks bots.
4. Save the image to `output/<domain>-<date>.png` (or return per `../CLAUDE.md`).

## Quality checklist
- [ ] Page fully rendered (no blank/loading state) — used `waitFor`.
- [ ] Full-page vs viewport matches what was asked.
- [ ] Image saved with a clear name.

## Edge cases
- **Blank/half-loaded shot:** increase `waitFor`; for lazy-loaded content add a
  `scroll` action before the screenshot.
- **Overlay/cookie wall blocking view:** dismiss it with a `click` action first.

## Notes / learnings
_(append discoveries here — waitFor values, selectors for common overlays, etc.)_
