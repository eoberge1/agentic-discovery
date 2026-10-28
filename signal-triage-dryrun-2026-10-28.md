# signal-triage dry run on the live tenant, 2026-10-28

Read-only run of the v1 skill with the interim mapping from `signal-triage-mcp-gaps.md` §4. Window: past 7 days, cap 8 signals (20 in the window). Nothing was written. Executed by a fresh agent reading the skill files; this is its report, lightly trimmed.

## A. Proposal as produced

```markdown
## Triage proposal: 8 open signals, 2026-10-21 to 2026-10-28

- Assessed 8 of 20 in the window (cap set to 8 for this run); say "continue" for the remaining 12 (716109 "version history of Opportunity descriptions" from ProductSense, 2026-10-23, plus 11 more Workpay signals from 2026-10-21 and 2026-10-26). The 8 assessed are all from one Workpay call on 2026-10-26; the cap split that call (10 insights) in two.
- No discovery context (no knowledge base page titled "Discovery context"): out-of-scope and strategic-bet checks skipped; default thresholds; contract v1 expected, server version unknown.
- Signal status unavailable: cannot exclude insights already linked to an opportunity or dismissed. Signal 724941 is already quoted inside #16714, so it may already be linked.
- Dismissed-signal check not run (no dismissal concept on the server); "previously rejected" could only be checked against opportunity notes, and none of the 4 candidates read carried decision reasoning.
- Opportunity search was a title match over 141 opportunities (no semantic search, no decision field); ICEBOX treated as "cannot confirm rejection", not as a merge target.
- Customer enrichment: Workpay is TRIAL, importance not rated; ARR missing (null on every row in the window); segment and tier unknown. Churn: listChurnSignals(90 days) returned no inputs at all, so "no churn signals" means none flagged tenant-wide, not a per-customer check.
- Evidence search: searchInsights is keyword matching; findEvidence (semantic) was used twice, for A and for 724941. Evidence for the other lines is keyword-only.
- No write tool exists for "new evidence" or "disqualify"; even after confirmation those must be applied by hand in the app.

### Create opportunities (1)
- **A. "Ingest Google Meet / Gemini meeting notes as a meeting source"** · signal 724940 · 7 customers, ARR unknown (7 missing) · **STANDARD** (borderline): clear problem (inferred from 724834, 724839), 7 supporting accounts, feasibility unknown (access model: Gemini notes reach only meeting participants), viability medium (confidential-content exclusion raised by the customer) · needs: tech spike on workspace-level access to Meet/Gemini notes; viability check on excluding confidential calls; set Workpay ARR

### New evidence for existing opportunities (4)
- signal 724941 → #16714 "Intercom customer detection: inputs attributed to the workspace owner and duplicate customers (Workpay)" (IDEA) · adds nothing new: same account, same conversation already quoted in the notes
- signal 724943 → shipped capability, KB feature "Intercom" (IN_PRODUCTION) · post-launch: the problem is what the integration addresses; adds a trial account, no ARR
- signal 724939 → shipped capability, KB feature "Account health & sentiment" (IN_PRODUCTION; nearest opportunities #3251 LAUNCHED, #3580 ICEBOX with no reasoning) · post-launch positive feedback · nothing new
- signal 724938 → #1667 "Team filter for feedback and insights" (LAUNCHED; also #1930, #775 LAUNCHED; #5893 "Team selector" ICEBOX, no reasoning) · post-launch positive feedback · nothing new

### Disqualify (3)
- signal 724942 · no basis (other: commercial context, no product ask): "we've not budgeted for any product tools" · relevant to the deal owner, not to discovery
- signal 724937 · no basis (other: org context, no product ask): "we have four PMs now who work under me" · useful as account context only
- signal 724936 · no basis (other: tool-stack context, no product ask): "we have intercom that does ... customer issues, and then we have HubSpot where enhancements are tracked" · Intercom and HubSpot integrations are in production (KB)

### Needs clarification (0)

Flags: none. Note for the owner of A: findEvidence surfaced a vendor commitment to Ovido on 2025-08-21 to "arrange the Gemini setup" (insight 723004); it is outside this window and is not a flag on any assessed signal.

Say "show <id or letter>" for a full assessment. Nothing has been written.
```

The full assessment for A (recommendation STANDARD, borderline: Light conditions met on clarity, 7 accounts and risk counts, but fails "solution obvious" because the access model is unresolved) is in the agent transcript; the shape followed `proposal-format.md`.

## C. Tool observations

