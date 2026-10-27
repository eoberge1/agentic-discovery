# Intent: Opportunity version history with author audit trail
Author: Erno (PM), agent-drafted 2026-10-26 from the requester's words and PM decisions. Status: accepted (2026-10-26). Opportunity: ProductSense #16516 (Intent 1)

> **STATUS: ACCEPTED 2026-10-26 (Erno, PM). Ready for SDLC Stage 2 Design (spec.md).** ProductSense #16516 moved to DEVELOPMENT.

## Problem
Customers who use ProductSense in a regulated quality management system (medical device) must be able to show how an Opportunity description changed over time and who changed it. ProductSense does not store or show that history today. Gosta Labs says version history is "absolutely necessary" for them to keep using ProductSense in their QMS.

## Proposed outcome
From an Opportunity, a user can open its version history ("a button to show the version history") and see every saved version of the opportunity's description, title, status, effort, importance and linked insights: when it was saved, who wrote it ("basic audit trail"), and where the change came from (UI, MCP or Chat). For MCP and Chat changes, the version also shows the reason the agent gave for the change.

## Affected users & systems
Regulated / QMS customers (Gosta Labs first) and anyone editing Opportunities. Systems: Opportunity editor (UI), Chat, opportunity storage (new history table), image storage (S3), ProductSense MCP (createOpportunity / updateOpportunity), existing author attribution.

## Constraints & guardrails
- Stored versions are immutable: no path in the UI, API or MCP can edit or delete a stored version.
- Every change to a versioned field (description, title, status, effort, importance, linked insights), through the UI, the MCP or Chat, creates a version that records the user who made it and the origin. MCP and Chat edits are not a side door around the history.
- MCP and Chat edits carry a reason for the update, filled in by the agent. UI edits do not ask for a reason.
- Images stay in S3 so old versions still show them. An image removed from the current description must not be deleted in a way that breaks older versions.
- History starts at launch. Edits made before launch are not backfilled.
- No loss or change to existing opportunities when the feature ships.

## Non-goals
- Reverting to an old version (later step; requirements to come from the customer after launch).
- Asking for a reason on UI edits.
- Templates with fixed headings (separate Gosta Labs ask).
- Electronic signatures and approval workflows on versions.
- Export of the history.
- Backfilling history for edits made before launch.
- Roadmap versioning / snapshots (Smartly, Metroc asks).

## Decisions (PM, 2026-10-26)
- **Audit trail scope:** a change log plus the user who made each change is enough. No e-signatures and no export for now.
- **QMS need** (Juuso, Slack): "Basic audit trail (who did what and when)", no need for signatures. A change log plus author, with immutable versions, meets it.
- **Immutability:** stored versions cannot be edited. History is append-only. When revert is built later, reverting creates a new version at the top of the list with the old version's content.
- **MCP:** in scope. MCP edits create versions and are attributed the same way as UI edits.
- **Templates with fixed headings:** out of scope for now (separate ask).
- **History start** (technical discussion): history starts at launch, no backfill.
- **Versioned fields:** description (note), title, status, effort, importance and linked insights.
- **First step = store and show only:** store versions and show old versions in the UI. No revert yet. Revert requirements are asked from the customer once the first step is live.
- **Reason for the update:** a field in the MCP / Chat cases that the agent fills in. Not asked or stored for UI edits for now, as it does not fit the editing flow.
- **Images:** stay in S3, where they are stored today, so old versions still find their images.

## Technical discussion (2026-10-26), input for spec.md
- New append-only history table, one row per change: opportunityId, date, version, note, previous note, reason for the update, user, origin (UI, MCP / Chat).
- Extended by PM decision: the table also stores title, status, effort, importance and linked insights. How previous values of these fields are stored is for spec.md.
- Reason for the update is filled by the agent for MCP / Chat edits and left empty for UI edits.
- Images stay in S3; history references them where they are today.
- History starts at launch; no backfill.

## Open questions
- Revert requirements (all fields at once or a single field), to ask the customer after the first step is live. (owner: PM)

## Discovery evidence
- **Opportunity:** ./opportunity.md · ProductSense #16516
- **Key quotes:** see opportunity.md (4 quotes, one person, one account)
- **QMS need (Juuso, Slack follow-up):** "Basic audit trail (who did what and when)", no need for signatures.
- **Segment / ARR represented:** Gosta Labs, ARR not set. Importance 5/5
- **Tests run:** none yet (`test-card.md` drafted, not run)
- **Risk assessment:** Value: M (explicit retention-level statement, one account) · Usability: L (standard pattern) · Feasibility: L (new history table agreed in technical discussion 2026-10-26; author data exists in UI and MCP) · Viability: L (requester confirmed the QMS need is a basic audit trail, no signatures)
- **Interrogation pass:** not run

## Measurement plan  (expanded into launch.md during SDLC Design)
- **Primary outcome metric:** Gosta Labs keeps using ProductSense in their QMS at day 90
- **Accepted with known gap (PM, 2026-10-26):** adoption definition, guardrail metrics and kill criteria are to be completed during Design (launch.md).

## Approval
- Product owner: Erno, 2026-10-26
- Tech lead (if high-risk): Erno, 2026-10-26
- Policy owners (if flagged): Erno, 2026-10-26
