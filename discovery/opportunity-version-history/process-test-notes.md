# Process test notes: running a Slack request through the Agentic Discovery process (v0.2)

Test case: Gosta Labs request for Opportunity version history, 2026-10-23.

## How it mapped
| Stage | What happened | Artifact |
|---|---|---|
| Path decision (§2) | Not a bug or polish; new capability with compliance (viability) risk → full loop, enter at D2 | this file |
| D0 Frame | **Could not run: `DISCOVERY.md` does not exist** | none |
| D1 Sense | Slack message treated as a manual Band-3 signal; enriched via `identifyCompany` + `getCustomerDetails`; ARR/tier missing | `signal.md` |
| D2 Map | Duplicate check across 136 opportunities (none). Reframed "add a button" into the need: traceable record of change for QMS. Created #16516 | `opportunity.md`, ProductSense #16516 |
| D3 Verify | Agent pre-check with `findEvidence` (Moderate). Filters 2-4 not runnable with current data. No separate verifier session; PM sign-off pending | `opportunity.md` |
| D4 Engage | Call with Juuso is the obvious next step; needs PM | `test-card.md` questions |
| D5 Test | Riskiest assumption: description-only history satisfies QMS | `test-card.md` |
| D6 Commit | Skeleton only; PM must write core | `intent.md` |

## Process findings
1. **A commitment was made before D3.** Erno promised "coming week or two" in the thread. The process has no concept of a *customer commitment made ahead of verification*. Options: (a) add a rule: reply with "we'll look at it and come back by <date>" until G2; (b) add a **commitment** flag on opportunities so the PM consciously overrides D3 and records why (an "expedite" path). Suggest (b), since expediting a retention-critical request is legitimate. Record it in the decision log.
2. **Customer-originated request vs. ambient signal.** D1 assumes headless ingestion with bands from `signals.yaml`. A direct, explicit "we can't continue without it" message doesn't fit; it needs a named entry route (e.g. "direct request") with a default band.
3. **The four filters are built for volume.** With a single-account, high-importance request, filter 1 ("3 distinct quotes") and filter 4 (temporal windows) are the wrong shape. Need a defined rule for single-account compliance/retention requests (e.g. substitute: churn-risk statement + second account or prospect check).
4. **Enrichment gap blocks filter 2.** ARR and tier are not populated for the account in ProductSense, so weighting by ARR can't be done. The process says "no metadata → not promoted", which would stall this one. Fix the data, or define who fills it at D1.
5. **Compliance requests hide a bundle.** Same account asked for fixed-heading templates in Nov 2025 for the same regulatory reason. The mapper should group these as one opportunity "regulated documentation", not two features. Worth a golden eval case.
6. **System-of-record question (§13) is now concrete.** The opportunity lives in ProductSense (#16516) and the markdown artifacts live in `discovery/`. They link by id only; no mechanism keeps them in sync. Decide which is canonical.
7. **MCP gap:** `createOpportunity` cannot set teams, labels, themes, or link insights as non-CONFIRMED; linking adjacent insights would have recorded them as accepted support, so I left them unlinked. `findEvidence` tends to return adjacent items; the process needs a rule for "adjacent vs. supporting".
8. **`updateOpportunity` is the natural place for the decision log**, but there is no field for the four-filter status beyond free-text notes.

## Needs from the PM
- Confirm status: needs-research vs. verified (expedite override).
- Paste the Slack permalink into `signal.md` / `opportunity.md`.
- Set Gosta Labs ARR / tier.
- Schedule the call with Juuso (D4).
