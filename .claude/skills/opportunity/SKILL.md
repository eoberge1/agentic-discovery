---
name: opportunity
description: Work with product Opportunities in ProductSense (via the ProductSense MCP) as part of the Agentic Discovery process. Use when creating, reading, updating, or reviewing an opportunity, or when drafting, adding, or exporting the intent.md sections an opportunity carries (the SDLC entry artifact). Triggers on "opportunity", "intent", "intent.md", "ProductSense opportunity", "write up this idea", "hand this to the SDLC".
---

# Opportunity

Opportunities are the unit of discovery work. They are part of the Agentic Discovery process defined in `agentic-discovery-process-v2.md` (stages: Capture → Triage → Curated/Plan → Prioritize). This skill is modular: each topic below lives in its own file. Read only the topic(s) the task needs.

## Core rules (always apply)

1. **ProductSense is where opportunities are managed.** Use the ProductSense MCP tools; never keep a parallel opportunity list elsewhere.
   - Find: `listOpportunities` (paged, 100 per call; page through all before concluding something doesn't exist).
   - Read: `getOpportunity` (the body is the HTML `notes` field).
   - Create: `createOpportunity` (run `listOpportunities` first to avoid duplicates).
   - Edit: `updateOpportunity`. **`notes` replaces the whole body with no conflict check.** Always `getOpportunity` immediately before writing, edit the HTML you read rather than rewriting from scratch, keep the gap short, and preserve every `<img>` tag (dropped images are permanently deleted).
   - Evidence: `searchInsights`, `findEvidence`, `getCustomerDetails`, `identifyCompany`, `listChurnSignals`, `getCompetitorDetails`.

2. **Every opportunity has at least one intent section, and often several.** An opportunity is a customer problem. Each intent section is one proposed change against that problem, written to the `intent.md` standard (see [intent.md](intent.md)). When you create an opportunity, also add a first intent section, even if it is only a draft with placeholders. When you touch an opportunity that has none, flag it and offer to draft one.

3. **Agents draft, humans decide.** Never mark an intent `accepted`, never move an opportunity's pipeline `status`, never confirm triage outcomes (disqualify, merge, tier), and never link insights as confirmed support without the opportunity owner saying so. Propose; let the human confirm. (`createOpportunity` `insightIds` are linked as CONFIRMED, so only pass insights the human has accepted.)

4. **Don't invent the originator's meaning.** Problem and outcome text comes from the originator's own words or from linked evidence. If you don't have it, ask, or leave a visible `<to fill: ...>` placeholder.

5. **Write changes back.** After any edit to an opportunity, tell the user the opportunity id, which sections changed, and anything left as a placeholder.

6. **Responsibilities are flexible.** The process uses responsibility-based ownership, not fixed roles. "Opportunity owner" is whoever is accountable for the opportunity. This may be a PM, an engineer, a designer, or anyone else. See `agentic-discovery-process-v2.md` §11 for details.

## Triage support

When working with opportunities, support the triage process:

- **New opportunities** should be ready for triage: clear problem statement, source identified, enough context for the triage agent to assess.
- **Triage assessments** may be attached to opportunities. Don't modify these without human instruction.
- **Disqualified opportunities** are archived with reasoning. Don't reopen or recreate them without human instruction.
- **Merged opportunities** link to their parent. Add evidence to the parent, not the merged duplicate.

## Topics

| Topic | File | Use when |
|---|---|---|
| Intent sections (`intent.md`) | [intent.md](intent.md) | Drafting, adding, reviewing, or exporting an intent on an opportunity |
| Triage | (inline above) | Understanding how triage affects opportunity workflow |

<!-- Add new topics here as rows, each in its own file in this folder. -->
