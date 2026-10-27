# Topic: intent sections (`intent.md`)

Source: Anthropic, *The AI-native SDLC playbook*, Stage 1 Plan, lesson "Capture as intent.md"
(https://academy.claude.com/courses/ai-native-sdlc-playbook/capture-intent), extended with the discovery fields from `agentic-discovery-process.md` §D6 and §11.3.

## What an intent is (Anthropic)

- `intent.md` is a **proto-spec**: what is wanted, why, and under which constraints, written **in the originator's own terms**. It replaces the backlog entry → user story → refinement chain, where each handoff moves the text further from what the originator meant.
- It is human readable, version controlled, and immediately consumable by the next stage (Stage 2 Design, which writes `spec.md`).
- It can start from a person's idea, a ticket, or an incident surfaced by an alert. **The same steps apply whatever the origin.**
- The **product owner reviews and corrects** the agent-written intent before it is committed, and the accept/reject decision is what sends it into Stage 2 Design.

## Opportunity vs. intent

An opportunity (customer problem) holds **one or more** intent sections. Split into separate intents when the parts:
- could be accepted or rejected independently by the product owner,
- have a different proposed outcome or success measure, or
- touch clearly different users or systems (for example a quick fix now plus a larger capability later).

Keep it as one intent if splitting would leave a part with no outcome of its own. Each intent should stay short: Anthropic's example is about ten lines; the discovery process caps it at 1 to 2 pages.

## How to construct an intent (Anthropic's steps)

1. **Originator describes the problem in their own words.** What they cannot do today, who is affected, what better looks like, what is out of scope. No formal language needed. If the originator isn't present, start from the opportunity's evidence (quotes, insights) and say so.
2. **Brainstorm until concrete.** Ask the questions an analyst would ask, a few at a time:
   - *Scope:* what is in, what is explicitly out?
   - *Users:* who has the problem, in what situation? Which segments, tiers, accounts?
   - *Constraints:* what must not change or break (security, PII, auth, compliance, performance, existing integrations)?
   - *Success:* what would be observably better, for whom, by how much?
3. **Write the result in the template below.** Keep the originator's phrasing for Problem and Proposed outcome; tighten, don't reword.
4. **Originator corrects** anything misunderstood. Show the draft and ask explicitly.
5. **Commit / save.** Write it into the opportunity in ProductSense (and, if the team uses one, the intent home, see "Export" below). Author and timestamp go on the record. The product owner picks it up from there.

## Template

Anthropic's core sections are marked **(core)**. The rest are the discovery extensions required by our process before an intent can be accepted into the SDLC.

```markdown
# Intent: <short title>
Author: <name (role)>. Status: draft | in-review | accepted | rejected. Opportunity: <ProductSense #id>

## Problem                      (core)
<What the originator cannot do today, who is affected, in what situation, and why it matters. Concrete, with a number if there is one.>

## Proposed outcome             (core)
<What better looks like, in the originator's terms. An observable result, not a solution design.>

## Affected users and systems   (core)
<User groups / segments / tiers; teams; systems, APIs and integrations touched.>

## Constraints                  (core)
<Non-negotiables the change must respect. Include guardrails against "locally rational, systemically wrong" builds.>

## Non-goals
<What we are explicitly not doing.>

## Open questions               (core)
<Unresolved items, each with an owner.>

## Discovery evidence
- Key quotes: <2–3 verbatim, linked>
- Segment / ARR represented: <...>
- Tests run: <test cards + results>
- Risk: Value <L/M/H> · Usability <...> · Feasibility <...> · Viability <...>
- Interrogation pass: <date, # issues raised / resolved>

## Measurement plan
- Primary outcome metric: <metric> · baseline <x> · target <y> · by day <n>
- Adoption definition: <events>
- Guardrail metrics: <must not degrade>
- Target cohorts / comparison: <segments; holdout or pre/post>
- Readouts: <e.g. 7 / 30 / 90 days>
- Kill criteria: <conditions for rollback or sunset>

## Approval
- Product owner: <name, date>
- Tech lead (if high-risk): <name, date>
- Policy owners (if flagged): <name, date>
```

Anthropic's reference example (shows the expected tone and length of the core):

```markdown
# Intent: claims status self-service
Author: J. Ortiz (claims operations). Status: draft.
## Problem
Customers phone the contact center to ask where their claim is.
Handlers spend roughly a third of call time on status-only queries.
## Proposed outcome
Customers see claim status, next step and expected date in the portal.
## Affected users and systems
Claims handlers, portal team, claims-core API.
## Constraints
No new PII in the portal session. Existing authentication only.
## Open questions
Do third-party loss adjusters need access too?
```

## Quality checks before marking `in-review`

- Problem names **who**, **what they can't do**, and **why it matters**. No solution hidden in the problem.
- Proposed outcome is observable and could become a metric.
- Constraints include at least one guardrail a coding agent might otherwise violate (the D6 *determinism check*: could an agent build something locally rational but systemically wrong from this?).
- Every open question has an owner.
- Nothing in Problem or Proposed outcome was invented; placeholders are visible.
- The intent is short. If it reads like a spec (screens, API shapes, task lists), cut it back; that belongs in `spec.md`.

An intent with only the core sections filled can be `draft` or `in-review`. It can only be **accepted into the SDLC** once Discovery evidence and Measurement plan are filled (fast-path intents may skip evidence; see process §2).

**Cleanup intents** (no customer-visible change, e.g. removing code for a feature already hidden from the UI): mark `Type: cleanup` in the header line. Write the Measurement plan as "Not required" with the reason, and add a **Done when:** list of verifiable completion conditions instead. These are tracked by Maintain only, with no Post-Launch Learning.

**Questions delegated to the SDLC:** when the PM leaves an open question to the agent running the AI-native SDLC (e.g. a code dependency inventory), say so in the Open questions heading. For each question, state what the agent must produce in `spec.md` and who confirms it.

## Storing intents in ProductSense

Intent sections live in the opportunity's `notes` (HTML). Use this structure so sections are easy to find and export. Put intents after any existing opportunity content; number them in order.

```html
<h2>Intent 1: claims status self-service</h2>
<p><strong>Author:</strong> J. Ortiz (claims operations). <strong>Status:</strong> draft.</p>
<h3>Problem</h3>
<p>...</p>
<h3>Proposed outcome</h3>
<p>...</p>
<h3>Affected users and systems</h3>
<p>...</p>
<h3>Constraints</h3>
<ul><li>...</li></ul>
<h3>Non-goals</h3>
<ul><li>...</li></ul>
<h3>Open questions</h3>
<ul><li>... (owner: ...)</li></ul>
<h3>Discovery evidence</h3>
<ul><li>...</li></ul>
<h3>Measurement plan</h3>
<ul><li>...</li></ul>
<h3>Approval</h3>
<ul><li>Product owner: pending</li></ul>
```

Rules:
- Follow the core rules in [SKILL.md](SKILL.md) for every `updateOpportunity` call (read immediately before, edit don't rewrite, keep `<img>` tags).
- Adding an intent: append a new `<h2>Intent N: ...</h2>` block. Don't renumber or rewrite existing intents.
- Changing an intent that is `accepted`: don't edit it silently. Anthropic counts changes to `intent.md` after `spec.md` exists as a quality signal, so add a dated line under its Approval list describing the change, and tell the PM.
- Rejected intents stay in the notes with status `rejected` and a one-line reason; they are part of the record.

## Export to `intent.md`

When an intent is ready to hand to the SDLC, or the user asks for the file, convert the section to markdown with the template above and save it to the intent home (Anthropic's default: an `intent/` folder in the product repo; for now in this workspace, `discovery/<opportunity-slug>/intent.md`, or `intent-<n>-<slug>.md` when an opportunity has several). Put the ProductSense opportunity id in the header line so the two stay linked.

## How Anthropic measures this (for reporting)

- **Leading:** time from first conversation to a committed intent (target: hours, not weeks).
- **Lagging:** survival rate, the share of intents the product owner accepts into Design rather than closes; plus the number of intent changes made after the first `spec.md` commit.
