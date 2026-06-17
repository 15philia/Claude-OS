# Workflow: Extract Structured Data

**Objective:** Pull specific, structured fields (prices, names, specs, lists) from one
or more pages into a defined schema.
**Trigger:** "Get the price/specs/contact info from these pages / build a table of X."

## Inputs
| Input | Required | Default |
|---|---|---|
| urls | yes | — |
| fields / schema | yes | infer from the request, then confirm if ambiguous |
| output destination | no | JSON inline; file/cloud if it's a dataset |

## Steps
1. Choose the tool:
   - **One URL** -> `firecrawl_scrape` with `formats:["json"]` +
     `jsonOptions:{prompt, schema}`.
   - **Multiple URLs** -> `firecrawl_extract` with `urls`, `prompt`, `schema`.
   Load the relevant schema via ToolSearch first.
2. Define a JSON `schema` for the exact fields wanted; mark required fields. Write a
   clear `prompt` describing what to extract.
3. For `firecrawl_extract`, set `enableWebSearch`/`allowExternalLinks`/
   `includeSubdomains` only if the data genuinely needs them.
4. Validate the output against the schema; if fields come back empty, the page may be
   JS-rendered — for the scrape path add `waitFor`, or `map` to the real page first.
5. Deliver: small result inline; a dataset -> JSON/CSV in `output/` or pushed to
   cloud (Sheets) per `../CLAUDE.md`.

## Quality checklist
- [ ] Output matches the requested schema; required fields populated.
- [ ] No invented values — empty over fabricated.
- [ ] Multi-URL results are consistent (same shape per row).

## Edge cases
- **Empty fields on a SPA:** `waitFor`, or map+scrape the real page, then re-extract.
- **Fields vary across pages:** loosen the schema (optional fields) rather than
  forcing values.
- **Many URLs:** prefer one `firecrawl_extract` call over many scrapes.

## Notes / learnings
_(append discoveries here — schema patterns that worked per site, etc.)_
