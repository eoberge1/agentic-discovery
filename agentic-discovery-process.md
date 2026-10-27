# Agentic Discovery & Post-Launch Learning: Process Definition

**Status:** Draft v0.2
**Owner:** Product Management
**Last updated:** 2026-10-23
**Related:** [The AI-Native SDLC Playbook](https://claude.com/resources/articles/the-ai-native-sdlc-playbook) · `context/Gemini-Agentic-AI-deep-research.md` · `context/reddit-Agentic-AI-research.md`

**Changes in v0.2:** Added **Post-Launch Learning** (§6) as a separate agentic process that runs after the AI-native SDLC and feeds back into discovery. Updated the loop diagram, integration contract, roles, gates, skills, anti-patterns, metrics, templates, and rollout to match.

---

## 1. Purpose

This document defines two connected agentic processes that surround the **AI-native SDLC** (Plan → Design → Build → Test → Deploy → Maintain):

1. **Agentic Discovery** comes *before* the SDLC. It decides what is worth building and produces an evidence-backed `intent.md`.
2. **Post-Launch Learning** comes *after* the SDLC. It tracks whether a launched feature actually delivered the outcome its `intent.md` promised, and feeds what it learns back into discovery.

In the SDLC playbook, discovery has no stage of its own. It sits inside **Plan**, where an originator brainstorms with Claude until the idea is concrete enough to write `intent.md`. That works for small, well-understood changes. For anything that carries real **value, usability, feasibility, or viability risk**, it isn't enough.

The playbook's **Maintain** stage watches *system health*: errors, latency, security, and incidents. It does not ask whether the feature worked for customers and the business. Without that question the loop breaks: teams ship, move on, and never find out whether their discovery bets paid off.

Our research context makes one point repeatedly. AI has made code, tickets, and documents cheap, so the bottleneck has moved to **deciding what is worth building**, **making sure the spec is right**, and **learning from what shipped**. Coding agents treat the written spec as absolute truth: a vague `intent.md` turns into a confident, locally rational, and wrong implementation, produced at machine speed. Shipping faster without measuring outcomes gives you a faster feature factory.

Both processes follow the same design rules as the SDLC:

- Every stage commits a reviewable artifact.
- Handoffs happen through artifacts, not meetings.
- Detection is deterministic, and models are called only after a threshold is crossed.
- Humans stay accountable for judgement.

> **Guiding principle:** *Agents remove administrative overhead so that humans can spend more time with customers, not less.*
> The loop keeps running. Human judgement stays above it.

---

## 2. Scope

| In scope | Out of scope |
| :---- | :---- |
| Continuous signal ingestion (support, sales calls, telemetry, churn, competitors) | Delivery mechanics between `intent.md` acceptance and release (covered by the SDLC playbook) |
| Opportunity structuring (Opportunity Solution Tree) | System health, incident response, and security monitoring (SDLC Maintain) |
| Evidence verification and human research | Annual strategy setting and OKR definition (an input to discovery, not part of it) |
| Assumption testing and throwaway prototypes | Production-grade prototyping (that is delivery) |
| Authoring and accepting `intent.md`, including its measurement plan | |
| **Post-launch outcome tracking:** adoption, outcome and guardrail metrics, feature-specific feedback, outcome reviews, and routing learnings back to discovery | |

**When to use the full discovery loop and when to use the SDLC fast path:**

| Situation | Path |
| :---- | :---- |
| Bug, incident follow-up, small UX polish, or a clear and low-risk request | **Fast path:** originator writes `intent.md` directly in SDLC Plan |
| New capability, new segment, pricing or packaging, or anything with unclear value | **Full discovery loop** (§5) |
| Maintain-loop finding that suggests a *systemic user need* rather than an incident | **Full discovery loop**, entering at D1 |
| Post-launch outcome review with a verdict of *miss* or *pivot* | **Full discovery loop**, entering at D2 (§6, L5) |

The product owner decides which path applies. When in doubt, use discovery: an extra week of discovery costs far less than a quarter of well-executed building in the wrong direction.

**Which launches get Post-Launch Learning:** every intent that went through full discovery, plus any fast-path intent that declares a customer or business outcome metric. Pure bug fixes and incident remediations are tracked by Maintain only.

---

## 3. How the Three Loops Fit Together

```
               ┌──────────────────── AGENTIC DISCOVERY ────────────────────┐
 Strategy ────►│ D0 Frame ► D1 Sense ► D2 Map ► D3 Verify ► D4 Engage ►     │
 (outcomes)    │                                  D5 Test ► D6 Commit ─────┼──► intent.md
               └──────▲─────────────▲──────────────────────────────────────┘  (+ measurement plan)
                      │             │                                              │
         new problems │             │ outcome evidence                             ▼
         & signals    │             │ (miss / pivot / partial)   ┌──────────── AI-NATIVE SDLC ─────────────┐
                      │             │                            │ Plan ► Design ► Build ► Test ► Deploy ──┼──┐
                      │             │                            │                         Maintain ◄──────┤  │
                      │             │                            └────────────┬────────────────────────────┘  │
                      │             │                                         │ health incidents              │ release
                      │             │                                         │ (operational)                 │ event
                      │             │       ┌─────────────── POST-LAUNCH LEARNING ──────────────┐            │
                      │             └───────┤ L5 Route ◄ L4 Review ◄ L3 Diagnose ◄ L2 Observe ◄  │◄───────────┘
                      └─────────────────────┤                                    L1 Launch-ready │
                                            │ iterate ──► SDLC fast path (new intent.md)         │
                                            │ graduate ─► Maintain baseline                      │
                                            └────────────────────────────────────────────────────┘
```

**The three loops have different jobs:**

| | Agentic Discovery | AI-native SDLC (incl. Maintain) | Post-Launch Learning |
| :---- | :---- | :---- | :---- |
| **Question** | What is worth building, and why? | How do we build, ship, and run it safely? | Did it deliver the promised outcome, and what did we learn? |
| **Primary owner** | PM (product trio) | Engineering (tech lead, service owner) | PM (product trio), with data/analytics |
| **Time horizon** | Continuous | Per change, then always-on health | Time-boxed per launch (e.g. 90 days), then graduates |
| **Watches** | Market and customer signals | Errors, latency, security, incidents | Adoption, outcome and guardrail metrics, feature-specific feedback |
| **Entry artifact** | Signals, `DISCOVERY.md` | `intent.md` | `launch.md` (measurement plan) + release event |
| **Exit artifact** | Accepted `intent.md` | Merged and deployed change | `outcome-review.md` + routing decision |

**Integration contract:**

1. **Discovery → SDLC.** The *only* output of discovery that enters the SDLC is an **accepted `intent.md`**, together with links to its evidence and its **measurement plan**. Accepting it triggers the SDLC Plan/Design pass, which writes `spec.md`. Prototypes, synthesis decks, and opportunity maps never enter the build pipeline directly.
2. **SDLC → Post-Launch Learning.** Before release, the SDLC Design stage turns the measurement plan into a committed `launch.md`, and Build adds the instrumentation. The **Deploy gate checks that `launch.md` exists and that its instrumentation is live**, for example as a hook on release authorization. The production release event starts the Post-Launch Learning loop for that launch.
3. **SDLC Maintain ↔ Post-Launch Learning.** The two loops share data but route findings differently. Health findings (errors, latency, incidents) belong to **Maintain** and lead to an SDLC fix. Product findings (low adoption, outcome miss, confusion, unmet need) belong to **Post-Launch Learning**. When one loop spots the other's kind of issue, it hands it over (§6, L3).
4. **Post-Launch Learning → Discovery.** Outcome reviews feed back as **outcome evidence**, which carries more weight than raw signals because it is tied to a hypothesis we tested in production:
   - *Miss / pivot*: re-enter discovery at **D2** and update the originating opportunity with actuals.
   - *Partial*: re-enter at **D5** with a new assumption test.
   - *New problems surfaced*: log as signals in **D1**.
   - *Hit*: update the opportunity tree to "validated", and record what worked in `DISCOVERY.md`.
5. **Post-Launch Learning → SDLC fast path.** Small, clear improvements (copy, defaults, minor UX friction) become new `intent.md` files directly, without going through full discovery.

---

## 4. Roles

Both processes keep the **product trio** (PM, designer, tech lead) and map it onto the SDLC roles.

| Role | SDLC equivalent | Responsibilities |
| :---- | :---- | :---- |
| **Product Manager** | Product owner / originator | Owns the opportunity tree, the discovery backlog, and each launch's outcome. Runs the D3, D6, and L4 gates. **Attends customer conversations personally**, including post-launch ones. Accountable for value and viability risk. |
| **Product Designer** | (contributes to Design) | Owns usability risk, research design, and prototype fidelity choices. Observes real users before *and* after launch. |
| **Tech Lead** | Tech lead | Owns feasibility risk, reviews technical spikes, and co-signs high-risk intents. Confirms instrumentation is in place before release. |
| **Data / Analytics Owner** | (new) | Co-owns `launch.md`: metric definitions, baselines, experiment and holdout design, and statistical judgement in outcome reviews. Approves `outcomes.yaml` thresholds. |
| **Customer Success / Support lead** | (new) | Makes sure feature-specific feedback is tagged at the source. Brings account context to outcome reviews. |
| **Service owner / on-call** | Service owner | Handles health issues that Post-Launch Learning hands over to Maintain. |
| **Discovery Platform Owner** | Platform engineer | Maintains MCP connections, skills, `signals.yaml` and `outcomes.yaml`, and evals for discovery and post-launch agents. |
| **Policy owners** (Legal, Security, Pricing, Brand) | Policy owners | Own the viability skills (compliance, pricing guardrails) that agents check against. |
| **Agents** | Claude, subagents, headless triggers | Ingest, cluster, enrich, draft, interrogate, scaffold, monitor, and analyse. **Never approve and never decide a verdict.** |

**Separation of duties (borrowed from the SDLC):**

- The agent that synthesises evidence cannot be the agent that verifies it.
- The agent that drafts an outcome review cannot be the agent that challenges its attribution.
- No agent can accept an `intent.md` or give an outcome verdict.

---

## 5. Process A: The Agentic Discovery Loop

Each stage lists: **what happens · who leads · artifact committed · gate.**

### D0. Frame: set the outcome and the boundaries
- **What:** Restate the business outcome (OKR or north-star metric) that discovery serves, the target segments, the strategic boundaries, and what we will *not* do.
- **Lead:** Human (PM with leadership). An agent can challenge the framing ("Which segment does this outcome actually depend on?"), but it does not set strategy.
- **Artifact:** `DISCOVERY.md`, the discovery equivalent of `CLAUDE.md`. It holds outcomes, segment definitions, ARR or tier weighting rules, strategic non-goals, known past mistakes, and **lessons from post-launch reviews** (what kinds of bets have hit or missed). Every discovery agent reads it at the start of each session.
- **Gate:** PM and Head of Product approve changes to `DISCOVERY.md` (CODEOWNERS-style).

### D1. Sense: ambient signal ingestion
- **What:** Agents continuously ingest support tickets, sales and CS call transcripts, product telemetry, churn and downgrade events, app reviews, and competitor surfaces (changelogs, pricing pages, API docs, review sites). Maintain-loop findings and **new problems surfaced by Post-Launch Learning** are routed in here.
- **Lead:** Agents (headless, scheduled). Humans look only at what crosses a threshold.
- **Key rule: detection stays deterministic.** `signals.yaml` defines bands in the same way the SDLC's `bands.yaml` does:
  - *Band 1:* log only (e.g. a new theme with fewer than N mentions).
  - *Band 2:* the agent enriches and drafts a signal brief, read-only (e.g. the theme appears in at least one top-tier account, or competitor parity changes).
  - *Band 3:* notify the PM and propose a new or updated opportunity (e.g. the theme is linked to churned or at-risk ARR above a threshold).
- **Mandatory enrichment:** each signal must carry **account, plan tier, ARR, tenure, churn status, and source link**. A signal without metadata is noise (see the "invisible variables" trap in §9).
- **Artifact:** Signal records in the discovery system of record. Competitor deltas are compiled into a weekly digest.
- **Tooling (via MCP):** `searchInsights`, `findEvidence`, `listChurnSignals`, `getCustomerDetails`, `identifyCompany`, `getCompetitorDetails` (ProductSense), plus analytics, CRM, and the issue tracker.

### D2. Map: structure opportunities, not feature requests
- **What:** Agents cluster signals into **customer problems** and map them onto the **Opportunity Solution Tree** (Outcome → Opportunity → Solution → Assumption test). Feature requests are reframed as the underlying need. For example, repeated "export" requests become the opportunity "cross-department stakeholder reporting".
- **Causal extraction over keyword clouds:** the agent has to propose a *mechanism*. An example: "sign-in failures come from corporate security gateways stripping magic-link tokens", not "users are unhappy with login".
- **Outcome evidence:** when a launch misses or needs a pivot, its `outcome-review.md` is attached to the originating opportunity. The solution node is marked with its actual result (hit, partial, miss), so the tree records which bets worked and not only which ones were made.
- **Lead:** The agent drafts and the PM curates. A human decides tree structure and placement.
- **Artifact:** `opportunity.md`, one per opportunity (template in §11), linked to its evidence.
- **Tooling:** `listOpportunities`, `getOpportunity`, `createOpportunity`, `updateOpportunity`.

### D3. Verify: the four validation filters (gate)
No synthesised theme can be prioritised until it passes all four filters. A **separate verification agent** runs them and the PM signs off. The same filters are reused on post-launch feedback (§6, L3).

| # | Filter | Check | Fails when | Then |
| :- | :---- | :---- | :---- | :---- |
| 1 | **Verbatim quote audit** | At least 3 distinct primary-source quotes, each with a link | Only paraphrases or generalisations | Discard as ungrounded |
| 2 | **Revenue and cohort segmentation** | Weight by ARR, tier, and tenure, not by mention count | Volume comes from trial, free, or non-core users | Re-prioritise by economic weight |
| 3 | **Negative space and churn** | Which cohorts are *silent*? Have churned accounts been checked? | Only vocal power users are represented | Commission outbound research with churned and disengaged accounts (D4) |
| 4 | **Temporal window shift** | Re-run the synthesis over 14, 60, and 180 days | The theme exists only in one acute window | Reclassify as an operational incident → SDLC fast path |

Also required: an **outlier pass**. LLM summaries drift towards the average, so the agent must list the most surprising or contradictory individual comments separately. Breakthrough insights often sit there.

- **Gate:** The PM marks the opportunity `verified`, `needs-research`, or `rejected`, and records the reason in `opportunity.md`.

### D4. Engage: human research, made cheaper by agents
- **What:** Real conversations with real customers. Agents handle preparation and logistics. Humans do the listening.
  - **Pre-interview brief (agent):** a Customer-360 dossier covering usage, tickets, contract and renewal date, and recent communications, so the conversation can focus on goals and friction rather than account details.
  - **Discussion guide (agent drafts, human edits):** focused on concrete *past behaviour*, not future intentions.
  - **Interview (human):** PM and designer attend. **The PM attends and takes their own notes.** AI transcripts supplement those notes and do not replace them.
  - **Async AI-moderated interviews (agent):** in-product conversations triggered by events (upgrade, abandonment, cancellation) to widen coverage. They follow the same past-behaviour rules, and their output counts as *signals* (D1 and D2), not as validation.
  - **Post-call debrief (human, agent-assisted):** a voice debrief straight after the call is turned into a structured note and attached to the opportunity.
- **Cadence:** at least weekly customer touchpoints per trio (continuous discovery habit).
- **Artifact:** Interview notes and transcripts linked as evidence on `opportunity.md`.
- **Gate:** None. This stage feeds evidence back into D2 and D3.

### D5. Test: assumption tests at the right fidelity
- **What:** For the chosen solution candidates, list the riskiest assumptions per risk type and run the cheapest test that would change our mind. Partial post-launch results come back here as new assumptions to test.
- **Fidelity must match the risk being tested:**

| Primary risk | Appropriate test | Avoid |
| :---- | :---- | :---- |
| **Value** (will they want it?) | Problem interviews, fake-door or painted-door tests, concierge tests, a rough clickable prototype | Polished, fully built prototypes. They answer the wrong question. |
| **Usability** (can they use it?) | Interactive prototype (Claude Artifacts, Claude Design, v0) tested with real users | Synthetic-user "usability" passes as evidence |
| **Feasibility** (can we build it?) | Technical spike: the agent explores the codebase in plan mode, checks API schema compatibility, and interrogates edge cases | PM-built "production-ish" code |
| **Viability** (does it work for the business?) | Agent checks against pricing, compliance, and unit-economics skills, then stakeholder review | Skipping Legal, Security, or Finance because "the AI checked it" |

- **Prototype rules:**
  1. Prototypes live in a **sandbox repo or workspace** and are labelled *disposable*.
  2. A hook or branch protection **blocks prototype code from merging into production repos**. Prototypes inform `spec.md`. They are never its implementation.
  3. Time-box: if a prototype has taken more than about two days, check whether you are doing delivery work under the name of discovery (the "fidelity trap").
- **Synthetic users:** allowed *only* for testing discussion guides, desk research, and edge-case brainstorming. **Never valid as evidence of value or usability.**
- **Artifact:** `test-card.md` for each test, with hypothesis, method, success threshold, result, and decision. The card is linked from `opportunity.md`.
- **Gate:** The trio reviews the results. Evidence strength is recorded on the opportunity.

### D6. Commit: author and accept `intent.md` (gate → SDLC)
- **What:** Turn the validated opportunity and solution into the SDLC's entry artifact.
  1. The PM writes the **core of `intent.md` personally**: problem, outcome, constraints, and non-goals. Keep it short, around 1 to 2 pages ("spec as fences, AI as draftsman").
  2. The PM defines the **measurement plan** with the data/analytics owner. It names one primary outcome metric with a baseline and target, the guardrail metrics, the adoption definition, the target segments, and the readout dates. **Thresholds are fixed now, before launch**, so the goalposts can't move later.
  3. An **interrogation agent** (a separate session) attacks the draft. It looks for unstated assumptions, edge cases, failure states, missing definitions, conflicts with `DISCOVERY.md` non-goals, lessons from past post-mortems and outcome reviews, and **metrics that can't be measured or attributed**. The PM resolves every point or marks it as an explicit open question.
  4. **Determinism check:** could a coding agent read this and build something locally rational but systemically wrong? (Example: "optimise resolution time" with no guardrail against unverified refunds.) If so, add the guardrail as a constraint *and* as a guardrail metric.
- **Artifact:** `intent.md` (template in §11) committed with an **Evidence** section that links the opportunity, the key quotes, the test cards, and the risk assessment, plus a **Measurement plan** section.
- **Gate:** The **product owner accepts `intent.md`**. Higher-risk intents also need tech lead sign-off, plus policy owners where viability flags were raised. Acceptance triggers the SDLC Plan/Design pass.

---

## 6. Process B: Post-Launch Learning (after the AI-native SDLC)

Post-Launch Learning is an **agentic, time-boxed loop for each launch**. It starts when the SDLC Deploy stage releases a feature to production and ends when the launch **graduates** to the Maintain baseline or is **routed** back into discovery or the SDLC.

Each stage lists: **what happens · who leads · artifact committed · gate.**

### L1. Launch-ready: lock the measurement plan (gate before Deploy)
- **What:** During SDLC Design and Build, the measurement plan in `intent.md` becomes a committed **`launch.md`**. It contains:
  - **Primary outcome metric** (from `intent.md`), with baseline, target, and minimum detectable effect.
  - **Adoption and engagement metrics** (discovery → first use → repeat use), with clear event definitions.
  - **Guardrail metrics** that must not get worse (e.g. churn, support volume, latency, refund rate, key-flow conversion).
  - **Target and comparison cohorts:** tier, segment, holdout, or experiment arms, plus rollout stages (internal → beta → % rollout → GA).
  - **Feedback hooks:** support tags, in-product micro-survey triggers, async interview triggers for adopters, non-adopters, and abandoners, and a sales/CS mention tag.
  - **Readout schedule:** for example, an early health check at 7 days, adoption at 30 days, and the outcome at 90 days.
  - **Kill / rollback criteria:** product-level criteria, which are separate from the SDLC's technical rollback.
- **Thresholds** are written to `outcomes.yaml` (§6.1) so that detection stays deterministic.
- **Lead:** The PM and the data/analytics owner. An agent (`launch-readiness`) checks that every metric has an instrumented event, that baselines exist, and that the readout dates are scheduled.
- **Gate (G6):** `launch.md` is approved by the PM and the data/analytics owner. **The SDLC Deploy release-authorization hook refuses to release a tracked intent unless `launch.md` is approved and its events are verified as firing in staging.**

### L2. Observe: agentic monitoring of metrics and feedback
- **What:** Once the release event fires, headless agents monitor the launch continuously:
  - **Metrics:** adoption funnel, primary outcome, and guardrails per cohort, compared with the baseline and the holdout.
  - **Feature-specific feedback:** support tickets, app reviews, community threads, sales and CS call mentions, and NPS/CSAT verbatims. Each item is tagged to the launch and **enriched with account, tier, ARR, and churn status**, the same as in D1.
  - **Targeted qualitative input:** event-triggered micro-surveys and async AI-moderated interviews with **adopters, non-adopters, and abandoners**. Non-adopters matter as much as adopters.
  - **Account movements:** expansions, downgrades, and churn among accounts exposed to the feature.
- **Detection is deterministic.** `outcomes.yaml` bands decide what happens:
  - *Band 1:* log and include in the weekly launch digest.
  - *Band 2:* an agent writes a read-only diagnostic brief (e.g. adoption is below the trajectory needed to reach target, or a new feedback theme appears in a top-tier account).
  - *Band 3:* notify the PM immediately (e.g. a guardrail is breached, or the kill criteria are met). The agent may **propose** a rollback, a feature-flag change, or a fast-path intent. A human decides.
- **Lead:** Agents (headless, scheduled). Humans get the weekly digest and threshold alerts.
- **Artifact:** A weekly launch digest and signal records tagged to the launch.
- **Tooling (via MCP):** product analytics and experimentation platform, `searchInsights` / `findEvidence` / `listChurnSignals` / `getCustomerDetails` (ProductSense), CRM, support desk, and the feature-flag system.

### L3. Diagnose: attribute, classify, and verify
- **What:** When a band is breached, or at each readout, an analysis agent explains what is happening:
  - **Attribution:** compare with the holdout or control. Check for novelty effects, seasonality, concurrent launches, marketing campaigns, and pricing changes.
  - **Segmentation:** break results down by tier, ARR, tenure, and rollout stage. A flat average can hide a strong win in the target segment and a loss elsewhere.
  - **Funnel diagnosis:** where adoption drops off (awareness, first use, or repeat use), and the likely cause.
  - **Classify the finding** and route it to the right owner:

| Finding type | Example | Route |
| :---- | :---- | :---- |
| **Operational / health** | Errors, latency, failing integration | Hand over to **SDLC Maintain** (service owner) |
| **Small usability friction** | Confusing label, bad default, missing empty state | **SDLC fast path** (new `intent.md`) |
| **Value / outcome problem** | Low repeat use, the outcome metric does not move, the wrong segment is adopting | **Outcome review (L4)**, then likely back to **discovery D2** |
| **New, unrelated problem** | Users reveal a different unmet need | **Discovery D1** signal |

- **Verify qualitative themes** with the **four validation filters** from D3 (verbatim quotes, segment weighting, negative space, temporal window). Launch feedback is especially prone to two distortions: a novelty spike, and vocal early adopters who don't represent the target segment.
- **Lead:** The agent drafts the analysis. A **separate challenger agent** tests the attribution ("What else could explain this?"). The data/analytics owner reviews any statistical claims.
- **Artifact:** A diagnostic brief attached to the launch.

### L4. Review: outcome readouts and verdict (gate)
- **What:** At each scheduled readout (e.g. 7, 30, and 90 days), an agent drafts **`outcome-review.md`** (template in §11). It sets the metrics against the targets fixed in `launch.md` and includes verified feedback themes with verbatim quotes, outliers, segment results, and the challenger agent's alternative explanations.
- **Human work:** before the final readout, the PM and designer **talk to at least a few adopters and non-adopters themselves**. The summary does not replace that contact.
- **Verdict (human only):**
  - **Hit:** the primary outcome reached its target and the guardrails held.
  - **Partial:** a meaningful movement, a win in some segments only, or adoption without the outcome yet.
  - **Miss:** no meaningful movement, or a guardrail was breached.
  - **Inconclusive:** too little data or a confounded result. Extend the window once, with an explicit reason.
- **Gate (G7):** The product trio and the data/analytics owner agree the verdict. The PM signs `outcome-review.md`.

### L5. Route: feed back and close
- **What:** Each verdict leads to a recorded routing decision:

| Verdict | Decision | Goes to |
| :---- | :---- | :---- |
| **Hit** | **Graduate:** stop launch-specific monitoring and move the key metrics to the Maintain baseline. Mark the opportunity solution node as *validated*. | SDLC Maintain + D2 tree update + `DISCOVERY.md` lessons |
| **Partial** | **Iterate:** frame what is missing as a new assumption | Discovery **D5** (new test card) or SDLC fast path for small fixes |
| **Miss** | **Pivot or kill:** attach the outcome evidence to the opportunity and re-open it | Discovery **D2**. If the decision is kill, a sunset `intent.md` enters the SDLC fast path. |
| **Any verdict** | Log new unrelated problems | Discovery **D1** signals |

- **Learning capture:**
  - Add a short "what we believed vs. what happened" entry to the lessons section of `DISCOVERY.md`.
  - Every **miss that discovery should have caught** becomes a **permanent eval** for the discovery agents (e.g. "The mapper ranked this theme highly. The outcome showed that the segment didn't care.").
  - Update the originating `opportunity.md` decision log.
- **Gate (G8):** The PM approves the routing decision. Kill and sunset decisions also need Head of Product sign-off.
- **Artifact:** Final `outcome-review.md` with the routing decision, linked from `intent.md` and `opportunity.md`, which closes the artifact chain.

### 6.1 `outcomes.yaml` (deterministic launch thresholds)

```yaml
launch: bulk-export-v1
intent: intents/2026-11-bulk-export.md
window_days: 90
cohorts: [enterprise, mid-market]
holdout: 10%

primary_outcome:
  metric: weekly_stakeholder_reports_shared
  baseline: 1.2
  target: 2.0
  readouts: [30, 90]

adoption:
  first_use_rate:   { band2_below: 0.15, band3_below: 0.05, from_day: 14 }
  repeat_use_rate:  { band2_below: 0.30, from_day: 30 }

guardrails:
  support_tickets_per_1k_accounts: { band2_above_pct: 20, band3_above_pct: 50 }
  enterprise_churn_rate:           { band3_above_pct: 10 }
  export_p95_latency_ms:           { route: maintain }   # health → SDLC Maintain

feedback:
  new_theme_in_top_tier_account: band2
  negative_theme_arr_at_risk_usd: { band3_above: 250000 }

kill_criteria:
  - "first_use_rate < 0.05 at day 30 in target cohorts"
```

---

## 7. Human / Agent Responsibility Split

| Risk | Agents do | Humans own |
| :---- | :---- | :---- |
| **Value** | Ingest signals, monitor churn triggers, structure the opportunity tree, transcribe and cluster, prepare interview briefs, **track adoption and outcome metrics after launch** | Direct interviews, willingness-to-pay, unarticulated needs, *the decision of what not to build*, **the outcome verdict** |
| **Usability** | Heuristic review, scaffold interactive prototypes, **diagnose the post-launch funnel and cluster friction feedback** | Watching real users struggle before and after launch, real environments, nuance |
| **Feasibility** | Interrogate edge cases, check schema and API compatibility, scan the codebase in plan mode, **check that instrumentation is in place** | Architecture, scalability, trade-off decisions |
| **Viability** | Compliance, pricing, and margin checks against policy skills, **monitor guardrail metrics and account movements** | Stakeholder alignment, go-to-market, enterprise risk, legal arbitration, **kill and sunset decisions** |

**Gates (human-in-the-loop checkpoints):**

| Gate | Process | Decision | Approver |
| :---- | :---- | :---- | :---- |
| G0 | Discovery | `DISCOVERY.md` changes (outcomes, segments, weights, lessons) | PM + Head of Product |
| G1 | Discovery | Band-3 signal → new or updated opportunity | PM |
| G2 | Discovery | Opportunity passes the four filters (`verified`) | PM |
| G3 | Discovery | Assumption test results accepted | Product trio |
| G4 | Discovery → SDLC | `intent.md` (incl. measurement plan) accepted → SDLC | Product owner (+ tech lead / policy owners when high-risk) |
| G5 | Both | Skill or agent changes, gated on evals | Discovery platform owner |
| G6 | SDLC → Post-launch | `launch.md` approved and instrumentation verified. Deploy hook enforces it. | PM + Data/analytics owner |
| G7 | Post-launch | Outcome verdict (hit / partial / miss / inconclusive) | Product trio + Data/analytics owner |
| G8 | Post-launch → Discovery | Routing decision (graduate / iterate / pivot / kill) | PM (+ Head of Product for kill) |

---

## 8. Skills and Agents

Repeatable practices are encoded as **skills** (`SKILL.md`). They are versioned, triggered by their description, and owned by a named person, in the same way as SDLC skills.

| Skill / agent | Stage | Purpose | Owner |
| :---- | :---- | :---- | :---- |
| `signal-ingest` | D1 | Pull, enrich (tier/ARR/churn), and dedupe signals; apply `signals.yaml` bands | Platform owner |
| `competitor-watch` | D1 | Weekly delta digest of competitor changelogs, pricing, docs, and reviews against our initiatives | PM |
| `opportunity` | D2–D6 | Manage opportunities in ProductSense (MCP). Every opportunity carries one or more `intent.md` sections, built to Anthropic's "Capture as intent.md" guidance. Lives in `.claude/skills/opportunity/` | PM |
| `opportunity-mapper` | D2 | Cluster signals into problems, reframe feature requests, propose causal mechanisms, place on the tree | PM |
| `evidence-verifier` | D3, L3 | Run the four filters and the outlier pass. **Must be a separate session from the mapper or drafter.** | PM |
| `customer-360` | D4, L4 | Pre-interview dossier | PM |
| `interview-guide` | D4, L4 | Draft a past-behaviour discussion guide and check it for leading questions | Designer |
| `assumption-mapper` | D5 | List riskiest assumptions per risk type and propose the cheapest test | Trio |
| `intent-interrogator` | D6 | Adversarial review of `intent.md` against edge cases, non-goals, post-mortems, and past outcome reviews; checks that metrics can be measured | PM + Tech lead |
| `viability-check` | D5/D6 | Check pricing, compliance, and unit economics against policy-owner skills | Policy owners |
| `launch-readiness` | L1 | Check that `launch.md` metrics map to instrumented events, that baselines exist, and that readouts are scheduled. Generate `outcomes.yaml`. | Data/analytics owner |
| `outcome-monitor` | L2 | Headless monitoring of metrics per cohort against `outcomes.yaml`; weekly launch digest | Platform owner |
| `launch-feedback-tagger` | L2 | Tag and enrich feature-specific feedback from support, reviews, sales and CS calls, and surveys | CS/Support lead |
| `impact-analyst` | L3 | Attribution against the holdout, segmentation, funnel diagnosis, and finding classification and routing | Data/analytics owner |
| `attribution-challenger` | L3 | Adversarial "What else could explain this?" pass on the analyst's conclusions. **Must be a separate session.** | Data/analytics owner |
| `outcome-review-drafter` | L4 | Draft `outcome-review.md` against the targets fixed in `launch.md` | PM |
| `learning-router` | L5 | Draft routing artifacts (opportunity update, test card, fast-path intent, `DISCOVERY.md` lesson, candidate eval) | PM |

**Evals.** These work like SDLC continuous evals and run whenever a skill, `DISCOVERY.md`, or `outcomes.yaml` changes:
- The verifier rejects themes that lack verbatim quotes.
- The mapper ranks a low-frequency, high-ARR theme above a high-frequency trial-user theme. (Golden case: 8 enterprise calls representing 45% of ARR vs 185 free-tier tickets.)
- The interrogator flags known edge cases on a reference upload spec (payload limits, partial batch failure, sanitisation, downtime states), and flags an outcome metric that has no instrumented event.
- The impact analyst doesn't claim a win from a novelty spike in week 1, and flags a confounding concurrent launch in a golden dataset.
- The classifier sends a latency regression to Maintain, not to discovery.
- Every discovery or launch mistake found later becomes a **permanent eval**, for example a shipped intent that missed its outcome where the evidence was weak or misread.

---

## 9. Anti-patterns (Do Not Do)

| Anti-pattern | Why it fails | Guardrail in this process |
| :---- | :---- | :---- |
| **Synthetic users as validation** | Sycophancy, little linguistic variety, none of the real-world friction. In tests, synthetic users "succeeded" at tasks that real users abandoned. | §5 D5: synthetic users are never evidence |
| **Ranking by mention volume** | Models weigh tokens, not revenue. Enterprise needs get buried. | D1 and L2 mandatory enrichment, D3 filter #2 |
| **Invisible variables** | Feedback without tier or ARR metadata leads to polished but strategically wrong recommendations | D1 and L2: no metadata means the signal is not promoted |
| **The churned-customer void** | Silence gets read as satisfaction | D3 filter #3, outbound research in D4, non-adopter interviews in L2 |
| **Flattening the outlier** | Summaries smooth away the odd comment that holds the insight | D3 and L3 outlier pass |
| **Outsourcing customer contact** | PMs lose intuition and the conviction to defend their decisions | D4 and L4: PM attends and takes own notes |
| **Fidelity trap / vibe-coding as discovery** | Delivery work done under the name of discovery, testing the wrong risk | D5 fidelity table and time-box |
| **Productionising prototypes** | No security, observability, or scalable schema, which means rewrites later | D5 merge-block hook |
| **One-shot AI PRDs** | Long documents that fall apart under scrutiny because the thinking was skipped | D6: PM writes the core personally, the agent only interrogates |
| **Ambiguous intent fed to coding agents** | Agents treat the spec as truth and build locally rational, systemically wrong things | D6 determinism check + guardrail metrics |
| **Bot-generated ticket floods** | Reviewers stop reading, and the bottleneck moves downstream | Only *accepted* intents enter the SDLC. Discovery and launch outputs never go straight into the tracker. |
| **AI for AI's sake** | Vanity AI features and internal agent sprawl | D0 non-goals; every intent must name its customer outcome |
| **Ship and forget** | Teams never learn whether their bets paid off, and the feature factory speeds up | L1 Deploy gate: no release without `launch.md`; L4 readouts are scheduled at launch |
| **Moving the goalposts** | Targets get redefined after launch so that every launch looks like a win | Targets are fixed in `intent.md` and `launch.md` before release; L4 compares only against them |
| **Vanity metrics** | Page views and clicks rise while the outcome stays flat | `launch.md` needs one primary *outcome* metric, separate from adoption |
| **Declaring victory on the novelty spike** | Week-1 curiosity gets mistaken for lasting value | Repeat-use metrics; the outcome verdict comes only at the final readout; challenger agent |
| **Listening only to adopters** | Survivorship bias. The people who didn't adopt hold the "why not". | L2 targeted interviews with non-adopters and abandoners |
| **Agent-declared success** | A polished AI summary becomes the verdict | G7: verdicts are human-only; attribution is challenged by a separate agent |
| **Mixing health and outcome** | Latency issues get debated as product failures, or value misses get "fixed" by engineering | L3 classification table routes findings to Maintain or discovery |

---

## 10. Metrics

Each metric is measured with leading and lagging indicators, in line with the SDLC playbook.

| Area | Leading indicators | Lagging indicators |
| :---- | :---- | :---- |
| **Discovery throughput** | Signal → verified-opportunity cycle time; customer touchpoints per trio per week | Time from opportunity creation to accepted intent |
| **Evidence quality** | % of verified opportunities with at least 3 verbatim quotes and segment weighting; % of decisions that include churned or silent cohorts | % of intents rejected or reworked in SDLC Design because of discovery gaps |
| **Launch readiness** | % of tracked releases with an approved `launch.md` and verified instrumentation at deploy | % of launches where the outcome couldn't be measured ("inconclusive" because of instrumentation gaps) |
| **Learning discipline** | % of readouts done on schedule; time from Band-3 launch alert to PM decision | % of launches with a signed outcome review and a routing decision within the window |
| **Decision quality** | % of intents with a declared success metric and test evidence | **Outcome hit rate:** % of shipped intents that hit their primary outcome. Trend over time = how well discovery predicts. |
| **Loop closure** | Time from a miss or partial verdict to the opportunity being re-opened in discovery | % of misses that produced a `DISCOVERY.md` lesson or eval |
| **Waste avoided** | # of opportunities killed at D3 or D5 | # of features sunset after a miss; build capacity spent on features later rolled back or unused |
| **Human connection** | PM attendance rate at customer conversations (pre- and post-launch) | Qualitative check: can the PM defend the decision without the summary? |

---

## 11. Templates

### 11.1 `opportunity.md`

```markdown
# Opportunity: <customer problem, phrased as a need>

- **ID / link:** <system-of-record link>
- **Outcome served:** <OKR / metric from DISCOVERY.md>
- **Status:** draft | needs-research | verified | in-test | committed | validated | re-opened | rejected
- **Owner:** <PM>

## Problem & causal mechanism
<What is happening, for whom, and why. Mechanism, not symptom.>

## Evidence
| Quote (verbatim) | Source link | Account | Tier | ARR | Churn status | Date |
|---|---|---|---|---|---|---|

- **Segment weighting:** <ARR represented, # accounts, tier mix>
- **Negative space:** <which cohorts are silent; outbound research done?>
- **Temporal check:** 14d: <n> · 60d: <n> · 180d: <n> → systemic | incident
- **Outliers worth noting:** <surprising or contradictory comments>

## Four-filter result
- [ ] Verbatim quotes (≥3)   - [ ] Segment-weighted   - [ ] Negative space checked   - [ ] Temporally stable
- **Verifier decision & reason:**

## Candidate solutions & tests
| Solution | Riskiest assumption | Risk type | Test card | Result |
|---|---|---|---|---|

## Launch outcomes
| Intent | Launch date | Verdict (hit/partial/miss) | Outcome review | Routing |
|---|---|---|---|---|

## Decision log
- <date> — <decision> — <who> — <why>
```

### 11.2 `test-card.md`

```markdown
# Test: <name>
- **Opportunity:** <link>
- **We believe:** <assumption>
- **Risk type:** value | usability | feasibility | viability
- **Method & fidelity:** <interview / fake door / prototype / spike>, time-box: <days>
- **Participants:** <real users, segment, n>  (synthetic users not permitted as evidence)
- **We are right if:** <measurable threshold, set before running>
- **Result:** <data + links>
- **Decision:** persevere | pivot | kill
```

### 11.3 `intent.md` (SDLC entry artifact, extended with discovery evidence and a measurement plan)

```markdown
# Intent: <title>

## Problem
<Who has the problem, in what situation, and why it matters. Written by the PM.>

## Proposed outcome
<The customer and business outcome.>

## Affected users & systems
<Segments/tiers; systems and integrations touched.>

## Constraints & guardrails
<Non-negotiables, including guardrails against locally rational, systemically wrong behaviour.>

## Non-goals
<What we are explicitly not doing.>

## Open questions
<Unresolved items from interrogation, with an owner for each.>

## Discovery evidence
- **Opportunity:** <link to opportunity.md>
- **Key quotes:** <2–3 verbatim, linked>
- **Segment / ARR represented:** <…>
- **Tests run:** <links to test cards + results>
- **Risk assessment:** Value: <L/M/H, evidence> · Usability: <…> · Feasibility: <…> · Viability: <…>
- **Interrogation pass:** <date, # issues raised / resolved>

## Measurement plan  (expanded into launch.md during SDLC Design)
- **Primary outcome metric:** <metric> · baseline <x> · target <y> · by day <n>
- **Adoption definition:** <first use / repeat use events>
- **Guardrail metrics:** <metrics that must not degrade>
- **Target cohorts / comparison:** <segments; holdout or experiment>
- **Readouts:** <e.g. 7 / 30 / 90 days>
- **Kill criteria:** <product-level conditions for rollback or sunset>

## Approval
- Product owner: <name, date>
- Tech lead (if high-risk): <name, date>
- Policy owners (if flagged): <name, date>
```

### 11.4 `launch.md` (Post-Launch Learning entry artifact)

```markdown
# Launch: <feature name>
- **Intent:** <link>   · **Opportunity:** <link>   · **Owner:** <PM>   · **Data owner:** <name>
- **Rollout plan:** internal <date> → beta <cohort, date> → <x>% <date> → GA <date>
- **Feature flag:** <name>

## Metrics
| Type | Metric | Event / query | Baseline | Target / threshold | Cohorts |
|---|---|---|---|---|---|
| Primary outcome | | | | | |
| Adoption | | | | | |
| Guardrail | | | | | |

- **Comparison method:** holdout <x%> | A/B | pre/post with controls <list>
- **Minimum detectable effect / sample needed:** <…>
- **Thresholds file:** outcomes.yaml <link>

## Feedback hooks
- Support tag: <tag>   · Sales/CS mention tag: <tag>
- Micro-survey trigger: <event, question>
- Async interviews: adopters <trigger> · non-adopters <trigger> · abandoners <trigger>

## Readouts
| Readout | Date | Question it answers |
|---|---|---|
| Early health | day 7 | Is it working and being found? |
| Adoption | day 30 | Are target users adopting and returning? |
| Outcome | day 90 | Did the primary outcome move? |

## Kill / rollback criteria
- <condition> → <action>

## Instrumentation verified
- [ ] All events firing in staging   (verified by: <name/agent>, date)
- **Approved (G6):** PM <name, date> · Data owner <name, date>
```

### 11.5 `outcome-review.md`

```markdown
# Outcome review: <feature> — readout <day n> (<interim | final>)
- **Launch:** <link to launch.md>   · **Intent:** <link>   · **Opportunity:** <link>

## Results vs. targets (fixed in launch.md)
| Metric | Baseline | Target | Actual | vs. holdout | Segment notes |
|---|---|---|---|---|---|

## Attribution & confounders
- Analyst conclusion: <…>
- Challenger's alternative explanations: <…>  → resolved / still open

## Customer evidence (verified with the four filters)
| Theme | Verbatim quotes (linked) | Segment / ARR | Adopter / non-adopter |
|---|---|---|---|
- **Outliers:** <…>
- **Human conversations held by trio:** <# adopters, # non-adopters, key takeaways>

## Health issues handed to Maintain
- <issue> → <ticket/intent link>

## Verdict (G7, human)
hit | partial | miss | inconclusive — <rationale>

## What we believed vs. what happened
<2–4 sentences — goes into DISCOVERY.md lessons>

## Routing decision (G8)
graduate | iterate | pivot | kill → <links to opportunity update / test card / fast-path intent / sunset intent / new eval>
- Signed: PM <name, date> · (Head of Product for kill) <name, date>
```

---

## 12. Rollout (suggested)

1. **Weeks 1–2, foundations:** Write `DISCOVERY.md`. Connect MCP sources (ProductSense, CRM, analytics, tracker). Make enrichment metadata mandatory.
2. **Weeks 3–4, low-risk, high-yield first:** Turn on `competitor-watch`, `customer-360`, and `signal-ingest` in Band-1 and Band-2 mode only.
3. **Weeks 5–6, verification:** Introduce `evidence-verifier` and the four-filter gate. Run it alongside the current process on 2 or 3 live opportunities.
4. **Weeks 7–8, intent handoff:** Adopt the extended `intent.md` (with measurement plan) and `intent-interrogator`. Connect accepted intents to the SDLC Plan/Design trigger.
5. **Weeks 9–10, post-launch pilot:** Pick 2 or 3 upcoming launches. Write `launch.md` and `outcomes.yaml` for each, and run `outcome-monitor` and `launch-feedback-tagger` in digest-only mode (Band 1 and Band 2). Hold the 30-day readouts by hand using the `outcome-review.md` template.
6. **Weeks 11–12, close the loop:** Turn on the G6 Deploy hook for tracked intents. Add `impact-analyst`, `attribution-challenger`, and `learning-router`. Write the first `DISCOVERY.md` lessons and the first post-launch evals.
7. **Ongoing:** Add Band-3 thresholds for signals and launches. Track the outcome hit rate over time. Review the processes quarterly.

---

## 13. Open Questions for This Draft

**Discovery**
- Where is the system of record for opportunities: ProductSense, the repo (markdown), or both with links? The SDLC playbook allows Jira/ServiceNow-style systems to stay the source of truth as long as they link to markdown artifacts.
- What are the concrete Band-2 and Band-3 thresholds (ARR at risk, mention counts, tier rules)?
- Who is the Discovery Platform Owner?
- Which events trigger async AI-moderated interviews, and what is the consent and privacy policy for them?
- What is the fast-path threshold: who decides, and on what criteria?

**Post-Launch Learning**
- Which analytics and experimentation platform is the source of truth for launch metrics, and is it reachable via MCP?
- Do we require holdouts by default, and which launches can use pre/post comparison instead?
- What is the standard readout cadence (7/30/90 days?), and should high-risk or long-cycle B2B features have longer windows?
- Who fills the Data/Analytics Owner role for each team?
- Should the G6 Deploy hook block or only warn during the pilot?
- Where do outcome reviews live (repo, ProductSense, or both), and how are they linked back to opportunities?
