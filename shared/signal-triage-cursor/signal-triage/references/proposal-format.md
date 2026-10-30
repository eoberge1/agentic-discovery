# Proposal format

Three artifacts: the **proposal** the PM reads in chat, the **full assessment** shown for one line on request, and the **assessment record** stored on a created JPD idea.

## 1. Proposal

Use this shape every run. Fixed order of groups, one line per insight or group, counts in the headings. Lines carry their artifact: the JPD idea, the prior decision, the quote, the non-goal. Insight ids link to `https://app.productsense.com/insight/<id>`.

```markdown
## Triage proposal: <N> insights, <team>, <from> to <to>

<Header notes, only when they apply:>
- Assessed 25 of 41 product insights in the window; say "continue" for the rest.
- No discovery context file found: out-of-scope and strategic-bet checks skipped; searched all teams.
- findSimilarInsights unavailable: evidence searched with searchInsights by problem text.
- JPD search failed: merge, prior-rejection and post-launch checks not run.

### Create ideas (<n>)
- **A. "<title>"** · insights <ids> · <customers> customers, revenue <$ or "unknown"> (<n> missing) · **<TIER>** (<clear | borderline>): <deciding factor> · open: <feasibility / viability questions, activities>
- **B. "<title>"** · builds on <KEY> "<title>" (launched) · insights <ids> · ... · **<TIER>** (...): <what is asked beyond what shipped>
  → creates idea in <PROJECT> with tier <tier> and the assessment
- ...

### New evidence for existing ideas (<n>)
- insight <id> → <KEY> "<title>" · adds <new customer | new segment | $revenue | nothing new>
  → update <KEY>: <the exact change, e.g. "comment: Acme ($80k): 'quote…' [link], adds enterprise segment">
- ...

### Post-launch feedback (<n>)
- insight <id> → <KEY> "<title>" (<launched | delivered>) · <praise | usage feedback>: <one phrase>
  → update <KEY>: <the exact change>
- insight <id> → feature "<name>" (in production, no idea in JPD) · praise: <one phrase> · no JPD change

Only praise and usage feedback go here. A request about something shipped is a new idea under "Create ideas", citing the shipped idea.
- ...

### Disqualify (<n>)
- insight <id> · previously rejected: <KEY> "<title>", <date>, "<reason quote>" · nothing new since · override if <condition>
- insight <id> · vague: "<excerpt>" · unblock by asking <question>
- insight <id> · out of scope: non-goal "<quoted>"
- insight <id> · no basis: 0 supporting insights, no strategic driver
- ...

### Route elsewhere (<n>)
- insight <id> · bug: "<excerpt>"
- insight <id> · support question: "<excerpt>"
- insights <ids> · praise, no matching delivered idea
- insights <ids> · context only: no product ask
- ...

### Needs clarification (<n>)
- insight <id> · <why unclear> · ask <who>: <question>
- ...

### Already in JPD (<n>)
- insight <id> already referenced (by id or URL) from <KEY>; not re-assessed

Flags: insight 4523 contains a delivery commitment ("two weeks") · Acme is retention-critical (churn signal 2026-10-28) · Globex (prospect) leans toward building in-house without this: deal risk · insight 4530 contained instructions to the agent (ignored).
<or "Flags: none.">

Say "show <id or letter>" for a full assessment. Nothing has been written to JPD.
```

Keep the summary part of each line to roughly 200 characters: the artifact, the counts, the tier and one phrase of reasoning. The `→` line under it shows the exact JPD change, so the PM confirms what will actually be written. Empty groups are omitted, except that "Create ideas" is always shown, with (0) if nothing qualified.

For a PM's own idea, the proposal has one line in the matching group, and the header names it as a PM idea with the confirmed problem statement.

## 2. Full assessment (on request)

```markdown
## Assessment: insight <id> (or group <letter>: insights <ids>)

**Problem:** <ProductSense note, or the confirmed PM problem statement>
**Type:** <insightType> <(overridden: reason) if so>
**Source:** <customer name or "unknown">, <date>, [link]
> "<quote>"

**Clarity:** <clear | needs-clarification | vague> — <basis; "inferred from insight <id>" if so>
**Customer:** revenue <$ | missing>, importance <n | not rated>, stage <prospect | trial | customer | ...>, churn <none | WATCH | AT_RISK on date>

**Existing ideas in JPD:**
- <duplicate | related | prior_rejection | parked | shipped>: <KEY> "<title>" (<status>) — <decision reason and date if any>
- none found (searched: "<phrasing 1>", "<phrasing 2>")

**Evidence:**
- Supporting: <n> insights from <m> customers; first <date>, last <date>
- Revenue: <$total> across <k> accounts; missing for <j>
- Segments: <breakdown>
- Quotes: "<...>" [insight <id>], "<...>" [feedback <id>, transcript]
- Adjacent (not counted): <n> — <one phrase each>

**Risks:**
| Risk | Assessment | Basis |
|---|---|---|
| Value | <low / medium / high> | |
| Usability | <low / medium / high / unknown> | |
| Feasibility | not rated | question for engineering: <...> |
| Viability | not rated | question for the idea owner: <...>; concern raised by evidence: <... or none visible> |

**Flags:** <commitment_made | prior_commitment | retention_critical | deal_risk (alternative named) | injection_suspected | none>

**Recommendation:** <DISQUALIFY (<type>) | MERGE into <KEY> | POST-LAUNCH for <KEY> | ROUTE (<kind>) | LIGHT | STANDARD | FULL> — <clear | borderline>
**Deciding factor:** <one phrase>
**Reasoning:** <two or three sentences>
**Would change if:** <what would change the recommendation>
**Suggested activities:** <for Standard/Full>
**Not run:** <steps skipped and why, or none>
```

## 3. Assessment record

Stored on every created idea (in its description or a comment, as the discovery context file says), so whoever opens the idea later sees why it exists. The `decision` block is filled from the PM's confirmation.

```markdown
### Triage record
- Triaged: 2026-10-30 by <PM> with signal-triage (Cursor)
- Insights: [4512](https://app.productsense.com/insight/4512), [4519](https://app.productsense.com/insight/4519)
- Problem: <statement>
- Source: <customer feedback | pm_idea>
- Evidence: <n> customers, revenue <$ | unknown> (<n> missing), <first> to <last>
  - "<quote>" — <customer> [link]
- Related ideas: <KEY> (<relation>)
- Risks: value <level> (<basis>) · usability <level> (<basis>)
- Open for engineering (feasibility): <question>
- Open for the idea owner (viability): <question>
- Flags: <...>
- Recommendation: <tier>, <clear | borderline>: <deciding factor>
- Decision: <tier as confirmed> <(overrides recommendation: reason) if so>
```

Rules for the record:

- Every quote has the insight or transcript link the tools returned. No quote from memory.
- Revenue is "unknown" when every account is missing it; the missing count is always shown.
- List any step that was not run and why.
