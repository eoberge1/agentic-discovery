# Discovery context

<!-- Copy this file to the root of your Cursor workspace as discovery-context.md and fill it in.
     The signal-triage skill reads it at the start of every run. Keep it short; delete sections you don't use. -->

## ProductSense

- Team: <team name or id from ProductSense; the daily window is filtered to this team>
- Include insights with no team: <yes | no>

## Jira Product Discovery

- Project: <project key, e.g. DISC>
- Idea issue type: <e.g. Idea>
- Tools: <Atlassian MCP | our own tool: name and how to call it>
- How to add evidence to an idea: <e.g. "comment with the insight link and quote" | "add a JPD insight" | "append to the Evidence field">
- Where to store the triage record on a new idea: <description | first comment>
- How the tier is recorded: <field name, label (tier-light / tier-standard / tier-full), or "in the description">

Status meanings (JPD status → meaning):

| Status | Meaning |
|---|---|
| <e.g. Parking lot> | parked |
| <e.g. Discovery, Ready for delivery, In delivery> | active |
| <e.g. Launch, Delivered> | shipped (in production; new requests become new ideas, not evidence) |
| <e.g. Archived, Won't do> | rejected (only counts when a reason is written on the idea) |

## Strategy

- Strategy in one paragraph: <...>
- Strategic bets (insights touching these are considered for Full tier):
  - <...>
- Non-goals (quoted verbatim when an insight is disqualified as out of scope):
  - <...>

## Segments

| Segment | Rule |
|---|---|
| <Enterprise> | <e.g. revenue ≥ $100k or > 500 seats> |
| <Mid-market> | <...> |
| <SMB> | <...> |
