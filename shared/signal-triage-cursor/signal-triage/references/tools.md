# Tools

The skill uses two MCP connections and one file. Call tools by the names below; if a tool is missing or fails, apply the fallback and say so in the proposal header.

## Discovery context file

`discovery-context.md` in the workspace root (or the path the PM gives). Read once per run. A template ships with this skill. Missing file: continue, search all teams, skip out-of-scope and strategic-bet checks, and use the default status meanings below.

## ProductSense MCP

| Tool | Used for | Notes |
|---|---|---|
| `searchInsights` | selecting the window; same-conversation lookup for clarity; fallback evidence search | Filter by `selectedTeams` from the context file, `startDate` / `endDate` as `YYYY/MM/DD`. Max 100 per call; page with `offset` until a page returns fewer than 100. Each result has `id`, `note`, `insightType`, `quote`, `customer { name, domain, value, valueIsFromCRM }`, `date`, `tags` and, when assessed, `evidenceLens` (problem clarity, demand scores, research gaps). |
| `findSimilarInsights` | Step 4 evidence: other insights about the same problem | Called from an insight id. Scans the whole organization: not per search result, not for routed, already-in-JPD, merge or post-launch insights, and once per group when several insights share a problem. Results are ranked by similarity; read each hit and judge it, the score does not decide. |
| `findEvidence` | optional, Step 4: deeper evidence for a proposed new idea | Searches insights and transcripts, takes about a minute. At most once per proposed idea. |
| `identifyCompany` | resolving a customer name to an id | Needed before `getCustomerDetails` and for customers a PM names. |
| `getCustomerDetails` | importance and stage when they would change the call; always for accounts that read like a sales conversation (stage tells prospect from customer, for the `deal_risk` flag) | Once per customer per run. |
| `listChurnSignals` | retention-critical flag | One call per run, `days: 30`; match by customer name. |
| `listTeams` | resolving the team name in the context file to an id | Only if the file gives a name, not an id. |

Customer `value` is annual revenue. `null` means missing: report it as missing, never as zero.

Insight links: `https://app.productsense.com/insight/<id>`. Transcript links: `https://app.productsense.com/feedback/<id>/transcript`.

## Jira Product Discovery

Use whichever JPD tools this workspace has: the Atlassian MCP or the team's own tool. The context file says which and how ideas are updated.

| Need | Typical call |
|---|---|
| search ideas, all statuses | JQL text search in the project, e.g. `project = KEY AND text ~ "excel export"`, no status filter. Keyword search: run two or three phrasings. |
| check whether an insight is already linked | JQL text search for the insight id or its URL |
| read an idea before updating | get issue by key, including status and the latest comments |
| create an idea | create issue of the idea type in the project |
| add evidence | the update method from the context file (comment, field, or JPD insight) |

Write only after the PM confirms in the same conversation.

**Status meanings.** Use the mapping in the context file. Without one: `Archived`, `Rejected`, `Won't do` are rejected; `Launch`, `Launched`, `Live`, `Released`, `In production`, `Delivered`, `Done`, `Shipped` are shipped (never merge targets); `Parking lot`, `Icebox`, `Paused`, `On hold` are parked; everything else is active. A rejected or archived idea counts as a prior decision only if a reason is written on it (description, comment or a reason field).

## Fallbacks

| Missing or failing | Do |
|---|---|
| discovery context file | defaults above; all teams; out-of-scope and strategic-bet checks skipped; stated in the header |
| `findSimilarInsights` | `searchInsights` with the problem as `searchTerm` (two phrasings), no date filter; stated in the header |
| `getCustomerDetails` / `identifyCompany` | use the insight's own `customer` fields; importance and stage `unknown` |
| `listChurnSignals` | retention-critical only from the insight's own words |
| JPD search | merge, prior-rejection, post-launch and already-in-JPD checks marked `not run`; every product insight can only be tiered; stated loudly in the header |
| JPD write | the proposal still stands; tell the PM exactly which changes to make by hand |
| a single call errors | that step is `not run` for that insight and its line says so; the run continues |
