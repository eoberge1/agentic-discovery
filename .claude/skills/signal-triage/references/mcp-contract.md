# MCP tool contract for signal-triage

**Contract version:** 2 (v2 adds similarity search from stored embeddings; a v1 server still works, see Degradation)
**Audience:** whoever maintains the ProductSense MCP connection for a tenant, and anyone checking why the skill degraded.

The skill depends on ten tools. Eight are required; `getDiscoveryContext` and `compareSignals` are optional. The skill calls them by these names and expects these shapes. A server that exposes the same capability under a different name needs an alias; the skill does not probe for alternatives.

The skill reports the contract version it was written for. `getDiscoveryContext` returns the version the server implements; on mismatch the skill warns in the proposal header and continues.

## Shared types

```
Customer {
  id: string
  name: string
  domain?: string
  segment: string | null        // per tenant definitions, e.g. "enterprise"
  tier: string | null           // plan tier
  arr: number | null            // null = missing; never substitute 0
  tenure_since?: date
  churn_status?: string         // e.g. "subscribed", "at_risk", "churned"
  churn_risk?: "low" | "medium" | "high" | null
  importance?: number           // tenant scale, e.g. 1–5
}

SignalSummary {
  id: string
  date: datetime
  source_type: string           // "call" | "slack" | "support" | "note" | "email" | ...
  customer_id: string | null
  excerpt: string               // first ~200 chars
  url: string
}

Signal extends SignalSummary {
  verbatim: string              // full text as captured
  source_link?: string          // link to the origin (thread, ticket, call)
  extracted_problem?: string    // ProductSense's own extraction, if any
  embedded_on?: "extracted_problem" | "verbatim"   // which text the stored embedding was computed from
  status: "open" | "linked" | "dismissed"
  linked_opportunity_ids: string[]
  dismissal?: { reason_type: string, reason: string, related_opportunity_id?: string, by: string, at: datetime }
}

OpportunitySummary {
  id: string
  title: string
  status: string                // tenant pipeline status; see "status classes" below
  problem: string               // one line
  url: string
  decision?: { outcome: string, reasoning: string, decided_by: string, at: datetime }
  similarity?: number           // 0–1 when returned by search
}
```

**Similarity search.** ProductSense stores an embedding for every signal and every opportunity. The two search tools accept either a text `query` or a reference to a stored item (`like_signal_id`, `like_opportunity_id`); exactly one of the three is given. With a reference, the server reuses the stored embedding: nothing is re-embedded, the call is fast, and two runs over the same signal get the same neighbours. A text `query` is embedded at call time and is still needed for pasted text (no stored item) and for a restated problem that differs from what was embedded. The item named by `like_*` is never returned in its own results.

`similarity` is cosine-style, 0–1, and comparable within one tenant and one embedding model. It is not calibrated across tenants, so the skill uses it to order and shortlist candidates, never as the decision. Servers should embed a signal on its `extracted_problem` when one exists (a verbatim like "yeah, that would be important" embeds as noise) and an opportunity on its title plus problem.

**Status classes.** The skill needs to know which tenant statuses are *active* (merge target), *rejected* (prior decision, needs reasoning), *shipped* (post-launch feedback) and *parked* (iceboxed or paused: related only, never a merge target, never a rejection basis). `getDiscoveryContext` may return this mapping; otherwise the skill treats `rejected`, `archived`, `disqualified` as rejected, `shipped`, `done`, `released`, `launched` as shipped, `icebox`, `parked`, `paused` as parked, and everything else as active.

## Read tools

### `getDiscoveryContext()` — optional

Returns tenant-level context the skill cannot infer.

```
{
  contract_version: "2",
  strategy_summary?: string,
  non_goals?: string[],                  // quoted verbatim in out-of-scope disqualifications
  strategic_bets?: string[],             // topics that force Full tier
  segments?: { name: string, rule: string }[],
  arr_bands?: { name: string, min: number, max?: number }[],
  status_classes?: { active: string[], rejected: string[], shipped: string[], parked: string[] },
  thresholds?: {                         // overrides for the defaults in SKILL.md
    light_min_customers?: number,
    full_min_high_or_unknown_risks?: number,
    default_window_days?: number,
    run_cap?: number,
    similarity_floor?: number            // below this, similarity hits are not shown to the skill; default: none, top-N only
  }
}
```

If missing: out-of-scope and strategic-bet checks are skipped, defaults apply, and the proposal header says so.

### `listSignals({ status, since?, until?, page? })` — required

`status` is `"open"` for triage. Returns `{ items: SignalSummary[], next_page?: string, total?: number }`. The skill pages until it has the window or hits the run cap. `total`, when provided, lets the skill state the remainder accurately.

### `getSignal({ id })` — required

Returns `Signal`. Dismissed signals must return their `dismissal` block; the reasoning is what makes "previously rejected" possible.

### `searchSignals({ query | like_signal_id | like_opportunity_id, customer_id?, since?, include_dismissed?, statuses?, limit? })` — required

Semantic search at problem level, not keyword match. `statuses` filters by signal status (`open`, `linked`, `dismissed`); `include_dismissed: true` is shorthand for all three. `limit` defaults to 20. Results are ordered by similarity, highest first. Returns:

```
{ items: (SignalSummary & {
    customer: Customer | null,          // inline, so evidence roll-up needs no extra calls
    linked_opportunity_ids: string[],
    dismissal?: { reason_type, reason, related_opportunity_id?, at },
    similarity: number
  })[] }
```

