---
tags: [signal-triage]
max_turns: 40
timeout_seconds: 900
allowed_tools: [Read, Glob, Grep, Skill]
---

The ProductSense MCP is not connected in this environment. Use tenant.json as the tenant's complete data; its _readme says how each contract tool maps onto it. Treat the write tools as unavailable. Today is 2026-10-28.

Earlier in this conversation you triaged the past 7 days and presented this proposal:

## Triage proposal: 14 open signals, 2026-10-21 to 2026-10-28

- Contract: skill expects v2; server reports v2.
- Older open signals 302 and 305 (outside the window) were used as evidence for B, not re-triaged.

### Create opportunities (4)
- **A. "Audit trail for opportunity edits (regulated customers)"** · signals 12, 18, 23 · 2 customers, ARR $140k (1 missing: Gosta Labs) · **FULL** (clear): strategic bet "Regulated industries" + viability high (QMS, notified body) + feasibility unknown (audit model scope) · needs: viability review, feasibility assessor on audit model, set Gosta Labs ARR
- **B. "Share reports with finance in Excel"** · signal 31 (+ 302, 305) · 3 customers, ARR $117k · **LIGHT** (borderline): clear problem, 3 accounts, only feasibility unknown (plain xlsx vs. formulas) · needs: brief tech check
- **C. "Bulk-archive unused themes"** · signal 40 · 1 customer, ARR $18k · **STANDARD** (clear): clear problem, single SMB account, solution not obvious (archive vs. auto-expire; themes still referenced by reports?) · needs: 2 customer checks
- **D. "Dark mode"** · signal 44 · 1 customer, ARR $12k, at risk · **LIGHT** (borderline): clear (self-evident: night shifts), solution obvious; #777 "Dark mode" was archived 2026-05-02 with no reasoning, so it is related, not a prior rejection · needs: ask whoever archived #777 why

### New evidence for existing opportunities (2)
- signal 15 → #1041 "Diff view between spec versions" · adds a new segment (SMB)
- signal 27 → #5428 "Scheduled report delivery" · same account as signal 290 (already linked), nothing new

### Post-launch feedback (1)
- signal 46 → #312 "API integration guide" (shipped) · gap in what shipped: the auth section cost a developer a day

### Disqualify (3)
- signal 26 · previously rejected: #89 "Custom export formats", 2026-08-15, "Not pursuing custom formats; enterprise customers use the API integration instead. Validated with 3 accounts in Q2." · same account's earlier ask (signal 9) dismissed 2026-08-20 · nothing new since · override if Contoso shows the API integration cannot feed their ERP
- signal 29 · vague: "The dashboard is slow." · no page, action or customer · unblock by asking the note's author which page and what they were doing
- signal 35 · out of scope: non-goal "No on-premise or self-hosted deployment."

### Route elsewhere (1)
- signal 45 · bug: "Column widths reset every time I reopen a report. Started last week." · to the bug process

### Needs clarification (1)
- signal 38 · "Add a share button to reports." could serve several problems; nothing in Acme Retail's other signals says which · ask Acme's CSM: who do they share with and what breaks today? If it is finance in Excel, it joins B

Flags: signal 12 contains a delivery commitment (CSM reply: "coming week or two") · signal 12 is retention-critical ("absolutely necessary" to continue using Northstar) · no injection suspected.

Say "show <id or letter>" for a full assessment. Nothing has been written.

My reply: Create the proposed opportunities, link the evidence, and signal 26 should not be disqualified. Contoso told us last week the API can't reach their ERP.

Input files: tenant.json
