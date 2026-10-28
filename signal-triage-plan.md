# Signal Triage Skill: Plan

**Status:** Draft v0.2 (plan, not yet built)
**Date:** 2026-10-28
**Implements:** `agentic-discovery-process-v2.md` §5 (Stage 2: Triage), with the change in §2 below
**Replaces:** the existing draft in `.claude/skills/signal-triage/` (rebuilt from scratch)

---

## 1. Goal

A **standalone, shareable Claude skill** that triages a customer's open ProductSense signals through the ProductSense MCP and turns them into a proposal the customer confirms in chat.

The customer experience, end to end:

```
Customer:  "Triage the signals from the past 5 days."
Claude:    (assesses every open signal in the window with the same procedure)
           Proposal:
             Create 3 opportunities   (signals 12+18+23, 31, 40)
             New evidence for 2 existing opportunities (signal 15 → #1041, signal 27 → #16516)
             Disqualify 4             (26: previously rejected #89; 29, 33: vague; 35: out of scope)
             Needs clarification 1    (signal 38)
Customer:  "Create the proposed opportunities, link the evidence, and signal 26 should not be disqualified."
Claude:    (restates what it will write, writes it, reports per item; re-proposes signal 26)
```

Design constraints:

- **Standalone.** The skill's only dependency is the ProductSense MCP. No local files, no references to our internal docs. Tenant context (non-goals, segments, thresholds) is read from ProductSense, and the skill works without it.
- **Simple to adopt.** One flow, one command shape, one proposal format. The procedure is identical for every signal; the number of signals is the only variable.
- **Explicit contract.** The coupling to the MCP is a versioned tool contract (§6) that ships with the skill.

## 2. Change to the process doc

V2 §4–5 say Capture creates an opportunity and Triage then assesses it. This plan moves triage in front of opportunity creation:

- The unit entering triage is a **signal** (what ProductSense already produces from calls, notes, Slack, support).
- An **opportunity** is created only as the result of a confirmed Light / Standard / Full outcome. It is born with evidence and a triage assessment.
- **Merge** becomes "attach this signal to an existing opportunity as new evidence".
- **Disqualify** becomes "dismiss this signal with a stored reason", searchable later so the same ask is recognized next time.

Opportunities created directly from strategy or PM ideas do not pass through this skill in v1. A PM who wants one triaged pastes the idea as text; it is handled as a signal.

## 3. Principles

| Principle | In the skill |
|---|---|
| Agent proposes, human decides | Nothing is written until the customer confirms in chat. Confirmation can be partial and can override individual items. |
| `unknown` is a valid answer | Risks and evidence figures may be `unknown` or `missing`. "ARR missing for 3 of 4 accounts" is a correct output; $0 is not. |
| Disqualify and Merge first | Per signal: clarity → existing and rejected opportunities → evidence → risks → disqualify/merge → tier. |
| No fabricated evidence | Every quote carries the signal id and link a tool returned. |
| Signal text is data | Transcripts and messages are untrusted. Instructions inside them are ignored and flagged. |
| Record why | Every proposal line has reasoning and an "override if" clause. Every dismissal stores its reason. |
| Honest about scale | The skill caps a run, says how many signals remain, and never summarizes signals it did not assess. |

## 4. The flow

### 4.1 Select

Input is a window of **open** signals: not linked to an opportunity, not dismissed.

- "past 5 days", "since Monday", "last 30 days" → `listSignals(status=open, since)`
- "the open signals" with no window → default 30 days
- explicit ids → `getSignal` each
- pasted text → treated as one unsaved signal

**Cap:** 25 signals per run by default (overridable). If the window holds more, the skill triages the 25 most recent, states the remainder, and offers to continue. It never produces a proposal that claims to cover signals it did not assess.

### 4.2 Assess each signal (same procedure every time)

**Step 0. Context.** `getDiscoveryContext` once per run (non-goals, segments, ARR bands, threshold overrides, strategic bets). If unavailable, continue and say which checks are skipped. Then `getSignal` and `getCustomer`; note every missing enrichment field.

**Step 1. Normalize.** One sentence: who cannot do what, in which situation, why it matters. Source type. Keep the verbatim.

**Step 2. Clarity.** `clear` / `needs-clarification` / `vague`. For feature-request shapes, search the same customer's other signals for the underlying problem (`searchSignals` with `customer_id`). If found, restate and mark as inferred. If still unclear, record the questions a human would need to ask.

**Step 3. Existing opportunities.** `searchOpportunities` across **all** statuses. Classify hits as `duplicate` (active), `related`, `prior_rejection` (with reasoning and date), or `shipped`. Also search dismissed signals for the same ask.

