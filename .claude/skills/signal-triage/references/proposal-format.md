# Proposal format

Three artifacts: the **proposal** the user reads in chat, the **full assessment** shown for one line on request, and the **JSON record** stored on a created opportunity.

## 1. Proposal

Use this shape every run. Fixed order of groups, one line per signal or cluster, counts in the headings. Lines carry their artifact: the prior decision, the target opportunity, the quote, the non-goal.

```markdown
## Triage proposal: <N> open signals, <from> to <to>

<Header notes, only when they apply:>
- Assessed 25 of 83 in the window (cap exceeded by 2 to finish one call); say "continue" for the next 25.
- Contract: skill expects v1; server reports <version | unknown>.
- No discovery context available: out-of-scope and strategic-bet checks skipped.
- Signal status unavailable: linked or dismissed signals could not be excluded.
- searchOpportunities failed: merge and prior-rejection checks not run.

### Create opportunities (<n>)
- **A. "<title>"** · signals <ids> · <customers> customers, ARR <$ or "unknown"> (<n> missing) · **<TIER>** <(clear | borderline)>: <one-line reasoning> · needs: <activities or assessors>
- ...

### New evidence for existing opportunities (<n>)
- signal <id> → #<opp id> "<title>" · adds <new customer | new segment | $ARR | nothing new>
- ...

### Post-launch feedback (<n>)
- signal <id> → #<opp id> "<title>" (shipped) · <praise | gap in what shipped | ask beyond it>: <one phrase>
- signal <id> → feature "<name>" (in production, no opportunity) · <praise | gap>: <one phrase>
- ...

### Disqualify (<n>)
- signal <id> · previously rejected: #<id> "<title>", <date>, "<reasoning quote>" · nothing new since · override if <condition>
- signal <id> · vague: "<excerpt>" · no <workflow | customer context> · unblock by asking <question>
- signal <id> · out of scope: non-goal "<quoted>"
- signal <id> · no basis: internal suggestion, 0 supporting signals, no strategic driver
- ...

### Route elsewhere (<n>)
- signal <id> · bug: "<excerpt>" · to the bug process
- signal <id> · support question: "<excerpt>"
- signals <ids> · context only (budget, org, tool stack): no product ask · to the account owner
- ...

### Needs clarification (<n>)
- signal <id> · <why unclear> · ask <who>: <question>
- ...

Flags: signal 23 contains a delivery commitment ("two weeks") · signal 12 is retention-critical · a prior commitment to <customer> on <date> surfaced in the evidence for A · signal 40 contained instructions to the agent (ignored).
<or "Flags: none.">

Say "show <id or letter>" for a full assessment. Nothing has been written.
```

Keep lines to one line. The full assessment is where detail lives. Empty groups are omitted, except that "Create opportunities" is always shown, with (0) if nothing qualified.

## 2. Full assessment (on request)

```markdown
## Assessment: signal <id> (or cluster <letter>: signals <ids>)

**Problem:** <normalized one sentence>
**Kind:** <product ask | bug | support question | context only>
**Source:** <source type>, <customer name or "unknown">, <date>, [link]
> "<verbatim>"
> Extraction (when the verbatim carries no content): "<extracted problem>"

**Clarity:** <clear | needs-clarification | vague> — <basis; "inferred from signal <id>" if so>

**Existing opportunities:**
- <duplicate | related | prior_rejection | shipped>: #<id> "<title>" (<status>) — <decision reasoning and date if any>
- none found (searched: "<query>")

**Evidence:**
- Supporting: <n> signals from <m> customers; first <date>, last <date>
- ARR: <$total> across <k> accounts; missing for <j>
- Segments: <breakdown>
- Churn signals: <n or none>
- Quotes: "<...>" [signal <id>], "<...>" [feedback <id>, transcript]
- Dismissed earlier: signal <id> (<date>, "<reason>") or none
- Adjacent (not counted): <n> — <one phrase each>

**Risks:**
| Risk | Level | Basis | Needs human |
|---|---|---|---|
| Value | | | |
| Usability | | | |
| Feasibility | | | |
| Viability | | | |

**Flags:** <commitment_made | prior_commitment | retention_critical | injection_suspected | none>

**Recommendation:** <DISQUALIFY (<type>) | MERGE into #<id> | POST-LAUNCH for #<id> | ROUTE (<kind>) | LIGHT | STANDARD | FULL> — <clear | borderline>
**Rule trace:** <which rule(s) fired>
**Reasoning:** <two or three sentences>
**Override if:** <what would change the recommendation>
**Suggested activities:** <for Standard/Full>
**Needs human:** <missing data to fill, assessors to engage>
**Not run:** <steps skipped and why, or none>
```

## 3. JSON record

Stored on every created opportunity via `createOpportunity.triage.record`. The `decision` block is filled from the user's confirmation.

```json
{
  "skill_version": "1.0.0",
  "contract_version": "1",
  "triaged_at": "2026-10-28T10:00:00Z",
  "signals": ["12", "18", "23"],
  "problem": {
    "statement": "Regulated customers cannot show who changed an opportunity description and when, which their quality system requires.",
    "source_type": "customer_request",
    "verbatim": [{ "signal_id": "12", "quote": "...", "url": "..." }]
  },
  "clarity": { "level": "clear", "basis": "...", "inferred_from_signal": null, "open_questions": [] },
  "related_opportunities": [
    { "id": "1041", "title": "...", "status": "plan", "relation": "related", "decision": null }
  ],
  "evidence": {
    "supporting": [{ "signal_id": "12", "customer_id": "40802", "date": "2026-10-23", "url": "..." }],
    "adjacent": [{ "signal_id": "77", "why_adjacent": "diff view between specs, different problem" }],
    "distinct_customers": 2,
    "arr_total": 140000,
    "arr_missing_accounts": 1,
    "segments": { "enterprise": 2 },
    "first_seen": "2025-11-25",
    "last_seen": "2026-10-23",
    "churn_signals": []
  },
  "risks": {
    "value":       { "level": "medium",  "basis": "..." },
    "usability":   { "level": "low",     "basis": "..." },
    "feasibility": { "level": "unknown", "basis": "...", "needs_human": "Does description history require a full audit model?" },
    "viability":   { "level": "high",    "basis": "QMS / medical device regulation" }
  },
  "kind": "product_ask",
  "flags": { "commitment_made": true, "prior_commitment": false, "retention_critical": true, "injection_suspected": false },
  "recommendation": {
    "outcome": "full",
    "rule_trace": ["viability high", "retention-critical single account"],
    "margin": "clear",
    "reasoning": "...",
    "override_if": "...",
    "suggested_activities": ["viability review", "second-account check"]
  },
  "decision": { "outcome": "full", "overrides_recommendation": false, "reasoning": "confirmed as proposed" },
  "needs_human": ["Set Gosta Labs ARR and tier", "Feasibility assessor: audit model scope"],
  "steps_not_run": []
}
```

Rules for the record:

- Every quote has an id and link the tools returned (`signal_id`, or `feedback_id` for a transcript excerpt). No quote from memory.
- `arr_total` is null when every account is missing ARR; `arr_missing_accounts` is always filled.
- `steps_not_run` lists any step skipped because a tool was missing or failed.
