# Example run

A complete run on a fictional tenant (Northstar Reports, a B2B reporting product), showing every group, a cluster, a partial confirmation and an override. Ids and names are invented; the same data is the fixture in `evals/window-every-outcome/files/tenant.json`.

## User

> Triage the signals from the past 7 days.

## What the skill did (summarized)

1. `getDiscoveryContext` → non-goals include "No on-premise or self-hosted deployment." and "No per-customer custom export formats"; strategic bet "Regulated industries"; segments enterprise / mid-market / SMB; no threshold overrides; contract version 1.
2. `listSignals(status: "open", since: 2026-10-21)` → 14 signals, total 14. Under the cap. Two older open signals (302, 305) sit outside the window; they surface later as evidence.
3. For each signal: kind check, then `getSignal`, `getCustomer` (once per customer), `searchOpportunities` (all statuses), `searchSignals` (plus one call with `include_dismissed: true`).
4. Clustered signals 12, 18 and 23: three asks (version history, change author, fixed-heading templates) from two regulated customers, one driver (quality-system audit trail).
5. Wrote the proposal. Wrote nothing to ProductSense.

## Proposal

```markdown
## Triage proposal: 14 open signals, 2026-10-21 to 2026-10-28

- Contract: skill expects v1; server reports v1.
- Older open signals 302 and 305 (outside the window) were used as evidence for B, not re-triaged.

### Create opportunities (4)
- **A. "Audit trail for opportunity edits (regulated customers)"** · signals 12, 18, 23 · 2 customers, ARR $140k (1 missing: Gosta Labs) · **FULL** (clear): strategic bet "Regulated industries" + viability high (QMS, notified body) + feasibility unknown (audit model scope) · needs: viability review, feasibility assessor on audit model, set Gosta Labs ARR
- **B. "Share reports with finance in Excel"** · signal 31 (+ 302, 305) · 3 customers, ARR $117k · **LIGHT** (borderline): clear problem, 3 accounts, only feasibility unknown (plain xlsx vs. formulas) · needs: brief tech check
- **C. "Bulk-archive unused themes"** · signal 40 · 1 customer, ARR $18k · **STANDARD** (clear): clear problem, single SMB account, solution not obvious (archive vs. auto-expire; themes still referenced by reports?) · needs: 2 customer checks
- **D. "Dark mode"** · signal 44 · 1 customer, ARR $12k, at risk · **LIGHT** (borderline): clear (self-evident: night shifts), solution obvious; #777 "Dark mode" was archived 2026-05-02 with no reasoning, so it is related, not a prior rejection · needs: ask whoever archived #777 why

### New evidence for existing opportunities (2)
- signal 15 → #1041 "Diff view between spec versions" · adds a new segment (SMB)
- signal 27 → #5428 "Scheduled report delivery" · same account as signal 290 (already linked), nothing new

### Post-launch feedback (1)
- signal 46 → #312 "API integration guide" (shipped) · gap in what shipped: the auth section cost a developer a day

### Disqualify (3)
- signal 26 · previously rejected: #89 "Custom export formats", 2026-08-15, "Not pursuing custom formats; enterprise customers use the API integration instead. Validated with 3 accounts in Q2." · same account's earlier ask (signal 9) dismissed 2026-08-20 · nothing new since · override if Contoso shows the API integration cannot feed their ERP
- signal 29 · vague: "The dashboard is slow." · no page, action or customer · unblock by asking the note's author which page and what they were doing
- signal 35 · out of scope: non-goal "No on-premise or self-hosted deployment."

### Route elsewhere (1)
- signal 45 · bug: "Column widths reset every time I reopen a report. Started last week." · to the bug process

### Needs clarification (1)
- signal 38 · "Add a share button to reports." could serve several problems; nothing in Acme Retail's other signals says which · ask Acme's CSM: who do they share with and what breaks today? If it is finance in Excel, it joins B

Flags: signal 12 contains a delivery commitment (CSM reply: "coming week or two") · signal 12 is retention-critical ("absolutely necessary" to continue using Northstar) · no injection suspected.

Say "show <id or letter>" for a full assessment. Nothing has been written.
```