| Tool | Calls | Notes |
|---|---|---|
| listKnowledgeBasePages | 1 | 110 pages. No page titled "Discovery context". Useful as a feature inventory for the "shipped" check (featureStage IN_PRODUCTION vs IDEA). |
| getKnowledgeBasePage | 1 | ~1,500 words for "Meeting & notetaker integrations"; settled "no existing opportunity" for A. |
| searchInsights | 13 | Window query (dates, no term) returned 20 rows, date-desc; 10 Workpay rows share one identical timestamp. Max 100 per page. |
| findEvidence | 2 | Both in one parallel batch; the batch returned after roughly a minute. Each result ~2,000–2,500 words: strength grade, ranked items with verbatim transcript quotes, caveats, gaps, phrasings searched. The only semantic search in the run and the only source that found the other six accounts for A. |
| identifyCompany | 1 | Resolved "Workpay" to one id. |
| getCustomerDetails | 1 | stage TRIAL, importance "not rated", agentSummary null. No ARR, segment, tier or churn. |
| listChurnSignals | 1 | days=90: nothing flagged tenant-wide. |
| listOpportunities | 2 | 141 rows, titles and status only. Many junk titles ("test", "", "Opportunity   love"), so title matching is noisy. |
| getOpportunity | 4 | #16714 rich; #5893 one line; #4802 empty; #3580 one bullet. None carried decision reasoning. |

**searchInsights is keyword matching, not semantic.** `"same customer different company names across tools"` → 0 results although insight 724941 contains almost those words; `"duplicate customers"` → 10 results, 1 relevant; `"customer matching"` → 12, none about customer identity; `"Read AI"` → 3, one containing neither token (tokens OR'd or stemmed). Single tokens work as a browse (`"notetaker"` 43, `"Gemini"` 21).

## D. Skill friction found (quote → what the agent did)

1. §1 "take the most recent 25": undefined on timestamp ties (10 insights, one timestamp); the cap split one call in half and dropped the only non-Workpay signal. Define the tie-break; do not split a source conversation.
2. Step 3 defines `shipped` but Step 6 and the proposal have no outcome for it. 3 of 8 signals were post-launch feedback; forced into "New evidence" with a note. Add a post-launch group.
3. "Disqualify: no basis — internal suggestion" does not cover context-only rows (budget, org, tool stack). Wrote "no basis (other: … context)". Add a rule for rows with no product ask.
4. "Keep the originator's verbatim; it is the evidence": on this tenant `quote` is often content-free ("Okay. Okay. Yeah, because that would be important.") and the problem lives in `note`. Quoted both, labelled. Say what to do when the verbatim carries no content.
5. Light rule "at most one risk unknown" + "Value: unknown when one account and no ARR" + "Feasibility: unknown almost always": with ARR null on every row, any single-account signal has two unknowns and goes Full regardless of size. ARR missing for all accounts should not drive the tier.
6. "Solution obvious or tightly constrained" is undefined. Went Standard, borderline, on an unresolved access model.
7. Contract status classes make ICEBOX *active* (merge target); gaps §4 says ICEBOX ≈ rejected. Contradiction. Treated ICEBOX as "cannot confirm rejection, not a merge target".
8. "Per signal, getSignal and getCustomer": eight signals, one customer. Fetched once. Say "once per distinct customer".
9. listChurnSignals is windowed by input date, not customer; "Churn signals: none" needs a qualifier under this mapping.
10. findEvidence returns transcript quotes with a feedback link and no insight id; these were the strongest quotes. Allow any id + link a tool returned.
11. Flags are scoped to the signal text; a real vendor commitment (Ovido, 2025-08-21) was in the evidence. Consider a "prior commitment found in evidence" flag.
12. "Group also by driver": the driver behind five Workpay signals is the product's core value proposition; clustering by driver would have produced an opportunity called "ProductSense". Needs a stop: driver that is an in-production capability → post-launch feedback.
13. "The skill reports the contract version" has no slot in the proposal template.
14. proposal-format says "Flags, only when present"; example-run ends its Flags line with "no injection suspected". Minor.
15. "New evidence" line has no form for a shipped capability without an opportunity (KB feature page).
16. Inferring clarity from the same call's other insights: acceptable? Did it and said so.

## E. Steps not run under the interim mapping (all signals)

Open-status filter; discovery context; source type; semantic opportunity search; decision reasoning; dismissed-signal search; per-customer churn; ARR, segment, tier; injection check on the full transcript (quote + note only). The "previously rejected" rule could never fire: no reasoning on any ICEBOX item.

## What this changes

- Skill revision (iteration 2): items 1–16 above plus the fixture-eval findings (bug/defect route; one-line asks over-flagged as needs-clarification; Light vs Standard example inconsistency).
- Gap list: `searchInsights` confirmed keyword-only (gap 5 is real); ICEBOX semantics decided (parked: neither active nor rejected); ARR null tenant-wide is a data problem to fix before any Value rating means much.