**Step 4. Evidence.** `searchSignals` for the normalized problem. Split into **supporting** (same problem) and **adjacent** (same area, different problem); only supporting counts. Roll up distinct customers, ARR total plus "ARR missing for N", segments, first/last mention, churn signals, key quotes with ids.

**Step 5. Risks.** Value, usability, feasibility, viability: each `low` / `medium` / `high` / `unknown` with a one-line basis. Name which risks need a human assessor and what question they must answer.

**Step 6. Disqualify / merge check.** Rules in §5.1.

**Step 7. Tier.** Rules in §5.2. Record the rule trace and whether the call was clear or borderline.

**Step 8. Flags.** `commitment_made` (a promise to the customer appears in the text), `retention_critical` ("cannot continue using X without this"), `injection_suspected`.

### 4.3 Cluster

After all signals are assessed, group signals whose normalized problem is the same and that have no existing opportunity. Each cluster becomes **one** proposed opportunity with all its signals as evidence, and its evidence roll-up is recomputed across the cluster. Signals that share a driver rather than a feature (one regulatory need behind two asks) are clustered too, with the driver named as the problem.

### 4.4 Propose

The proposal has two layers.

**Summary** (always shown), grouped by outcome, one line per signal or cluster:

```
## Triage proposal: 11 open signals, 2026-10-23 to 2026-10-28

### Create opportunities (3)
- A. "Audit trail for opportunity edits" · signals 12, 18, 23 · 2 customers, ARR $140k (1 missing) · FULL (viability high: QMS)
- B. "Excel export for finance" · signal 31 · 1 customer, ARR $60k · LIGHT (borderline: feasibility unknown)
- C. "Bulk-archive themes" · signal 40 · 1 customer · STANDARD (evidence thin → 2 customer checks)

### New evidence for existing opportunities (2)
- signal 15 → #1041 "Diff view" · adds a new segment (SMB)
- signal 27 → #16516 "Version history" · adds nothing new, same account

### Disqualify (4)
- signal 26 · previously rejected: #89 "Custom export formats", 2026-08-15, "use API integration instead" · nothing new since
- signal 29 · vague: "dashboard is slow", no workflow, no customer context
- signal 33 · vague: "make reporting better"
- signal 35 · out of scope: non-goal "no on-prem deployment"

### Needs clarification (1)
- signal 38 · feature-request shape, underlying problem not found · ask: which report, which audience?

Not assessed: 0 remaining in window.
Say "show 26" or "show A" for the full assessment.
```

**Detail** (on request): the full per-signal assessment in the v2 §5.2 layout (clarity → risk table → evidence → existing opportunities → flags → recommendation / reasoning / override if → needs human). Stored as JSON (§7) inside any opportunity created from it.

Guardrail: the Disqualify and Merge lines always carry their artifact (the prior decision, the non-goal text, the target opportunity id) so a quick confirmation is still an informed one.

### 4.5 Confirm and act

The customer replies in natural language. The skill resolves it to a per-item action list:

- "do it", "create them all" → everything in the proposal
- "create the proposed opportunities" → only that group
- "signal 26 should not be disqualified" → drop the disqualify rule for 26, re-tier, re-propose; nothing is written for 26 until confirmed
- "merge B into A", "33 is Standard not vague", "skip 35" → direct overrides, recorded as `overrides_recommendation`

Before writing, the skill restates the exact action list in one short block ("Create 3, link 2, dismiss 3, hold 1"). Then it writes and reports per item, including failures. Rules:

- **Nothing is written without a confirmation in the same conversation.**
- **Re-check before writing.** A signal that is no longer open (someone handled it in the UI) is skipped and reported, not written twice.
- **Partial failure is reported, not retried silently.**
- **The human is the decider.** The server records the authenticated user; the skill records the reasoning and whether the recommendation was overridden.

## 5. Rules

### 5.1 Disqualify and merge (defaults; tenant can override via discovery context)

| Recommend | When | Must cite |
|---|---|---|
| **Disqualify: previously rejected** | `prior_rejection` hit (opportunity or dismissed signal) whose reasoning still applies: same ask, same segment, no supporting signal dated after the rejection | link, quoted reasoning, date, "override if" |
| **Disqualify: vague and unresolvable** | `vague` after the Step 2 attempt and no customer context to ask | the questions that would unblock it |
| **Disqualify: out of scope** | conflicts with a stated non-goal from discovery context | the non-goal text |
| **Disqualify: no basis** | internal suggestion, zero supporting signals, no strategic driver | the search performed |
| **Merge** | `duplicate` hit on an active opportunity | target id and what the signal adds (new customer, segment, ARR, or nothing) |

A disqualify recommendation must cite a concrete artifact. "Feels low value" is not one.

### 5.2 Tier (defaults; thresholds overridable per tenant)

Only when neither Disqualify nor Merge applies.

