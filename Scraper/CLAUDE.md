# CLAUDE.md — Scraper

## Purpose
This project gathers data from the web: **scraping** page content, **extracting**
structured data, capturing **screenshots**, **crawling** whole sites, and **mapping**
site URLs. When the user points you at a URL or asks for web data, the work and its
outputs live here.

## How this project works (WAT, adapted)
This is a **WAT** project (Workflows · Agents · Tools), like the rest of Claude OS —
with one adaptation: the deterministic **Tools layer is the Firecrawl MCP server**,
not Python scripts.

- **Workflows** (`workflows/*.md`) — plain-language SOPs: objective, inputs, steps,
  edge cases. Read the relevant one before acting.
- **Agent** (you) — read the workflow, pick the right Firecrawl tool, run it in
  sequence, recover from failures, ask when genuinely unsure.
- **Tools** (Firecrawl MCP) — does the actual web work. Its full reference is
  **`FIRECRAWL_CHEATSHEET.md`** — read it before using Firecrawl.

## Operating rules
1. **Check `workflows/` first.** If an SOP fits the request, follow it. Prefer
   Firecrawl over the built-in WebFetch/WebSearch tools.
2. **Firecrawl tools are deferred** — load each schema with `ToolSearch`
   (`select:mcp__firecrawl__firecrawl_<tool>`) before calling, or it errors.
3. **Format rule:** specific data points -> JSON format + schema; whole page for
   reading/summary -> markdown. (See cheat sheet for tool-by-tool detail.)
4. **Empty/JS-heavy result?** Escalate: `waitFor` -> `map` -> `stealth` proxy ->
   `agent`. Don't jump straight to the agent.
5. **Self-improvement loop:** when you learn something durable (rate limits, a site
   needing stealth, a better tool choice), append it to the relevant workflow's
   *Notes* section. Don't create or overwrite workflows without asking unless told to.

## Tool selection (quick reference — details in the cheat sheet)
| Need | Tool |
|---|---|
| One known URL's content/screenshot | `firecrawl_scrape` |
| Specific fields from known URL(s) | `firecrawl_scrape`/`firecrawl_extract` (JSON) |
| Don't know which site has it | `firecrawl_search` |
| Discover a site's URLs | `firecrawl_map` |
| Many pages of one site | `firecrawl_crawl` (async -> poll) |
| Multi-step interaction (click/form) | `firecrawl_scrape` + `firecrawl_interact` |
| Open-ended research, unknown sources | `firecrawl_agent` (async -> poll) |
| Watch a page for changes | `firecrawl_monitor_*` |

## Output conventions (decide per task)
- **Ad-hoc / one-off:** return results inline, or save to `output/` (create it as
  needed). Markdown for readable content; JSON/CSV for structured data. Name clearly,
  e.g. `output/<domain>-<date>.md`.
- **Structured or recurring datasets:** push the deliverable to the cloud (Google
  Drive/Sheets) so the user can access it directly; keep only disposable
  intermediates locally.
- When unsure which a task wants, ask once, then proceed.

## Git
Follow the repo-wide conventions in `../CLAUDE.md` (clean commit per meaningful
change, push to the project remote).