Default `include_dismissed` is false. Per signal, the skill makes one call with `like_signal_id` and `include_dismissed: true`; that single result serves the prior-dismissal check, the evidence roll-up and clustering.

### `searchOpportunities({ query | like_signal_id | like_opportunity_id, statuses?, limit? })` — required

Semantic search, ordered by similarity. `limit` defaults to 10. `statuses` defaults to **all**, including rejected, archived and shipped; a server that silently filters to active opportunities breaks the prior-rejection check. Returns `{ items: OpportunitySummary[] }`. Rejected and archived items must carry `decision.reasoning`; when it is empty the skill says it cannot confirm a prior rejection.

### `compareSignals({ signal_ids })` — optional

Pairwise similarity between the stored embeddings of up to 50 signals, for clustering a window in one call. Returns `{ pairs: { a: string, b: string, similarity: number }[] }`, each unordered pair once, highest first, filtered by `similarity_floor` when the tenant sets one. Signals without a stored embedding are listed in `missing: string[]`.

### `getCustomer({ id })` — required

Returns `Customer & { open_opportunity_ids?: string[], signal_count?: number }`.

## Write tools

Called only after the user confirms. The server records the authenticated user as the actor; the skill supplies reasoning and whether the user overrode the recommendation.

### `createOpportunity({ title, problem, source, signal_ids, tier, triage })` — required

- `problem`: the normalized problem statement (who, what, situation, why)
- `source`: one line naming where the signals came from
- `signal_ids`: all signals in the cluster; linked as supporting evidence
- `tier`: `"light" | "standard" | "full"`; the server maps it to its pipeline status (typically Light → curated, Standard/Full → plan)
- `triage`: `{ record: <JSON record from proposal-format.md>, markdown: string }` stored on the opportunity

Returns `{ id, url }`.

### `linkSignalToOpportunity({ signal_id, opportunity_id, relation, note })` — required

`relation` is `"supporting"`, `"adjacent"` or `"post_launch"` (feedback on something the opportunity already shipped). `note` says what the signal adds ("adds SMB segment", "same account, nothing new", "praise", "gap in what shipped"). Marks the signal `linked`. Returns `{ ok: true }`.

### `dismissSignal({ signal_id, reason_type, reason, related_opportunity_id?, overrides_recommendation })` — required

`reason_type` is one of `previously_rejected | vague | out_of_scope | no_basis | not_discovery | other`. `not_discovery` covers bugs, support questions, context-only rows and post-launch feedback on a feature that has no opportunity to link to; the `reason` names which ("bug: column widths reset", "post-launch: Intercom integration") so the account, support or bug owner can find it. `reason` is the human-readable reasoning that future triage runs will read back through `searchSignals(include_dismissed: true)`. Returns `{ ok: true }`.

## Rules the server must honour

1. **Null means missing.** ARR, tier and segment are nullable. The skill never treats null as 0, and the server must not either.
2. **Stable ids and URLs** on every signal and opportunity, so proposal lines can link.
3. **Reasoning travels with decisions.** Rejected opportunities and dismissed signals carry reasoning text. This is the memory that stops the same ask from being re-triaged from scratch. A rejected item without reasoning is treated as related, not as a prior decision.
4. **Search is semantic.** "Audit trail for edits" must find "version history of descriptions". Reference search (`like_*`) uses the stored embedding and must not be slower than a text query.
5. **Paging is explicit.** The skill pages before concluding "none found".
6. **Writes are idempotent enough to re-check.** The skill calls `getSignal` before each write and skips signals no longer `open`. The server should reject a link or dismissal on a non-open signal rather than duplicating it.

## Degradation

| Missing or failing | Skill behaviour |
|---|---|
| `getDiscoveryContext` | defaults used; out-of-scope and strategic-bet checks skipped; stated in the header |
| `getCustomer` | ARR, segment, churn reported as `unknown` |
| `searchOpportunities` | prior-rejection and merge checks marked `not run`; every signal can only be tiered; stated loudly in the header |
| `searchSignals` | evidence roll-up marked `not run`; value risk `unknown` for every signal |
| `like_*` parameters (v1 server) or an item with no stored embedding | text `query` with the normalized problem, as in v1; stated once in the header |
| `compareSignals` | clusters are found from each signal's own `searchSignals` neighbours |
| `listSignals` | window selection unsupported; the skill asks for signal ids or pasted text |
| any write tool | the proposal is still produced; the skill tells the user which actions to apply in ProductSense by hand |
| a single call errors | that step is `not run` for that signal and the line says so; the run continues |

## Implementer notes

- Reference search is the main path: the skill calls `like_signal_id` for every signal in the window. Serve it from the vector index directly; there is nothing to embed.
- Re-embed a signal when its `extracted_problem` changes, and an opportunity when its title or problem changes; a stale embedding finds yesterday's neighbours.
- Return inline `customer` on `searchSignals` rows. Without it the roll-up for one signal can cost ten `getCustomer` calls.
- `total` on `listSignals` is cheap for the server and lets the skill say "25 of 83 assessed" instead of "there may be more".
- Keep `excerpt` short and `verbatim` complete. Populate `extracted_problem`: when the verbatim is content-free ("yeah, that would be important"), the extraction is what the skill can show.
- Any quote the server returns must come with an id and a link, whether it is a signal, a feedback item or a transcript excerpt. The skill will not use a quote it cannot link.
- Decision reasoning should be the human's own words, not a status code. The next triage run quotes it to the next human.
