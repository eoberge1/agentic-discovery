# signal-triage: contract v1 vs. the current ProductSense MCP

**Date:** 2026-10-28 (updated after the live dry run; see `signal-triage-dryrun-2026-10-28.md`)
**Purpose:** engineering handoff. What the `signal-triage` skill needs (`.claude/skills/signal-triage/references/mcp-contract.md`) versus what the ProductSense MCP exposes today, ranked by what blocks the skill. The last section is an interim mapping so the skill can be dry-run read-only before any server work.

Vocabulary: the contract says **signal**; the current MCP calls the same thing an **insight** (`searchInsights`, `insightIds`). Treat them as one concept.

## 1. Tool-by-tool

| Contract tool | Current MCP | Fit | Gap |
|---|---|---|---|
| `getDiscoveryContext` | none; `listKnowledgeBasePages` + `getKnowledgeBasePage` could hold a page by convention | partial | no structured non-goals, segments, ARR bands, thresholds, status classes |
| `listSignals(status: open, since)` | `searchInsights(startDate, endDate)` with no term | partial | **no signal status**. Nothing says whether an insight is already linked to an opportunity or was dismissed, so "open" cannot be computed |
| `getSignal(id)` | none; `searchInsights` rows carry `quote`, `note`, `customer`, `date`, `tags`, link | partial | no single fetch; no `source_type`, `status`, `linked_opportunity_ids`, `dismissal`, full `verbatim` beyond `quote` |
| `searchSignals(query, customer_id, include_dismissed)` | `searchInsights(searchTerm, selectedCustomers, dates)`; `findEvidence(specification)` | partial | **`searchInsights` is keyword matching, confirmed in the 2026-10-28 dry run** (long phrasings return 0; single tokens browse); rows lack `segment`, `tier`, linked opportunities, dismissal. `findEvidence` is semantic and strong but ~1 min per call, markdown only, one idea per call: usable for a cluster, too slow per signal |
| `searchOpportunities(query, statuses: all)` | `listOpportunities` (index, paged 100, no search) + `getOpportunity` for bodies | weak | **no search, no decision reasoning**. Rejections are at best `ICEBOX` with reasoning somewhere in the HTML notes; the prior-rejection check needs reasoning as a field |
| `getCustomer(id)` | `getCustomerDetails` (name, domain, stage, importance, agentSummary); ARR only appears as `customer.value` on insight rows; churn only via `listChurnSignals` (by window, not by customer) | partial | no ARR, segment, tier or churn risk on the customer object |
| `createOpportunity(signal_ids, tier, triage)` | `createOpportunity(title, notes, status, insightIds, impact, effort)` | good | `insightIds` link as CONFIRMED (fine after human confirmation). No `tier` field; map to `status`. No structured `triage` field; the record goes into `notes` HTML |
| `linkSignalToOpportunity(signal_id, opportunity_id, relation, note)` | none; `updateOpportunity` has no `insightIds` | **missing** | the "new evidence" outcome cannot be written |
| `dismissSignal(signal_id, reason_type, reason)` | none | **missing** | the "disqualify" outcome cannot be written, and nothing remembers it next run |

## 2. Gaps ranked by what they block

