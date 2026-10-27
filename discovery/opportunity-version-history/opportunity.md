# Opportunity: Regulated teams need to show how an Opportunity changed over time, and who changed it

- **ID / link:** ProductSense #16516 · https://app.productsense.com/ (opportunity 16516)
- **Outcome served:** TBD, `DISCOVERY.md` does not exist yet. Candidate: retain and expand regulated / compliance-driven accounts.
- **Status:** needs-research (proposed by agent, PM to confirm at G2)
- **Owner:** Erno (PM)

## Problem & causal mechanism
Gosta Labs uses ProductSense as the place where product opportunities are documented, inside a medical-device quality management system (QMS). A QMS needs traceable records: what a document said at a point in time, who changed it, and when. ProductSense stores only the current Opportunity description, so edits overwrite earlier text and a QMS auditor cannot be shown the history.
**Mechanism (hypothesis, to verify with the requester):** the requirement comes from document-control expectations in their QMS, not from a general wish to see old text. This decides how much we must build (plain history vs. immutable, exportable record).

## Evidence
| Quote (verbatim) | Source link | Account | Tier | ARR | Churn status | Date |
|---|---|---|---|---|---|---|
| "Do you store version history of Opportunity descriptions? ... This would be kind of crucial from regulatory perspective." | Slack thread (add permalink) | Gosta Labs | n/a | not set | Subscribed | 2026-10-23 |
| "...to continue using ProductSense as part of medical device quality management system, version history is absolutely necessary." | Slack thread | Gosta Labs | n/a | not set | Subscribed | 2026-10-23 |
| "It would be important to see who has written the change. Basic audit trail." | Slack thread | Gosta Labs | n/a | not set | Subscribed | 2026-10-23 |
| "Basic audit trail (who did what and when)" (QMS needs follow-up; also said no need for signatures) | Slack thread | Gosta Labs | n/a | not set | Subscribed | <date> |
| "...would want to define Template with specific headings... a requirement from Medical Device Regulation perspective" (adjacent: templates, not history) | https://app.productsense.com/feedback/33588/transcript | Gosta Labs | n/a | not set | Subscribed | 2025-11-25 |
| Roadmap change log pain point (adjacent: roadmap, not descriptions) | https://app.productsense.com/insight/237688 | Smartly | n/a | not set | n/a | 2025-02-03 |
| Roadmap versioning / snapshots (adjacent) | https://app.productsense.com/insight/234978 | Metroc | n/a | not set | n/a | 2025-12-04 |

`findEvidence` verdict: **Moderate**, 3 customers. No direct request for *description* history from anyone else.

- **Segment weighting:** Not done. One account, importance 5/5, ARR unknown.
- **Negative space:** Not done. Unknown whether other regulated or compliance-driven accounts (or prospects) have the same need unspoken. Churned/lost accounts not checked.
- **Temporal check:** 14d: 1 request (this thread) · 60d: not run · 180d: not run → cannot classify yet. Related adjacent signals date from 2025.
- **Outliers worth noting:** The 2025-11-25 call shows the same account already framed ProductSense use around Medical Device Regulation, and asked for fixed-heading templates. The history request is probably part of a larger compliance bundle.

## Four-filter result
- [~] Verbatim quotes (≥3): 4 quotes, but one person, one account (not "distinct")
- [ ] Segment-weighted
- [ ] Negative space checked
- [ ] Temporally stable
- **Verifier decision & reason:** Not run by a separate verification agent. Agent pre-check only: cannot pass G2 yet. Filters 2-4 are open. **PM decision needed** (see "Process finding 1" in `process-test-notes.md`).

## Candidate solutions & tests
| Solution | Riskiest assumption | Risk type | Test card | Result |
|---|---|---|---|---|
| A. Description-only version history with author + timestamp, "History" button on the opportunity | History of the description alone satisfies their QMS | Viability | `test-card.md` | open |
| B. Full opportunity audit log (title, status, scores, links too) | Regulators/auditors expect all fields traced | Viability | not drafted | open |
| C. A + restore / export / immutability | They need an exportable or tamper-evident record | Viability / Feasibility | not drafted | open |

## Launch outcomes
| Intent | Launch date | Verdict | Outcome review | Routing |
|---|---|---|---|---|
| (none yet) | | | | |

## Decision log
- 2026-10-23: Request received in Slack. Erno committed to "coming week or two" before discovery ran. Recorded as a commitment, not as a verified opportunity. (Erno)
- 2026-10-23: Opportunity #16516 created in ProductSense, status DISCOVERY. (agent-drafted, PM to confirm)