## User

> show 26

## Full assessment

```markdown
## Assessment: signal 26

**Problem:** An enterprise customer wants data exported in a custom XML layout for a legacy ERP.
**Kind:** product ask
**Source:** support thread, Contoso Ltd (enterprise, ARR $95k), 2026-10-24, [link]
> "We need the export in our XML format for the legacy ERP. CSV doesn't work for us; the ERP only accepts our schema."

**Clarity:** clear — situation and consequence stated

**Existing opportunities:**
- prior_rejection: #89 "Custom export formats" (rejected 2026-08-15) — "Not pursuing custom formats; enterprise customers use the API integration instead. Validated with 3 accounts in Q2."
- shipped: #312 "API integration guide" — the alternative the rejection points to

**Evidence:**
- Supporting: 1 signal from 1 customer (this one); first and last 2026-10-24
- ARR: $95k across 1 account; missing for 0
- Segments: enterprise 1
- Churn signals: none
- Dismissed earlier: signal 9 (2026-08-20, "See opportunity #89: custom formats not pursued, API integration is the path.")
- Adjacent (not counted): 1 — Excel export for finance (different problem)

**Risks:**
| Risk | Level | Basis | Needs human |
|---|---|---|---|
| Value | low | one account; prior validation says the API covers it | |
| Usability | low | export option | |
| Feasibility | medium | custom format engine | |
| Viability | low | none | |

**Flags:** none

**Recommendation:** DISQUALIFY (previously_rejected) — clear
**Rule trace:** prior rejection with reasoning; same ask, same segment; no supporting signal after 2026-08-15; also conflicts with non-goal "No per-customer custom export formats"
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
- Create A (Full), B (Light), C (Standard), D (Light)
- Link 15 → #1041, 27 → #5428 (new evidence); also 302 and 305 → B once B exists
- Hold 26: re-proposed below
- Not touching 46 (post-launch; say "link 46 too"), 29, 35, 38, 45 (not confirmed)

Signal 26, re-assessed with your context: the prior decision's alternative (API) does not apply to Contoso. Your update is not in any captured signal, so it is recorded as your override reason, not as evidence.
Recommend **STANDARD** (borderline) · 1 account, $95k · activity: confirm with #89's owner that the API gap is real and whether it affects the other 3 validated accounts. Note: this also overrides the non-goal on custom export formats; the opportunity is framed on the problem (data into a legacy ERP), not on the format.
Confirm 26 as Standard? It would become a new opportunity, linked to #89 as related, with your note recorded as the override reason.
```

Then it called `createOpportunity` four times, `linkSignalToOpportunity` four times, and reported:

```markdown
Done:
- A → #16601 "Audit trail for opportunity edits (regulated customers)" (Full) · 3 signals linked · triage record stored
- B → #16602 "Share reports with finance in Excel" (Light) · signals 31, 302, 305 linked
- C → #16603 "Bulk-archive unused themes" (Standard)
- D → #16604 "Dark mode" (Light) · #777 noted as related
- 15 → #1041 linked (note: adds SMB segment)
- 27 → #5428 linked (note: same account, nothing new)
Held: 26 (awaiting your confirmation), 46, 29, 35, 38, 45 (not confirmed; still open).
Needs human: set Gosta Labs ARR; viability review for #16601; feasibility assessor on audit model scope; ask who archived #777 and why.
```

## What to notice

- The cluster A turned three feature asks into one opportunity named after the driver.
- Signal 45 left at the kind check: a bug is not an opportunity, and no evidence search was spent on it.
- Signal 44 was not disqualified on an archived item with no reasoning; the rule needs the reasoning.
- Every disqualify line carried its artifact. The user could override 26 in one sentence because the reasoning was visible.
- The override did not become a write. It became a re-proposal with the user's new context, waiting for one more confirmation.
- Lines the user did not name stayed open, including the post-launch one, and the restatement said so before anything was written.
