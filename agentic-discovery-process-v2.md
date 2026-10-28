# Agentic Discovery: Process Definition

**Status:** Draft v2.1
**Owner:** Product Management
**Last updated:** 2026-10-28
**Previous version:** `agentic-discovery-process.md` (v0.2)
**Related:** [The AI-Native SDLC Playbook](https://claude.com/resources/articles/the-ai-native-sdlc-playbook)

---

## 1. Purpose

This document defines the **Agentic Discovery** process that sits before the **AI-native SDLC** (Plan → Design → Build → Test → Deploy → Maintain). Discovery decides what is worth building and produces an evidence-backed `intent.md` ready for implementation.

**Key change from v1:** The process is now **tiered**. Not every opportunity needs full discovery. A triage agent assesses each incoming signal and routes it appropriately—simple, well-framed requests move fast; uncertain or risky bets get deeper investigation.

**Change in v2.1:** Triage runs on **signals**, before an opportunity exists. An opportunity is created only when the owner confirms a triage proposal (§4–5). The `signal-triage` skill is the normative definition of the triage procedure.

> **Guiding principle:** *The depth of discovery should match the uncertainty of the problem.*

---

## 2. The Discovery Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           STARTING POINTS                                   │
│  Strategy · PM ideas · Customer feedback · Organization · Post-launch       │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         1. CAPTURE                                          │
│      Signals created from inputs (calls, notes, Slack, support)             │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         2. TRIAGE                                           │
│   Agent assesses each open signal, clusters, proposes; owner confirms       │
│   Output: Disqualify / New evidence / Create (Light / Standard / Full)      │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                 ┌──────────────────┼──────────────────┐
                 ▼                  │                  ▼
┌────────────────────────┐          │     ┌─────────────────────────────────┐
│     DISQUALIFIED       │          │     │      NEW EVIDENCE               │
│                        │          │     │  Link signal to existing opp.   │
│  • Already rejected    │          │     └─────────────────────────────────┘
│  • Vague/unactionable  │          │
│  • No path forward     │          │
└────────────────────────┘          │
                          ┌─────────┴─────────┐
                          ▼                   ▼
┌─────────────────────────────────┐   ┌─────────────────────────────────────┐
│        3a. CURATED              │   │        3b. PLAN                     │
│   Ready for prioritization      │   │   Needs more discovery work         │
│   (Light tier)                  │   │   (Standard / Full tier)            │
│                                 │   │                                     │
│   • Problem is clear            │   │   • Problem validation              │
│   • Evidence exists             │   │   • Solution validation             │
│   • Risks are low/known         │   │   • Customer interviews             │
│   • Solution is obvious         │   │   • Prototypes                      │
│                                 │   │   • Technical spikes                │
│                                 │   │   • Assumption tests                │
└─────────────────────────────────┘   └─────────────────────────────────────┘
                          │                   │
                          └─────────┬─────────┘
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         4. PRIORITIZE                                       │
│         Decide which intent.md's go to implementation                       │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
                            ┌───────────────┐
                            │   intent.md   │───► AI-native SDLC
                            │   (accepted)  │
                            └───────────────┘
```

---

## 3. Starting Points

Opportunities can originate from anywhere. The source affects how much context arrives with them.

| Source | Typical context | Common gaps |
|--------|-----------------|-------------|
| **Strategy / OKRs** | Clear outcome, may have segment | Problem specifics, customer evidence |
| **Product Manager** | Problem hypothesis, maybe solution ideas | Customer validation, evidence |
| **Customer feedback** (support, sales, CS) | Verbatim problem, specific customer | Broader evidence, segment weighting |
| **Organization** (engineering, ops, legal) | Technical or operational constraint | Customer impact, priority vs other work |
| **Post-launch learning** | Outcome data, what didn't work | What to try next |
| **Competitor moves** | What they shipped | Whether our customers care |

All sources feed into the same process. The triage agent normalizes them.

---

## 4. Stage 1: Capture

**What:** A **signal** is the atomic unit entering discovery: one piece of input with its verbatim text, its source and, where known, its customer. ProductSense creates signals automatically from the inputs it reads (call transcripts, meeting notes, Slack messages, support threads). Anyone can also add one by hand, for example by pasting a message into the triage conversation.

**Who:** Mostly automatic. A human adds manual signals and, where ProductSense could not, names the customer.

**Minimum content of a signal:**

- Verbatim text, as the originator said it
- Source: where it came from (call, thread, ticket, doc), with a link
- Customer, if known (this is what makes ARR, segment and churn context possible)

**Change from v2.0:** Capture no longer creates an opportunity. Opportunities are created by triage (Stage 2) once a signal, or a cluster of signals, has been assessed and the owner has confirmed. This keeps the backlog free of opportunities that would be merged or disqualified immediately, and means every opportunity is born with evidence and a triage assessment. Ideas that originate internally (strategy, a PM, engineering) enter the same way, written down as a signal with source type `internal`.

**Artifact:** Open signal in ProductSense.

**Gate:** None. Capture is open.

---

## 5. Stage 2: Triage

**What:** The triage agent takes a window of open signals, assesses each one with the same procedure, clusters signals that describe the same problem, and presents one proposal. The opportunity owner confirms the proposal in chat, in full or in part, and can override any line. Only then does anything change in ProductSense.

**Who:** Run on demand by whoever owns the inbox ("triage the signals from the past 5 days"). The opportunity owner reviews and confirms.

**Skill:** `signal-triage`. It is standalone (depends only on the ProductSense MCP) and is the normative definition of the procedure; this section summarizes it.

### 5.1 The flow

```
select → assess each signal → cluster → propose → confirm → act
```

| Step | What happens | Writes to ProductSense |
|---|---|---|
| **Select** | Open signals in a window (default 30 days, cap 25 per run; the remainder is stated, never implied) | – |
| **Assess** | Per signal, in fixed order: clarity → existing and rejected opportunities → evidence → risks → disqualify/merge check → tier → flags (§5.2) | – |
| **Cluster** | Signals with the same problem, or the same underlying driver, become one proposed opportunity | – |
| **Propose** | Grouped proposal: create / new evidence / disqualify / needs clarification (§5.3) | – |
| **Confirm** | Owner replies in chat; partial confirmation and per-line overrides are allowed; an override becomes a re-proposal, not a write | – |
| **Act** | Create opportunities, link evidence, dismiss signals with reasons; re-check each signal is still open first | yes |

### 5.2 What the agent assesses per signal

#### A. Problem clarity

Distinguishes between:
- **Clear problem:** "I can't share reports with finance because they need Excel format"
- **Feature request:** "I want CSV export" (the *why* is missing)
- **Vague:** "Reporting is hard"

Output: `clear` / `needs-clarification` / `vague`

For a feature request, the agent searches the same customer's other signals for the underlying problem and, if found, restates it and marks the clarity as inferred. If it cannot, it records the questions a human would have to ask.

#### B. Existing opportunity check

```
searchOpportunities (all statuses, incl. rejected / archived / shipped)
searchSignals (include_dismissed)   → the same ask dismissed earlier
```

Output per hit:
- **Duplicate:** same problem, active opportunity → merge candidate
- **Related:** same area, different problem → mentioned, not merged
- **Prior rejection:** rejected or archived with reasoning → disqualify candidate; reasoning and date kept
- **Shipped:** already addressed → post-launch feedback for that opportunity, unless the ask goes beyond what shipped

#### C. Evidence search

```
searchSignals   → related signals; each row carries its customer (segment, tier, ARR)
getCustomer     → tier, ARR, churn status, importance
```

Hits are split into **supporting** (same problem) and **adjacent** (same area, different problem); only supporting hits count.

Output:
- Distinct customers, signal count, first and last mention
- ARR represented **and** "ARR missing for N accounts" (missing is never zero)
- Segment breakdown per the tenant's definitions
- Churn signals among those customers
- Key verbatim quotes, each with a signal id and link

#### D. Risk scan

| Risk | What agent checks | Possible outputs |
|------|-------------------|------------------|
| **Value** | Supporting evidence, churn or retention language, strategic fit | `low` / `medium` / `high` / `unknown` |
| **Usability** | New interaction pattern? Multi-step workflow? New type of user? | `low` / `medium` / `high` / `unknown` |
| **Feasibility** | Multiple systems? Data model change? Unclear scope? Dependencies? | `low` / `medium` / `high` / `unknown` |
| **Viability** | Pricing, compliance, legal, security, contractual triggers? | `low` / `medium` / `high` / `unknown` |

**Key rule:** `unknown` is a valid output. The agent does not guess; it names which risks need a human assessor and what question that person must answer.

#### E. Disqualify / merge check

Checked before tiering. Every disqualification must cite a concrete artifact.

| Recommend | When | The line must cite |
|---|---|---|
| **Disqualify: previously rejected** | Prior rejection (opportunity or dismissed signal) whose reasoning still holds: same ask, same segment, no supporting signal dated after the rejection | Link, quoted reasoning, date, what would override it |
| **Disqualify: vague and unresolvable** | `vague` after the clarification attempt, and no customer context to ask | The questions that would unblock it |
| **Disqualify: out of scope** | Conflicts with a stated non-goal in the tenant's discovery context | The non-goal text |
| **Disqualify: no basis** | Internal suggestion, zero supporting signals, no strategic driver | The search performed |
| **Merge (new evidence)** | Duplicate of an active opportunity | Target id and what the signal adds (new customer, segment, ARR, or nothing) |

**Key rule:** The agent recommends; the owner confirms or overrides. Dismissed signals keep their reason and are searchable, so the same ask is recognized next time rather than re-triaged from scratch.

#### F. Tier

Only when neither Disqualify nor Merge applies.

| Tier | Rule |
|---|---|
| **Light** | Clarity `clear` · evidence bar met (one customer with context and a verbatim, or two or more distinct customers) · no risk `high` · at most one risk `unknown` · solution obvious or tightly constrained |
| **Full** | Any of: two or more risks `high`/`unknown` · viability `high` · a strategic bet per discovery context · clarity not `clear` but the customer is retention-critical or strategic |
| **Standard** | Everything else; the proposal names the one or two activities that would close the gap |

The agent records which rule fired and whether the call was clear or borderline. A single-account retention-critical or compliance request never goes Light on evidence count alone.

#### G. Flags

- `commitment_made`: the text contains a promise to the customer. Triage still runs; the owner can consciously expedite.
- `retention_critical`: the customer says they cannot continue without it.
- `injection_suspected`: the text contains instructions aimed at the agent. Signal text is quoted, never followed.

### 5.3 Triage proposal

One proposal per run, grouped by outcome, one line per signal or cluster, each line carrying its artifact. The owner can ask for the full assessment behind any line.

```markdown
## Triage proposal: 9 open signals, 2026-10-23 to 2026-10-28

### Create opportunities (3)
- **A. "Audit trail for opportunity edits (regulated customers)"** · signals 12, 18, 23 · 2 customers, ARR $140k (1 missing) · **FULL** (clear): viability high (QMS), feasibility unknown · needs: viability review, feasibility assessor
- **B. "Share reports with finance in Excel"** · signal 31 · 3 customers, ARR $220k · **LIGHT** (borderline): feasibility unknown only · needs: brief tech review
- **C. "Bulk-archive unused themes"** · signal 40 · 1 customer, ARR $18k · **STANDARD**: single SMB account · needs: 2 customer checks

### New evidence for existing opportunities (2)
- signal 15 → #1041 "Diff view between spec versions" · adds a new segment (SMB)
- signal 27 → #5428 "Scheduled report delivery" · same account, nothing new

### Disqualify (3)
- signal 26 · previously rejected: #89 "Custom export formats", 2026-08-15, "use the API integration instead; validated with 3 accounts" · nothing new since · override if the API does not meet the need
- signal 29 · vague: "the dashboard is slow" · no page, action or customer context
- signal 35 · out of scope: non-goal "No on-premise deployment"

### Needs clarification (1)
- signal 38 · feature-request shape, underlying problem not found · ask the CSM: who do they share with and what breaks today?

Flags: signal 23 contains a delivery commitment · signal 12 is retention-critical.
Say "show <id or letter>" for a full assessment. Nothing has been written.
```

The full per-signal assessment (clarity, risk table, evidence, existing opportunities, flags, recommendation, reasoning, override-if, needs-human) and the JSON record stored on created opportunities are defined in the skill's `references/proposal-format.md`.

### 5.4 Triage outcomes

| Outcome | Criteria | What happens on confirmation |
|---------|----------|------------------------------|
| **Disqualify** | Previously rejected (reasoning still valid) OR vague/unactionable OR out of scope OR no basis | Signal dismissed with reason type, reasoning and link to the prior decision; searchable later; can be reopened |
| **New evidence (merge)** | Duplicate of an active opportunity | Signal linked to that opportunity with a note on what it adds |
| **Light** | Problem clear + evidence exists + risks low or known + solution obvious | Opportunity created with its signals and triage record → Curated |
| **Standard** | Problem clear but evidence thin OR 1–2 risks medium/unknown | Opportunity created → Plan (targeted validation, activities named) |
| **Full** | Problem unclear OR multiple high/unknown risks OR strategic bet | Opportunity created → Plan (full discovery) |
| **Needs clarification** | Feature-request shape, underlying problem not found | Nothing written; the proposal names who to ask and what |

**Processing order:** Disqualify and Merge are checked first. Only if neither applies is the tier assessed.

**Human decision:** The agent proposes. The owner confirms in chat, in full or in part. An unconfirmed line stays open; an overridden line is re-proposed with the owner's context before anything is written.

---

## 6. Stage 3a: Curated (Light Tier)

**What:** Opportunities that are ready for prioritization without further discovery work.

**Criteria:**
- Problem is clear and well-articulated
- Evidence exists (at least 1 customer + context OR multiple signals)
- Risks are low or known (no major unknowns)
- Solution is obvious or constrained

**What happens:**
1. Opportunity owner reviews triage output
2. Intent author completes the `intent.md` if needed (constraints, non-goals, scope fence)
3. Opportunity moves to prioritization backlog

**Artifact:** Completed lightweight `intent.md`:

```markdown
# Intent: <title>

## Problem
<Who has the problem, in what situation, why it matters>

## Evidence
- **Source:** <customer, ticket, call>
- **Quote:** "<verbatim>"
- **Other mentions:** <N customers, $X ARR>

## Proposed outcome
<What success looks like>

## Solution
<What we'll build—can be brief>

## Scope fence
<What we're NOT doing>

## Risks
<Any known risks or unknowns flagged by triage>

## Success metric
<Simple: "X customers use it within 30 days" or specific outcome>

## Approval
- Approved by: <name, date>
```

---

## 7. Stage 3b: Plan (Standard / Full Tier)

**What:** Opportunities that need more discovery work before they're ready for prioritization.

**When:**
- Evidence is thin (need to validate the problem)
- Solution is unclear (need to validate approach)
- Risks are medium/high/unknown (need to reduce uncertainty)
- Strategic importance warrants deeper investigation

### 7.1 Discovery Activities

The opportunity owner selects which activities are needed based on what's uncertain:

| Activity | When to use | Output |
|----------|-------------|--------|
| **Problem validation** | Evidence thin, problem unclear | Customer interviews, signal analysis |
| **Solution validation** | Multiple approaches possible | Concept tests, prototype feedback |
| **Customer interviews** | Need direct customer input | Interview notes, verbatim quotes |
| **Prototype** | Usability risk, need to show concept | Clickable prototype, test results |
| **Technical spike** | Feasibility risk, scope unclear | Feasibility assessment, scope estimate |
| **Assumption test** | Specific hypothesis to validate | Test card with result |
| **Competitive analysis** | Unclear differentiation | Competitive brief |
| **Viability review** | Pricing, compliance, legal questions | Stakeholder sign-off |

### 7.2 Standard Tier (Targeted Validation)

For opportunities with 1-2 specific gaps:

- Pick the 1-2 activities that address the gaps
- Time-box: days, not weeks
- Goal: resolve the unknowns, then move to Curated

Example: "Evidence is thin" → do a quick signal search + 2-3 customer conversations → update opportunity → re-triage or move to Curated.

### 7.3 Full Tier (Full Discovery)

For opportunities with significant uncertainty or strategic importance:

- Multiple activities may be needed
- May require the four validation filters from v1:
  1. Verbatim quote audit (≥3 distinct quotes)
  2. Revenue and cohort segmentation
  3. Negative space and churn check
  4. Temporal window analysis
- Prototypes must stay disposable (sandbox, time-boxed)
- Intent author writes core of `intent.md` personally (not delegated to agents)

**Artifact:** Full `intent.md` with:
- Discovery evidence section
- Risk assessment (all four types)
- Test cards for assumption tests run
- Measurement plan (for high-stakes launches)

### 7.4 Exit from Plan

When uncertainty is sufficiently reduced:
1. Update the opportunity with findings
2. Complete the `intent.md`
3. Move to Curated (prioritization backlog)

Or: if discovery reveals the opportunity isn't viable → Archive with learnings.

---

## 8. Stage 4: Prioritize

**What:** Decide which opportunities with completed `intent.md` go to implementation.

**Who:** Org-dependent. May be a product manager, product owner, team, or leadership group.

**Inputs:**
- Curated opportunities (from Light tier or after Plan)
- Strategic priorities (`DISCOVERY.md`)
- Capacity constraints
- Dependencies

**Decision factors:**
- Evidence strength (how confident are we this matters?)
- Impact (outcome size × segment size)
- Effort (from feasibility assessment)
- Strategic fit (alignment with OKRs)
- Dependencies and sequencing

**Output:** Accepted `intent.md` → enters SDLC

**Gate:** Opportunity owner accepts the `intent.md`. High-risk intents also need:
- Feasibility assessor sign-off (if feasibility risk flagged)
- Viability assessor sign-off (if viability risk flagged)

---

## 9. Artifacts Summary

| Artifact | Created at | Contains | Lives in |
|----------|------------|----------|----------|
| **Signal** | Capture | Verbatim, source, customer | ProductSense |
| **Triage proposal** | Triage | Per-signal recommendations with their artifacts | Chat; the record is stored on created opportunities |
| **Opportunity** | Triage (on confirmation) | Problem, linked signals, triage record | ProductSense |
| **intent.md (draft)** | After triage | Problem, source, proposed outcome | Attached to opportunity |
| **intent.md (complete)** | Curated or Plan exit | Full intent ready for SDLC | Attached to opportunity, exported to repo |
| **Test cards** | Plan (if needed) | Hypothesis, method, result | Linked from opportunity |
| **Interview notes** | Plan (if needed) | Customer conversations | Linked from opportunity |

---

## 10. Human / Agent Responsibility Split

| Stage | Agent does | Human owns |
|-------|------------|------------|
| **Capture** | Extracts signals from inputs | Adds manual signals, names the customer |
| **Triage** | Assesses clarity, scans risks, searches evidence, finds matches and prior rejections, clusters, proposes | Confirms or overrides the proposal in chat; owns the created opportunities |
| **Plan** | Prepares interview briefs, drafts guides, finds evidence, scaffolds prototypes | Conducts interviews, makes judgment calls, writes core intent |
| **Prioritize** | May score/rank based on criteria | Makes the prioritization decision |

**Gates (human-only):**
- Triage outcome decision (disqualify / merge / tier)
- Move to Curated (problem validated)
- Accept `intent.md` (enters SDLC)
- Reopen a disqualified opportunity
- Prioritization decisions

See §11 for how responsibilities are assigned flexibly across people.

---

## 11. Responsibilities

Traditional role boundaries (PM, Designer, Engineer) are fluid. What matters is that the responsibilities below are fulfilled—by whoever has the skills and context to do them well.

### 11.1 Responsibilities That Must Be Assigned

Each opportunity needs clear ownership of these responsibilities. One person may hold several; responsibilities may shift as the opportunity moves through stages.

| Responsibility | What it means | Typical owner | Required capability |
|----------------|---------------|---------------|---------------------|
| **Opportunity owner** | Accountable for moving the opportunity through discovery. Makes triage decisions, ensures work happens, signs off on intent. | Whoever captures it, or explicitly assigned | Decision authority, context on the problem |
| **Customer contact** | Talks to actual customers. Takes own notes. Cannot be delegated to agents. | Anyone with access and conversation skills | Customer access, listening skills |
| **Value assessment** | Judges whether customers want this and whether it matters to the business | Anyone with customer/market understanding | Customer empathy, business context |
| **Usability assessment** | Judges whether users can figure it out and whether it fits their workflow | Anyone with UX sensibility | User observation skills, design judgment |
| **Feasibility assessment** | Judges whether we can build it, at what cost, with what risks | Anyone with technical understanding | System knowledge, engineering judgment |
| **Viability assessment** | Judges business model fit, compliance, legal, pricing implications | Anyone with business/policy context | Policy knowledge, stakeholder access |
| **Intent author** | Writes the core of `intent.md`: problem, outcome, constraints, non-goals | Opportunity owner (usually) | Writing clarity, problem understanding |
| **Prioritization decision** | Decides which opportunities get built next | Org-dependent (may be PM, may be team, may be leadership) | Strategic context, capacity visibility |
| **Platform maintenance** | Maintains MCP connections, triage agent, skills, evals | Assigned per org | Technical skills, agent tuning |

### 11.2 How Responsibilities Flow

```
Opportunity created
       │
       ▼
┌─────────────────────────────────────────────────────────────┐
│  OPPORTUNITY OWNER assigned (or defaults to creator)        │
│  → Responsible for triage decision, moving through stages   │
└─────────────────────────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────────────────┐
│  RISK ASSESSORS engaged as needed                           │
│  → Value, Usability, Feasibility, Viability                 │
│  → May be same person, may be different people              │
│  → Triage agent suggests which risks need human assessment  │
└─────────────────────────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────────────────┐
│  CUSTOMER CONTACT happens (Plan stage)                      │
│  → Opportunity owner may do it, or assign to someone closer │
│  → Must be a human, must take own notes                     │
└─────────────────────────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────────────────┐
│  INTENT AUTHOR writes intent.md                             │
│  → Usually opportunity owner                                │
│  → Could be delegated if someone else has better context    │
└─────────────────────────────────────────────────────────────┘
       │
       ▼
┌─────────────────────────────────────────────────────────────┐
│  PRIORITIZATION DECISION made                               │
│  → Org-specific: PM, product council, team, leadership      │
│  → Uses input from all risk assessors                       │
└─────────────────────────────────────────────────────────────┘
```

### 11.3 What Agents Do (and Don't Do)

Agents handle:
- Ingesting and enriching signals
- Assessing problem clarity
- Scanning for risks and flagging unknowns
- Searching for evidence
- Finding related/prior opportunities
- Recommending triage outcomes
- Drafting artifacts (briefs, guides, intent sections)
- Preparing context for human decisions

Agents **never**:
- Make triage decisions (disqualify, tier, merge)
- Accept or reject opportunities
- Approve intent.md
- Replace customer conversations
- Make prioritization decisions

### 11.4 Flexibility Principle

The goal is **coverage, not titles**. An organization might:
- Have engineers own opportunities in their domain
- Have PMs create prototypes
- Have designers write intent.md
- Have a rotating "discovery lead" responsibility
- Have cross-functional pairs share opportunity ownership

What matters:
1. Every opportunity has a clear owner
2. All four risk types get assessed (by someone qualified)
3. Customer contact happens (by a human)
4. Decisions are made and recorded

---

## 12. Skills and Agents

| Skill / Agent | Stage | Purpose |
|---------------|-------|---------|
| `signal-triage` | Triage | Triage a window of open signals: assess each, cluster, propose create / new evidence / disqualify, act on the owner's confirmation |
| `opportunity` | All | Manage opportunities in ProductSense, draft and export intent.md |
| `evidence-search` | Triage, Plan | Search for related signals, quotes, customer context |
| `customer-360` | Plan | Pre-interview dossier |
| `interview-guide` | Plan | Draft past-behaviour discussion guide |
| `assumption-mapper` | Plan (Full) | List riskiest assumptions, propose tests |
| `intent-interrogator` | Plan (Full) | Adversarial review of intent.md |
| `prototype-scaffold` | Plan | Generate disposable prototypes for validation |

---

## 13. Anti-patterns

| Anti-pattern | Why it fails | Guardrail |
|--------------|--------------|-----------|
| **Everything goes through Full discovery** | Slows down clear requests, wastes effort | Triage agent + tier system |
| **Everything goes Light** | Skips validation on uncertain bets | Triage must flag unknowns; opportunity owner accountable for tier choice |
| **Over-disqualifying** | Good ideas get killed by lazy pattern matching | Disqualify needs clear reasoning; opportunity owner reviews all disqualifications |
| **Under-disqualifying** | Same rejected ideas keep coming back, wasting triage effort | Agent checks prior rejections; reasoning must be recorded |
| **Disqualifying without recording why** | Can't learn, can't reopen appropriately | Disqualification requires linked reasoning |
| **Synthetic users as validation** | No real-world friction, sycophancy | Never valid as evidence |
| **Agent decides tier** | Removes human judgment | Agent recommends, human decides |
| **Prototypes become production** | No security, observability | Sandbox + time-box |
| **Skip triage for "urgent" requests** | Creates invisible risk | All opportunities go through triage |
| **No one talks to customers** | Loses intuition, relies on secondhand data | Customer contact responsibility must be assigned and fulfilled |

---

## 14. Integration with SDLC and Post-Launch Learning

### Discovery → SDLC

The **only** output of discovery that enters the SDLC is an **accepted `intent.md`**. Accepting it triggers the SDLC Plan/Design pass.

### Post-Launch Learning → Discovery

After a feature launches, post-launch learning may route findings back to discovery:

| Verdict | Route back |
|---------|------------|
| **Miss / Pivot** | Re-open opportunity at Plan stage |
| **Partial** | New opportunity or additional validation |
| **New problem surfaced** | New opportunity at Capture |
| **Hit** | Mark opportunity validated, capture learnings |

See `agentic-discovery-process.md` (v1) §6 for full post-launch learning process.

---

## 15. Metrics

| Metric | What it measures |
|--------|------------------|
| **Triage → decision time** | How fast opportunities get routed |
| **Tier distribution** | % Light / Standard / Full (are we appropriately tiering?) |
| **Plan stage cycle time** | How long opportunities spend in Plan |
| **Evidence quality** | % of prioritized intents with customer evidence |
| **Outcome hit rate** | % of shipped intents that hit their primary outcome |
| **Waste avoided** | # of opportunities archived before SDLC (saved build effort) |

---

## 16. Open Questions

- ~~Specific thresholds for tier recommendation~~ Defined in the `signal-triage` skill (§5.2 F); tenants can override via discovery context. To be validated with the first customers.
- ~~Automatic or on-demand triage?~~ On demand per window in v2.1 ("triage the past 5 days"). Automatic runs are a later option once the proposal format has proven itself.
- How do we handle urgent/expedited requests?
- What's the SLA for triage assessment?
- Should Light tier intents still get post-launch learning, or only adoption tracking?

---

## 17. Migration from v1

The v2 process simplifies the v1 stages:

| v1 Stage | v2 Equivalent |
|----------|---------------|
| D0 Frame | `DISCOVERY.md` still exists as strategic context |
| D1 Sense | Signals feed into Capture |
| D2 Map | Triage agent + evidence search |
| D3 Verify | Triage agent (for Light) or Plan stage (for Standard/Full) |
| D4 Engage | Plan stage activities |
| D5 Test | Plan stage activities |
| D6 Commit | Exit from Curated or Plan → Prioritize |

Post-launch learning (L1-L5) remains as defined in v1, triggered for intents that declare outcome metrics.