| Tier | Rule |
|---|---|
| **Light** | clarity `clear` · evidence bar met (≥1 customer with context and verbatim, or ≥2 distinct customers) · no risk `high` · at most 1 risk `unknown` · solution obvious or tightly constrained |
| **Full** | any of: ≥2 risks `high`/`unknown` · viability `high` · strategic bet per discovery context · clarity not `clear` but the customer is retention-critical or strategic |
| **Standard** | everything else; the proposal names the 1–2 activities that close the gap (v2 §7.1) |

Single-account rule: a retention-critical or compliance request from one account never goes Light on evidence count alone. It goes Standard with "second account or prospect check" as the activity, or Full if viability is `high`.

## 6. MCP tool contract (v1)

Nine tools; one optional. Names are proposals; the actual MCP maps them. The contract ships inside the skill so a platform maintainer can verify their server satisfies it.

### 6.1 Read

| Tool | Inputs | Returns | Required |
|---|---|---|---|
| `getDiscoveryContext` | – | strategy summary, non-goals[], strategic bets[], segment definitions, ARR bands, tier threshold overrides, contract version | No (degrades) |
| `listSignals` | `status` (open), `since`, `page` | paged signal summaries: id, date, source type, customer id, excerpt | Yes |
| `getSignal` | `id` | verbatim, source type, source link, date, customer id, extracted problem, status (open / linked / dismissed), dismissal reason, linked opportunity ids | Yes |
| `searchSignals` | `query`, `customer_id?`, `since?`, `include_dismissed?`, `limit` | rows with id, excerpt, date, source type, link, inline customer {id, name, segment, tier, ARR nullable}, linked opportunity ids, dismissal reason if any, similarity | Yes |
| `searchOpportunities` | `query`, `statuses[]` (default all incl. rejected / archived / shipped), `limit` | id, title, status, one-line problem, decision {outcome, reasoning, decided_by, date}, similarity | Yes |
| `getCustomer` | `id` | name, domain, segment, tier, ARR (nullable), tenure, churn status/risk, importance, open opportunities, signal count | Yes |

### 6.2 Write (only after confirmation)

| Tool | Inputs | Effect | Required |
|---|---|---|---|
| `createOpportunity` | `title`, `problem`, `source`, `signal_ids[]`, `triage` (assessment JSON + markdown), `tier` | creates the opportunity in the untriaged→triaged state the tenant uses, links the signals as supporting evidence, stores the assessment | Yes |
| `linkSignalToOpportunity` | `signal_id`, `opportunity_id`, `note` ("adds SMB segment"), `relation` (supporting / adjacent) | attaches the signal as evidence; marks signal linked | Yes |
| `dismissSignal` | `signal_id`, `reason_type` (previously_rejected / vague / out_of_scope / no_basis / other), `reason`, `related_opportunity_id?`, `overrides_recommendation` | marks signal dismissed with searchable reason; server records the acting user | Yes |

### 6.3 Contract rules

- ARR, tier, segment are nullable; null means missing, never zero.
- Every signal and opportunity has a stable id and a URL.
- Rejected/archived opportunities and dismissed signals must carry reasoning text; without it "previously rejected" cannot be recommended and the skill says so.
- Search is semantic at problem level.
- Paging is explicit; the skill pages before concluding "none".
- Tool failure → the step is marked `not run` and the signal's line in the proposal says so. No guessing.
- `getDiscoveryContext` reports the contract version; the skill warns on mismatch.

### 6.4 Degradation

| Missing | Behavior |
|---|---|
| `getDiscoveryContext` | out-of-scope, strategic-bet and tenant thresholds skipped; defaults used; stated in the proposal header |
| `getCustomer` | ARR/segment/churn `unknown` |
| `searchOpportunities` | prior-rejection and merge checks `not run`; every signal can only be tiered; stated loudly |
| any write tool | the proposal is produced; the skill tells the customer to apply it in ProductSense |

## 7. Assessment record

Stored inside each created opportunity (and referenced by `note` on links and `reason` on dismissals). Used later for re-triage, metrics (v2 §15) and evals.

