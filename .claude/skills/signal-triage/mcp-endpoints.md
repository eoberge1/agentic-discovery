# Topic: ProductSense MCP endpoints for the triage agent

Status: proposal, 2026-10-27. This file has two uses: it tells the agent which workaround to use today, and it is the backlog for ProductSense MCP work. When an endpoint ships, update the matching step in `SKILL.md` and mark it shipped here.

## What exists today

| Triage step (§5.1) | Tools available now | Good enough? |
|---|---|---|
| Read signal | `getOpportunity`, `identifyCompany`, `getCustomerDetails` | Partly. No owner, creator, created date or source link on the opportunity. |
| A. Problem clarity | `findEvidence`, `searchInsights` (`evidenceLens.problemClarity`) | Yes |
| B. Risk scan | `listKnowledgeBasePages`, `getKnowledgeBasePage`, `listTeams` | Partly. No feature-to-system/team mapping and no viability rules. |
| C. Evidence search | `findEvidence`, `searchInsights`, `listChurnSignals`, `getCompetitorDetails` | Partly. Counts, ARR sums and segments must be computed by hand; no segment/plan data. |
| D. Existing opportunities | `listOpportunities`, `getOpportunity` | **No.** Title-only scan, no semantic search, no linked insights, no decision history. |
| E. Disqualification | Same as D, plus KB for strategy | **No.** Rejection reason and date only exist if someone typed them into the notes. |
| Attach assessment | `updateOpportunity(notes)` | **Risky.** Rewrites the whole body with no conflict check, so it can overwrite a person's concurrent edit. |
| Find untriaged work | `listOpportunities` + `getOpportunity` per item | **No.** N+1 reads, and "untriaged" is detected by a heading in HTML. |

## Proposed endpoints

Priority: **P0** = without it the agent cannot reliably make the recommendation the process requires. **P1** = improves the quality of the assessment. **P2** = nice to have.

### P0

