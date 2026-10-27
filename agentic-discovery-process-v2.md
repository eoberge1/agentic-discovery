# Agentic Discovery: Process Definition

**Status:** Draft v2.0
**Owner:** Product Management
**Last updated:** 2026-10-27
**Previous version:** `agentic-discovery-process.md` (v0.2)
**Related:** [The AI-Native SDLC Playbook](https://claude.com/resources/articles/the-ai-native-sdlc-playbook)

---

## 1. Purpose

This document defines the **Agentic Discovery** process that sits before the **AI-native SDLC** (Plan → Design → Build → Test → Deploy → Maintain). Discovery decides what is worth building and produces an evidence-backed `intent.md` ready for implementation.

**Key change from v1:** The process is now **tiered**. Not every opportunity needs full discovery. A triage agent assesses each incoming signal and routes it appropriately—simple, well-framed requests move fast; uncertain or risky bets get deeper investigation.

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
│           Create opportunity with draft intent.md                           │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                                    ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         2. TRIAGE                                           │
│    Agent assesses: problem clarity, risks, evidence, existing opportunities │
│    Output: Disqualify / Merge / Light / Standard / Full                     │
└─────────────────────────────────────────────────────────────────────────────┘
                                    │
                 ┌──────────────────┼──────────────────┐
                 ▼                  │                  ▼
┌────────────────────────┐          │     ┌─────────────────────────────────┐
│     DISQUALIFIED       │          │     │        MERGE                    │
│                        │          │     │   Add to existing opportunity   │
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
│         PM / PO decides which intent.md's go to implementation              │
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

**What:** Create an opportunity with a draft `intent.md`. This is the atomic unit of discovery.

**Who:** Anyone can create an opportunity. The PM owns it once created.

**How:** Use the `opportunity` skill or create directly in ProductSense. The opportunity must contain at minimum:

```markdown
# Intent: <title>

## Problem
<Who has the problem, what they can't do, why it matters>

## Source
<Where this came from: customer name, ticket, call, strategy doc, etc.>

## Proposed outcome (optional)
<What success looks like, if known>
```

**Artifact:** Opportunity in ProductSense with draft `intent.md` attached.

**Gate:** None. Capture is open—anyone can add to the backlog.

---

## 5. Stage 2: Triage

**What:** The triage agent assesses each new opportunity and enriches it with context. This is the key innovation—front-loading intelligence so routing is informed.

**Who:** Agent runs automatically on new opportunities. PM reviews output.

### 5.1 What the Triage Agent Does

#### A. Problem Clarity Check

Distinguishes between:
- **Clear problem:** "I can't share reports with finance because they need Excel format"
- **Feature request:** "I want CSV export" (the *why* is missing)
- **Vague:** "Reporting is hard"

Output: `clear` / `needs-clarification` / `vague`

If not clear, the agent attempts to find the underlying problem from evidence. If it can't, it flags for human clarification.

#### B. Risk Scan

Quick assessment across four risk types:

| Risk | What agent checks | Possible outputs |
|------|-------------------|------------------|
| **Value** | Is there evidence customers want this? Other mentions? ARR represented? | `low` / `medium` / `high` / `unknown` |
| **Usability** | Is the interaction complex? New patterns? | `low` / `medium` / `high` / `unknown` |
| **Feasibility** | Multiple systems? Unclear scope? Dependencies? | `low` / `medium` / `high` / `unknown` |
| **Viability** | Pricing, compliance, legal triggers? | `low` / `medium` / `high` / `unknown` |

**Key rule:** `unknown` is a valid output. The agent should not guess—it surfaces what it can't determine.

#### C. Evidence Search

Uses MCP tools to find related signals:

```
searchInsights    → related feedback from other customers
findEvidence      → verbatim quotes from transcripts
identifyCompany   → context on who's asking
getCustomerDetails → tier, ARR, churn status
listChurnSignals  → related churn risk signals
```

Output:
- Number of other customers mentioning this
- Total ARR represented
- Key verbatim quotes (linked)
- Segment breakdown (enterprise / mid-market / SMB)

#### D. Existing Opportunity Check

```
listOpportunities → scan for related problems
getOpportunity    → check if signal fits existing opportunity
```

Output:
- **Match found:** link to existing opportunity, recommend merging
- **Related:** link to related opportunity, flag relationship
- **Previously rejected:** link to rejected opportunity with reasoning
- **None:** new opportunity

#### E. Disqualification Check

The triage agent can recommend **disqualification** when continuing would waste effort. Disqualification is not a judgment call—it's based on clear signals.

**Disqualify when:**

| Condition | Example | What agent does |
|-----------|---------|-----------------|
| **Previously rejected with valid reasoning** | Existing opportunity rejected 3 months ago with PM note "Not doing because conflicts with enterprise architecture direction" and nothing has changed | Link to prior decision, quote the reasoning, recommend disqualify |
| **Vague and unresolvable** | "Make it better" / "Improve performance" with no specifics and no way to clarify | Flag as vague, note no actionable path, recommend disqualify |
| **Out of scope per DISCOVERY.md** | Request clearly conflicts with stated non-goals or strategic boundaries | Quote the relevant non-goal, recommend disqualify |
| **Duplicate of active work** | Exact same ask as an opportunity already in Plan or Curated | Link to active opportunity, recommend merge instead |
| **No customer or strategic basis** | Internal suggestion with no customer evidence and no strategic driver | Note absence of basis, recommend disqualify |

**Key rule:** The agent recommends disqualification with reasoning. The PM confirms or overrides. Disqualified opportunities are archived with the reason, not deleted—they can be reopened if context changes.

**Disqualification output:**

```markdown
**Recommended action:** DISQUALIFY
**Reason:** Previously rejected (2026-08-15)
**Prior decision:** Opportunity #89 "Custom export formats"
**PM reasoning:** "Not pursuing custom formats—enterprise customers use
our API integration instead. Validated with 3 enterprise accounts."
**Current signal adds:** Nothing new—same request, same segment.
**Override if:** Strategic direction has changed, or new evidence shows
API integration isn't meeting the need.
```

### 5.2 Triage Output

The agent produces a structured assessment attached to the opportunity.

**Example 1: Routed to Light tier**

```markdown
## Triage Assessment

**Problem clarity:** Clear
> "I can't share data with my finance team because they only work in Excel"

**Risk scan:**
| Risk | Level | Basis |
|------|-------|-------|
| Value | Low | 4 other customers, $220k ARR |
| Usability | Low | Standard export pattern |
| Feasibility | Unknown | Scope unclear—needs tech review |
| Viability | Low | No pricing/compliance flags |

**Evidence:**
- 4 mentions from 3 customers (2 enterprise, 1 mid-market)
- ARR represented: $220,000
- Key quotes: [link1], [link2]

**Existing opportunities:**
- Related: "Reporting flexibility" (ID: 123, status: in-discovery)
- No prior rejections found

**Recommended outcome:** LIGHT
**Reasoning:** Clear problem, strong evidence, low risk except feasibility unknown.
Recommend brief tech review then prioritize.
```

**Example 2: Disqualified (previously rejected)**

```markdown
## Triage Assessment

**Problem clarity:** Clear
> "We need to export data in custom XML format for our legacy system"

**Existing opportunities:**
- **Prior rejection found:** Opportunity #89 "Custom export formats" (rejected 2026-08-15)
- PM reasoning: "Not pursuing custom formats—enterprise customers use our API
  integration instead. Validated with 3 enterprise accounts Q2."

**Current signal comparison:**
- Same request type (custom export format)
- Same segment (enterprise)
- No new evidence that API integration is insufficient

**Recommended outcome:** DISQUALIFY
**Reasoning:** Exact same ask as previously rejected #89. PM reasoning ("use API
integration") still valid—no evidence of change. Archive and link to prior decision.
**Override if:** Customer can show API integration doesn't meet their need, or
strategic direction on custom formats has changed.
```

**Example 3: Disqualified (vague/unactionable)**

```markdown
## Triage Assessment

**Problem clarity:** Vague
> "The dashboard is slow"

**Attempted clarification:**
- No specific workflow identified
- No customer context (who, what tier, what they're trying to do)
- Evidence search found 2 mentions of "slow" but different contexts
  (one about initial load, one about export generation)

**Recommended outcome:** DISQUALIFY
**Reasoning:** Too vague to act on. "Slow" could mean many things. No clear
problem to solve. If specific performance issues exist, they should come
through with reproduction steps and context.
**Override if:** Source can provide specifics (which page, which action,
what they were trying to accomplish).
```

### 5.3 Triage Outcomes

| Outcome | Criteria | Next step |
|---------|----------|-----------|
| **Disqualify** | Previously rejected (reasoning still valid) OR vague/unactionable OR out of scope per DISCOVERY.md | → Archive with reason (can reopen if context changes) |
| **Merge** | Matches existing active opportunity | → Add as evidence to existing opportunity |
| **Light** | Problem clear + evidence exists + risks low or known + solution obvious | → Curated (ready for prioritization) |
| **Standard** | Problem clear but evidence thin OR 1-2 risks medium/unknown | → Plan (targeted validation) |
| **Full** | Problem unclear OR multiple high/unknown risks OR strategic bet | → Plan (full discovery) |

**Processing order:** The agent checks for Disqualify and Merge first. Only if neither applies does it assess the tier (Light / Standard / Full).

**Human decision:** The agent recommends an outcome. The PM confirms or overrides.

---

## 6. Stage 3a: Curated (Light Tier)

**What:** Opportunities that are ready for prioritization without further discovery work.

**Criteria:**
- Problem is clear and well-articulated
- Evidence exists (at least 1 customer + context OR multiple signals)
- Risks are low or known (no major unknowns)
- Solution is obvious or constrained

**What happens:**
1. PM reviews triage output
2. PM completes the `intent.md` if needed (constraints, non-goals, scope fence)
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
- PM: <name, date>
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

The PM selects which activities are needed based on what's uncertain:

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
- PM writes core of `intent.md` personally

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

**What:** PM / PO decides which opportunities with completed `intent.md` go to implementation.

**Who:** Product Manager / Product Owner, with input from the product trio.

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

**Gate:** PM accepts the `intent.md`. High-risk intents also need:
- Tech lead sign-off (feasibility risk)
- Policy owner sign-off (viability risk flagged)

---

## 9. Artifacts Summary

| Artifact | Created at | Contains | Lives in |
|----------|------------|----------|----------|
| **Opportunity** | Capture | Problem, source, context, triage assessment | ProductSense |
| **intent.md (draft)** | Capture | Problem, source, proposed outcome | Attached to opportunity |
| **Triage assessment** | Triage | Clarity, risks, evidence, tier recommendation | Attached to opportunity |
| **intent.md (complete)** | Curated or Plan exit | Full intent ready for SDLC | Attached to opportunity, exported to repo |
| **Test cards** | Plan (if needed) | Hypothesis, method, result | Linked from opportunity |
| **Interview notes** | Plan (if needed) | Customer conversations | Linked from opportunity |

---

## 10. Human / Agent Responsibility Split

| Stage | Agent does | Human owns |
|-------|------------|------------|
| **Capture** | May draft from signal, structures the opportunity | Decides to create, owns the opportunity |
| **Triage** | Assesses clarity, scans risks, searches evidence, finds matches, checks for prior rejections, recommends disqualification when warranted | Reviews assessment, confirms or overrides outcome (including disqualification) |
| **Plan** | Prepares interview briefs, drafts guides, finds evidence, scaffolds prototypes | Conducts interviews, makes judgment calls, writes core intent |
| **Prioritize** | May score/rank based on criteria | Makes the prioritization decision |

**Gates (human-only):**
- Triage outcome decision (disqualify / merge / tier)
- Move to Curated (problem validated)
- Accept `intent.md` (enters SDLC)
- Reopen a disqualified opportunity

---

## 11. Roles

| Role | Responsibilities |
|------|------------------|
| **Product Manager** | Owns opportunities, reviews triage, confirms disqualifications or overrides, decides tier, conducts customer conversations, writes intent, prioritizes |
| **Product Designer** | Owns usability risk, observes users, creates prototypes |
| **Tech Lead** | Owns feasibility risk, reviews technical spikes, co-signs high-risk intents |
| **Discovery Platform Owner** | Maintains MCP connections, triage agent, skills, evals |
| **Agents** | Ingest, enrich, assess, search, recommend disqualification, draft. **Never make final decisions on disqualification, tier, or intent acceptance.** |

---

## 12. Skills and Agents

| Skill / Agent | Stage | Purpose |
|---------------|-------|---------|
| `signal-triage` | Triage | Assess problem clarity, scan risks, search evidence, find existing opportunities, check for prior rejections, recommend disqualification |
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
| **Everything goes Light** | Skips validation on uncertain bets | Triage must flag unknowns; PM accountable for tier choice |
| **Over-disqualifying** | Good ideas get killed by lazy pattern matching | Disqualify needs clear reasoning; PM reviews all disqualifications |
| **Under-disqualifying** | Same rejected ideas keep coming back, wasting triage effort | Agent checks prior rejections; reasoning must be recorded |
| **Disqualifying without recording why** | Can't learn, can't reopen appropriately | Disqualification requires linked reasoning |
| **Synthetic users as validation** | No real-world friction, sycophancy | Never valid as evidence |
| **Agent decides tier** | Removes human judgment | Agent recommends, human decides |
| **Prototypes become production** | No security, observability | Sandbox + time-box |
| **Skip triage for "urgent" requests** | Creates invisible risk | All opportunities go through triage |
| **PM doesn't talk to customers** | Loses intuition | PM attends interviews in Plan stage |

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

- What are the specific thresholds for tier recommendation? (e.g., "if ≥2 risks are unknown → Standard")
- Should triage run automatically or on-demand?
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