1. **Signal status and dismissal (`dismissSignal`, `status` on signals, `listSignals(status: open)`).** Without it the skill cannot select "open" signals and cannot record a disqualification. Every run re-triages the same signals and the "previously rejected" check has no memory. This is the one new concept the server needs: a dismissal record on an insight (reason type, reason text, related opportunity, user, time).
2. **Link a signal to an existing opportunity (`linkSignalToOpportunity`).** Today `insightIds` exists only on create. The merge outcome needs it on update, with a `relation` (supporting / adjacent) and a note.
3. **Opportunity search with decision reasoning (`searchOpportunities`).** Semantic search over title + problem across all statuses, returning a `decision { outcome, reasoning, decided_by, at }`. Needs a decision field on opportunities; today a rejection is `ICEBOX` plus free text.
4. **Customer enrichment on `getCustomer`.** ARR (the `value` already on insight rows), segment, tier, churn risk. Segment and tier may not exist as data yet; the skill treats them as `unknown` if null, so this is about exposing what exists.
5. **`searchSignals` row shape.** Inline `customer` with ARR already exists. Add `linked_opportunity_ids`, `dismissal`, `source_type`, and make the search semantic (or confirm it already is).
6. **`getDiscoveryContext`.** Lowest urgency: the skill degrades honestly without it. Interim convention: one knowledge base page titled "Discovery context" with headed sections (Non-goals, Strategic bets, Segments, Thresholds).

## 3. Decisions for the ProductSense side

| # | Question | Suggestion |
|---|---|---|
| A | Where does a dismissal live? | On the insight: `dismissal { reason_type, reason, related_opportunity_id, by, at }`. Visible in the app, searchable, reversible. |
| B | How does `tier` map to pipeline `status`? | Light → `IDEA` (curated, ready to prioritize); Standard and Full → `DISCOVERY`. Store the tier itself on the opportunity so it is not lost. |
| C | Where does the triage record go on a created opportunity? | A structured field is better than HTML notes: the record is JSON, and `notes` is replaced wholesale by any later edit. Interim: an `<h2>Triage</h2>` section in `notes`. |
| D | What counts as "rejected"? | A decision reasoning text on the opportunity. `ICEBOX` alone is **parked**: related only, never a merge target, never a rejection basis (contract status class `parked`). No ICEBOX item read in the dry run carried reasoning, so "previously rejected" cannot fire on this tenant until a decision field exists. |
| E | Is `searchInsights` semantic? | No (dry run, 2026-10-28). Expose a semantic variant or make `searchInsights` semantic; the prior-rejection and clustering checks depend on finding "version history" from "audit trail". |
| F | ARR is null on every insight row in the dry-run window | Either the CRM sync is not populating `value` or the accounts are unrated. Until fixed, every Value rating is evidence-count only and the skill reports "ARR missing for N of N". |

## 4. Interim mapping for a read-only dry run

The skill can run today in proposal-only form (nothing written) with these substitutions. Expect the degradation notes in the proposal header.

| Contract call | Use instead | Caveat |
|---|---|---|
| `getDiscoveryContext` | `listKnowledgeBasePages` → page titled "Discovery context" → `getKnowledgeBasePage` | if absent, state "no discovery context" |
| `listSignals(open, since)` | `searchInsights(startDate, endDate)` | cannot exclude linked or dismissed insights; say so in the header |
| `getSignal(id)` | the `searchInsights` row (`quote` as verbatim, `note` as extracted problem, `https://app.productsense.com/insight/<id>` as link) | no source type |
| `searchSignals(query)` | `searchInsights(searchTerm)`; for a cluster, one `findEvidence(specification)` | `findEvidence` is slow; one call per cluster at most |
| `searchSignals(customer_id)` | `identifyCompany` → `searchInsights(selectedCustomers)` | |
| `searchOpportunities` | `listOpportunities` (page all) → match titles against the normalized problem → `getOpportunity` on candidates for reasoning; `listKnowledgeBasePages` feature pages with `featureStage` IN_PRODUCTION serve as the shipped-capability check | `ICEBOX` = parked (related only); a rejection needs reasoning in the notes, which none had in the dry run; title matching is noisy (junk titles) |
| `getCustomer` | `getCustomerDetails` for stage and importance; ARR from the insight row's `customer.value`; `listChurnSignals(days: 90)` once per run for churn | segment and tier `unknown`; `listChurnSignals` is windowed by input date, not by customer, so an empty result means "nothing flagged tenant-wide" and the evidence block must say so rather than "none" |
| any write | none; list the actions for the user to apply in the app | |
