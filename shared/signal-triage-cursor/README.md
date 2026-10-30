# Signal triage for Cursor

A Cursor agent skill that triages new customer insights from ProductSense, and your own product ideas, into a proposal you confirm in chat. Confirmed lines become new ideas or evidence updates in Jira Product Discovery.

## What it does

Each insight gets one recommendation, with the evidence behind it:

- **Create idea** in JPD, with a discovery tier (Light / Standard / Full)
- **New evidence** for an existing JPD idea
- **Post-launch feedback**: praise or usage feedback on something already launched. New requests about a launched idea become a new idea that links back to it.
- **Disqualify**, with the reason (previously rejected, out of scope, vague)
- **Route elsewhere**: bugs, support questions, praise
- **Needs clarification**: what to ask, and whom

Nothing is written to JPD until you confirm. You can confirm everything, part of it, or override any line.

## Setup

1. Copy the `signal-triage` folder to `.cursor/skills/signal-triage/` in your workspace.
2. Copy `discovery-context.template.md` to the workspace root as `discovery-context.md` and fill it in: your ProductSense team, JPD project and statuses, how ideas are updated, strategy, non-goals.
3. Make sure the Cursor agent has both MCP connections: ProductSense, and your Jira Product Discovery tool (read and update).

## Daily use

In Cursor's agent chat:

- `triage` — insights from the last 24 hours for your team
- `triage since Friday` — a wider window (e.g. on Mondays)
- `show A` / `show 4512` — the full assessment behind a line
- `do it`, `create the ideas`, `skip 4521`, `4519 is Standard` — confirm or override

For your own idea, just describe it: "I think we should let admins bulk-archive themes - triage this". The agent asks a few questions to get to the problem behind it, then checks it against JPD and customer evidence like any insight.

## What to know

- **Risks.** The agent estimates value and usability from customer evidence. It does not rate feasibility or business viability. Instead it writes the question each one needs answered by engineering or the idea owner.
- **Tiers are recommendations.** The agent explains the deciding factor and whether the call was borderline; you decide.
- **Disqualified and routed insights** are reported in chat only; nothing is written for them.
- **Overlapping windows.** If you run triage twice over the same days, insights already linked from a JPD idea are listed as "Already in JPD" and not re-assessed.
