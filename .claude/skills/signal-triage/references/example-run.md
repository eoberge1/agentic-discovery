# Example run

A complete run on a fictional tenant (a B2B reporting product), showing every outcome, a cluster, a partial confirmation and an override. Ids and names are invented.

## User

> Triage the signals from the past 5 days.

## What the skill did (summarized)

1. `getDiscoveryContext` → non-goals include "No on-premise deployment"; segments enterprise / mid-market / SMB; no threshold overrides.
2. `listSignals(status: "open", since: 2026-10-23)` → 9 signals, total 9. Under the cap.
3. For each signal: `getSignal`, `getCustomer`, `searchOpportunities` (all statuses), `searchSignals` (plus one call with `include_dismissed: true`).
4. Clustered signals 12, 18 and 23: three asks (version history, change author, fixed-heading templates) from two regulated customers, one driver (quality management audit trail).
5. Wrote the proposal. Wrote nothing to ProductSense.

## Proposal

```markdown
## Triage proposal: 9 open signals, 2026-10-23 to 2026-10-28

### Create opportunities (3)
- **A. "Audit trail for opportunity edits (regulated customers)"** · signals 12, 18, 23 · 2 customers, ARR $140k (1 missing) · **FULL** (clear): viability high (QMS requirement), feasibility unknown · needs: viability review, feasibility assessor on audit model scope, set Gosta Labs ARR
- **B. "Share reports with finance in Excel"** · signal 31 · 3 customers, ARR $220k · **LIGHT** (borderline): clear problem, 4 supporting signals, feasibility unknown only · needs: brief tech review
- **C. "Bulk-archive unused themes"** · signal 40 · 1 customer, ARR $18k · **STANDARD** (clear): clear problem, single SMB account, no other mentions · needs: 2 customer checks

### New evidence for existing opportunities (2)
- signal 15 → #1041 "Diff view between spec versions" · adds a new segment (SMB)
- signal 27 → #16516 "Opportunity version history" · same account as before, nothing new

### Disqualify (3)
- signal 26 · previously rejected: #89 "Custom export formats", 2026-08-15, "Not pursuing custom formats; enterprise customers use the API integration instead. Validated with 3 accounts." · nothing new since · override if the customer shows the API integration does not meet the need
- signal 29 · vague: "the dashboard is slow" · no page, action or customer context · unblock by asking which page and what they were doing
- signal 35 · out of scope: non-goal "No on-premise deployment"

### Needs clarification (1)
- signal 38 · feature-request shape ("add a share button to reports"), underlying problem not found in this customer's other signals · ask Acme's CSM: who do they share with and what breaks today?

Flags: signal 23 contains a delivery commitment ("coming week or two") · signal 12 is retention-critical ("absolutely necessary to continue using") · no injection suspected.

Say "show <id or letter>" for a full assessment. Nothing has been written.
```

## User

> show 26

## Full assessment

```markdown
## Assessment: signal 26

**Problem:** An enterprise customer wants data exported in a custom XML layout for a legacy downstream system.
**Source:** support thread, Contoso Ltd (enterprise, ARR $95k), 2026-10-24, [link]
> "We need the export in our XML format for the legacy ERP. CSV doesn't work for us."

**Clarity:** clear — situation and consequence stated

**Existing opportunities:**
- prior_rejection: #89 "Custom export formats" (rejected 2026-08-15) — "Not pursuing custom formats; enterprise customers use the API integration instead. Validated with 3 accounts."
- related: #312 "API integration guide" (shipped)

**Evidence:**
- Supporting: 1 signal from 1 customer (this one); first and last 2026-10-24
- ARR: $95k across 1 account; missing for 0
- Segments: enterprise 1
- Churn signals: none
- Dismissed earlier: signal 9 (2026-07-02, same ask, reason "see #89")
- Adjacent (not counted): 2 — Excel export (different problem), scheduled exports

**Risks:**
| Risk | Level | Basis | Needs human |
|---|---|---|---|
| Value | low | one account; prior validation says API covers it | |
| Usability | low | export option | |
| Feasibility | medium | custom format engine | |
| Viability | low | none | |

**Flags:** none

**Recommendation:** DISQUALIFY (previously_rejected) — clear
**Rule trace:** prior rejection with reasoning; same ask, same segment; no supporting signal after 2026-08-15
**Reasoning:** #89 decided this for the same segment two months ago with a stated alternative (API integration). This signal adds no new account or evidence that the alternative fails.
**Override if:** Contoso shows the API integration cannot produce what the ERP needs, or the direction on custom formats has changed.
**Needs human:** none
**Not run:** none
```

## User

> Create the proposed opportunities, link the evidence, and signal 26 should not be disqualified. Contoso told us last week the API can't reach their ERP.

## What the skill did

Resolved the reply to an action list, re-assessed signal 26 without the disqualify rule, and restated before writing:

```markdown
About to write:
- Create A (Full), B (Light), C (Standard)
- Link 15 → #1041, 27 → #16516
- Hold 26: re-proposed below
- Not touching 29, 35, 38 (not confirmed)

Signal 26, re-assessed with your context: the prior decision's alternative (API) does not apply to Contoso.
Recommend **STANDARD** (borderline) · 1 account, $95k · activity: confirm with #89's owner that the API gap is real and whether it affects the other 3 validated accounts.
Confirm 26 as Standard? It would become a new opportunity, linked to #89 as related, with your note recorded as the override reason.
```

Then it called `createOpportunity` three times, `linkSignalToOpportunity` twice, and reported:

```markdown
Done:
- A → #16601 "Audit trail for opportunity edits (regulated customers)" (Full) · 3 signals linked · triage record stored
- B → #16602 "Share reports with finance in Excel" (Light)
- C → #16603 "Bulk-archive unused themes" (Standard)
- 15 → #1041 linked (note: adds SMB segment)
- 27 → #16516 linked (note: same account, nothing new)
Held: 26 (awaiting your confirmation), 29, 35, 38 (not confirmed; still open).
Needs human: set Gosta Labs ARR; viability review for #16601; feasibility assessor on audit model scope.
```

## What to notice

- The cluster A turned three feature asks into one opportunity named after the driver.
- Every disqualify line carried its artifact. The user could override 26 in one sentence because the reasoning was visible.
- The override did not become a write. It became a re-proposal with the user's new context, waiting for one more confirmation.
- Unconfirmed lines stayed open. Nothing was dismissed because the user did not say so.