#### 1. `searchOpportunities`: semantic search over opportunities
Supports the existing-opportunity check (D), duplicate and merge detection, and prior-rejection detection (E).
- **In:** `query` (problem text), `statuses[]` (default: all, **including ICEBOX/archived/disqualified**), `limit`, `minScore`.
- **Out:** per hit: `id`, `title`, `status`, `similarity` (0–1), `matchedOn` (title/notes/linked insights), `triageOutcome`, `decision` summary (see #2), `updatedAt`.
- **Why:** paging `listOpportunities` and matching titles misses semantic duplicates ("CSV export" vs "Share reports with finance") and scales badly. Today's workaround is a full scan plus `getOpportunity` per candidate.

#### 2. Decision history on opportunities: `getOpportunityHistory` (or `decisions[]` on `getOpportunity`)
Supports the "previously rejected with valid reasoning" disqualification rule and re-triage.
- **Out:** chronological list of `{ type: status_change | triage_decision | merge | reopen, from, to, reason, decidedBy, decidedAt }`.
- **Why:** the process says disqualified opportunities are "archived with the reason" and must be quoted with date and author. Today the only signal is `ICEBOX` status plus whatever happens to be in the notes.

#### 3. `setTriageAssessment` / `getTriageAssessment`: structured, separate from notes
Supports attaching the assessment (§5.2) and §9 ("Triage assessment … attached to opportunity").
- **In:** `opportunityId`, `assessment` = `{ problemClarity, risks: { value, usability, feasibility, viability: { level, basis, needsHumanAssessment } }, evidence: { customerCount, mentionCount, arrTotal, arrUnknownCount, segments, quotes[{insightId, text}] }, relatedOpportunities[{id, relation}], recommendedOutcome, mergeTargetId, confidence, reasoning, overrideIf, markdown }`.
- **Behaviour:** stores a new version each run (history kept) and sets `triageState = pending_decision`. **Does not** change `status`. The owner's confirm/override is an app action that records a decision (#2) and sets `triageState = decided`.
- **Why:** removes the risky whole-body `notes` rewrite, gives the app a "Pending triage decisions" queue, and makes outcomes queryable for evals (agent recommendation vs. human decision).

#### 4. Richer `listOpportunities`: filters and fields
Supports running "automatically on new opportunities" (§5) and finding the owner to route to.
- **New filters:** `triageState` (`untriaged | pending_decision | decided`), `createdAfter`, `updatedAfter`, `statuses[]`, `ownerId`.
- **New fields (also on `getOpportunity`):** `ownerId`/`ownerName` (§11: every opportunity has an owner), `createdBy`, `createdAt`, `updatedAt`, `source` (`{ type: strategy|pm|customer_feedback|organization|post_launch|competitor, feedbackId?, customerId? }`), `linkedInsightCount`.
- **Why:** removes the N+1 scan and the HTML-heading heuristic; the source type sets which gaps triage should expect (§3).

#### 5. `linkInsightsToOpportunity` with `status: SUGGESTED`, plus `listOpportunityInsights`
Supports the Merge outcome ("Add as evidence to existing opportunity") and evidence comparison against a candidate.
- **In:** `opportunityId`, `insightIds[]`, `status` (`SUGGESTED` default, `CONFIRMED` only on explicit human acceptance, same model as `deliveryItems` `AI_CREATED`).
- **List out:** linked insights with status, customer, value, date.
- **Why:** today insights can only be linked as CONFIRMED at creation. The agent must not confirm, and can't see a candidate's evidence to judge "same request, same segment, nothing new".

### P1

#### 6. `summarizeEvidence`: aggregate over insights
Supports the evidence output (§5.1 C: customers, ARR, segment breakdown).
- **In:** `insightIds[]` or the same filters as `searchInsights`, `excludeCustomerIds[]`.
- **Out:** `distinctCustomers`, `mentions`, `arrTotal`, `customersWithoutValue`, `bySegment` (enterprise / mid-market / SMB), `byStatus` (prospect/trial/customer), `byChurnRisk`, `dateRange`, `byTeam`.
- **Needs:** segment/plan tier on customers. Extend `getCustomerDetails` with `segment`, `plan`, `arr`, `churnRisk`, `renewalDate`.
- **Why:** de-duplicating customers and summing ARR across pages by hand is error-prone, and segment data isn't exposed at all.

#### 7. `getDiscoveryStrategy`: strategy, OKRs, non-goals
Supports the "out of scope per DISCOVERY.md" disqualification and the "strategic bet" route to Full.
- **Out:** `{ priorities[], okrs[], nonGoals[{ text, owner, since }], strategicBoundaries[], updatedAt }`, or a designated KB page type `STRATEGY`.
- **Why:** the triage agent needs one authoritative source to quote non-goals from. Today it's a repo file or a KB page found by guessing titles.

#### 8. `getFeedback`: read the originating signal
Supports reading the raw source of the signal (problem clarity, source context).
- **In:** `feedbackId`. **Out:** source channel, customer, date, author role, transcript/text, extracted insights.
- **Why:** `findEvidence` and `searchInsights` link to transcripts, but there is no tool to read one. Clarity judgments made from the summarized `note` lose the originator's words.

#### 9. `addOpportunityComment` and owner notification
Supports handing off to the human ("Opportunity owner reviews output") and asking clarification questions.
- **In:** `opportunityId`, `body`, `mentionUserIds[]`, `kind` (`triage_ready | clarification_needed`).
- **Why:** questions for the owner or source shouldn't be buried in the notes; the owner should be notified that a decision is pending.

### P2

#### 10. `listResponsibilities` / `suggestAssessors`
Supports "Triage agent suggests which risks need human assessment" (§11.2).
- **Out:** people and teams with responsibilities (Value / Usability / Feasibility / Viability assessor) per product area.
- **Why:** turns "needs feasibility assessment" into "needs feasibility assessment: <team / person>".

#### 11. Feature/area map on knowledge base pages
Supports the feasibility and usability scans.
- **Out:** on `getKnowledgeBasePage` for features: `owningTeamId`, `systems[]`, `dependencies[]`, `linkedOpportunityIds[]`.
- **Why:** "multiple systems / dependencies" is the core feasibility signal, and today it's guessed from prose.

#### 12. `getViabilityRules`
Supports the viability risk scan.
- **Out:** org-defined triggers (keywords/areas → policy owner): pricing, PII, data residency, contracts, security.
- **Why:** makes viability flags consistent and routes them to the right policy owner.

#### 13. Event trigger: `opportunity.created` / `opportunity.reopened`
Supports running automatically. This is a webhook/trigger rather than an MCP tool, so the agent runs on creation instead of polling #4.

## Suggested build order

1. **#4 + #3** — find untriaged work and write results safely. Minimum for an automated agent.
2. **#1 + #2** — make Merge and Disqualify recommendations trustworthy.
3. **#5 + #6** — evidence that can be merged and aggregated.
4. **#7, #8, #9** — strategy, source fidelity, handoff.
5. **P2** as the agent is tuned against owner overrides (the #3 history gives the eval set).
