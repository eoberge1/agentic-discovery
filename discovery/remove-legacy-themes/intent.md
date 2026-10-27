# Intent: Remove legacy themes
Author: Erno (PM), drafted with Claude. Status: accepted. Type: cleanup (no customer-visible change). Opportunity: ProductSense #16615

## Problem
We still have old themes in the product. They are no longer visible in the UI, but they are still created every night over every insight, per organisation, and the code for creating and viewing them, including the theme tools in the in-app chat and the MCP, is still there. Themes did not work in practice, because they are always too general; customers who saw them also understood that they give no meaningful information. Patterns replaced them: the user filters first, and only then are the insights clustered, which works better. The remaining theme job and code deliver no value.

## Proposed outcome
Patterns are the only way to cluster insights in ProductSense. The old way of creating and viewing themes is gone, and all code related to it has been removed, including the in-app chat tools and the MCP tools for themes.

## Affected users and systems
- Users: no visible UI change, since themes are already hidden. Users of the in-app chat and the MCP lose the theme tools. Theme-reliant customers (Workwize, RealPad) are already handled per PM.
- Systems: the nightly theme generation job, theme view code in the UI, theme tools in the in-app AI chat, theme tools in the ProductSense MCP server, theme data in the database (themes and insight-to-theme links), and any other code that reads themes (to be inventoried, see below).

## Constraints
- Patterns must keep working unchanged. Patterns must not depend on any theme code or data that is removed.
- Remove everything related to old themes: no feature-flagged or dead code left behind.
- Theme data is deleted, not archived. Because deletion cannot be undone, delete the data only after the code removal is in production and nothing reads the tables any more.
- No other feature may silently break or show empty states because themes are gone. Any dependent found must be handled explicitly (see below).

## Non-goals
- A new unfiltered "what is bubbling up across everything" view to replace themes. Old themes did not deliver this in practice. If it is needed, it is a separate opportunity.
- Changes to how Patterns work.
- Exporting or archiving existing theme data.

## Open questions (to be resolved by the agent running the AI-native SDLC, in Plan/Design)
- **Dependency inventory (required before any removal):** find every part of the code that uses themes (for example weekly summary, reports, dashboard, opportunity creation, related insights, prompts). For each dependent, propose in `spec.md` whether to remove it or switch it to Patterns; the product owner confirms in the spec review.
- **Chat and MCP tools:** list the exact theme tools in the in-app chat and the MCP server, and flag in `spec.md` if removing an MCP tool needs a notice to customers who use the MCP.

## Discovery evidence
- Opportunity: ProductSense #16615 (quotes from Alma Media, Metroc, Supermetrics, Teamspective, RealPad)
- Segment / ARR represented: not weighted; not needed for a cleanup task.
- Tests run: none. Patterns in production serve as the comparison.
- Risk: Value L (cleanup; themes already hidden) · Usability L (no visible UI change) · Feasibility M (unknown dependents) · Viability L-M (irreversible data deletion; MCP tool removal is an API change)
- Interrogation pass: not run

## Measurement plan
Not required. This is a cleanup task: themes are no longer visible in the UI, so there is no customer outcome to measure. Tracked by Maintain only, no Post-Launch Learning.

**Done when:** the nightly theme job is removed; theme code, chat tools and MCP tools are removed; every dependent from the inventory is handled as agreed in `spec.md`; Patterns work unchanged; theme data is deleted after the code removal is in production.

## Approval
- Product owner: Erno, 2026-10-26 (accepted into SDLC)
- Tech lead: pending (data deletion)