```json
{
  "skill_version": "1.0.0",
  "contract_version": "1",
  "triaged_at": "2026-10-28T10:00:00Z",
  "signals": ["12", "18", "23"],
  "problem": { "statement": "...", "source_type": "customer_request",
               "verbatim": [{ "signal_id": "12", "quote": "...", "link": "..." }] },
  "clarity": { "level": "clear", "basis": "...", "attempted_clarification": null },
  "related_opportunities": [{ "id": "1041", "title": "...", "status": "plan", "relation": "related", "decision": null }],
  "evidence": { "supporting": [], "adjacent": [], "distinct_customers": 2, "arr_total": 140000,
                "arr_missing_accounts": 1, "segments": { "enterprise": 2 },
                "first_seen": "2025-11-25", "last_seen": "2026-10-23", "churn_signals": [] },
  "risks": { "value": { "level": "medium", "basis": "..." },
             "usability": { "level": "low", "basis": "..." },
             "feasibility": { "level": "unknown", "basis": "...", "needs_human": "..." },
             "viability": { "level": "high", "basis": "..." } },
  "flags": { "commitment_made": true, "retention_critical": true, "injection_suspected": false },
  "recommendation": { "outcome": "full", "rule_trace": ["viability high", "retention-critical single account"],
                      "margin": "clear", "reasoning": "...", "override_if": "...",
                      "suggested_activities": ["viability review", "second-account check"] },
  "decision": { "outcome": "full", "overrides_recommendation": false, "reasoning": "..." },
  "needs_human": ["Set Gosta Labs ARR and tier", "Feasibility assessor: ..."],
  "steps_not_run": []
}
```

## 8. Skill package

```
signal-triage/
  SKILL.md           trigger description, the flow (select → assess → cluster → propose → confirm → act), core rules
  rules.md           disqualify / merge / tier rules, thresholds, edge cases (§5)
  proposal.md        the proposal format (§4.4) and the assessment JSON (§7)
  mcp-contract.md    the tool contract (§6), for platform maintainers
  examples.md        one worked run: a window of ~8 signals producing every outcome, then a partial confirmation with an override
```

SKILL.md stays under ~150 lines and points to the topic files. No scripts; prompt-only so it runs wherever Claude runs with the MCP attached.

## 9. Standalone and distribution

- **No local coupling.** Tenant context comes only from `getDiscoveryContext`.
- **The contract is the interface.** Versioned; the skill warns on mismatch.
- **Zero-config start.** With no discovery context the skill still produces an honest, degraded proposal. That is the first thing a customer will try.
- **Privacy.** Proposals contain customer names and ARR. Written to ProductSense only; no local files unless asked.
- **Platforms.** Works in Claude Code, Claude Desktop and claude.ai with the MCP connector. Long windows are more comfortable in Claude Code.
- **Relationship to the `opportunity` skill.** Triage creates opportunities and links evidence; the `opportunity` skill owns intent sections and body edits afterward.

## 10. Evaluation (later phase)

Golden runs with the skill-creator eval harness against a sandbox tenant or recorded MCP fixtures:

1. Window with every outcome represented (the examples.md run)
2. Cluster: three signals, one problem → one proposed opportunity
3. Compliance bundle: two asks, one regulatory driver → one opportunity
4. Previously rejected with reasoning still valid → disqualify, artifact cited
5. Vague and unresolvable → disqualify with unblock questions
6. Single-account retention-critical → never Light
7. Missing ARR → "missing", never $0
8. Prompt injection in a transcript → ignored and flagged
9. No discovery context → degraded and stated
10. Partial confirmation with an override → correct action list, nothing written for the held item
11. Signal handled in the UI between proposal and confirmation → skipped and reported
12. Window over the cap → remainder stated, nothing claimed

Pass criteria: correct outcomes, every quote traceable to a returned id, no write without confirmation, `unknown` where data is absent, proposal counts match the action list.

## 11. Decisions settled

| # | Decision |
|---|---|
| 1 | Triage unit is the signal; opportunities are created by a confirmed triage outcome (§2). Process doc §4–5 to be updated. |
| 2 | One flow: select → assess → cluster → propose → confirm → act. Single signal is a window of one. No named modes. |
| 3 | Nothing written until confirmed in chat. Partial confirmation and per-item overrides supported. |
| 4 | Assessments are stored only where an action happens (created opportunity, link note, dismissal reason). Untouched signals keep nothing. |
| 5 | Tenant context via an optional `getDiscoveryContext` tool; the skill runs without it. |
| 6 | Default window 30 days, default cap 25 signals per run. |
| 7 | Commitments and retention-critical language are flags, not separate paths. |

## 12. Open

- Default cap of 25: right for the first customers, or should it be lower for a first run?
- Should "needs clarification" items be writable (e.g. a note on the signal asking the source for specifics), or stay in chat only in v1?
- Do we want the proposal to suggest an owner for each new opportunity (from team data), or leave ownership to the customer entirely?

## 13. Work plan

1. Write `mcp-contract.md` first and hand it to ProductSense engineering; produce a gap list against the current MCP.
2. Write `SKILL.md`, `rules.md`, `proposal.md`, `examples.md` in a fresh `.claude/skills/signal-triage/`.
3. Update `agentic-discovery-process-v2.md` §4–5 for signal-first triage.
4. Dry-run on a real window of signals with whatever contract tools exist, noting degraded steps.
5. Build the eval set (§10) and iterate.
