# Topic: the Triage Assessment

Format from `agentic-discovery-process-v2.md` §5.1 E and §5.2. Show it to people as markdown; write it to ProductSense as HTML (the opportunity `notes` field is HTML from the rich text editor).

## Markdown template

Omit a block only when it doesn't apply (for example, *Attempted clarification* when the problem is clear). Never omit Risk scan, Existing opportunities or Recommended outcome.

```markdown
## Triage Assessment
Triaged by: signal-triage agent · <YYYY-MM-DD> · Status: pending owner decision
<Re-triage of <previous date>: <what changed>>   ← only on re-triage

**Problem clarity:** Clear | Clear (recovered from evidence) | Needs clarification | Vague
> "<verbatim problem statement, from the signal or the evidence it was recovered from>"

**Attempted clarification:**            ← only if not clear
- <what was searched, what was found>
- Questions for the owner: <specific questions>

**Risk scan:**
| Risk | Level | Basis | Needs human assessment |
|------|-------|-------|------------------------|
| Value | low/medium/high/unknown | <one line> | <yes: Value assessor / no> |
| Usability | ... | ... | ... |
| Feasibility | ... | ... | <yes: Feasibility assessor (team: <name>)> |
| Viability | ... | <trigger named, if any> | ... |

**Evidence:**
- Need strength (findEvidence): Strong | Moderate | Weak | No evidence found
- <N> mentions from <M> other customers (<segment breakdown, or "segment unknown">)
- ARR represented: $<sum> (<K> customers without a value)
- Date range: <earliest> – <latest>
- Key quotes:
  - "<quote>" — <customer>, <date> ([link](https://app.productsense.com/insight/<id>))
- Churn link: <customer at risk, link> | none found
- Caveats: <existing capability / different approach wanted / none>

**Existing opportunities:** (<N> scanned, title-based match)
- Match: #<id> "<title>" (status) | Related: ... | Previously rejected: ... | Already shipped: ... | None
- Prior reasoning: "<verbatim>" (<author>, <date>)   ← if rejected

**Strategic fit:** <aligned with / conflicts with "<quoted non-goal>" / DISCOVERY.md not found>

**Recommended outcome:** DISQUALIFY | MERGE → #<id> | LIGHT | STANDARD | FULL
**Confidence:** high | medium | low
**Reasoning:** <2–3 sentences>
**Next step:** <Light: what intent.md still needs · Standard/Full: activities from §7.1 · Merge: which evidence to move · Disqualify: archive with this reason>
**Override if:** <what would change this recommendation>
```

For a DISQUALIFY with a prior rejection, include the §5.1 E block fields: **Reason**, **Prior decision** (id, title, date), **PM reasoning** (verbatim), **Current signal adds**.

## HTML for ProductSense

Mirror the markdown with editor tags: `<h2>` for the heading, `<p>` per line, `<blockquote>` for the problem quote, `<table>` for the risk scan, `<ul>/<li>` for lists, `<a href>` for links, `<strong>` for labels. Wrap every paragraph in `<p>…</p>`.

```html
<h2>Triage Assessment</h2>
<p><em>Triaged by: signal-triage agent · 2026-10-27 · Status: pending owner decision</em></p>
<p><strong>Problem clarity:</strong> Clear</p>
<blockquote><p>"I can't share data with my finance team because they only work in Excel"</p></blockquote>
<p><strong>Risk scan:</strong></p>
<table>
  <tr><th>Risk</th><th>Level</th><th>Basis</th><th>Needs human assessment</th></tr>
  <tr><td>Value</td><td>Low</td><td>4 other customers, $220k ARR</td><td>No</td></tr>
  <tr><td>Feasibility</td><td>Unknown</td><td>Scope unclear</td><td>Yes: Feasibility assessor</td></tr>
</table>
<p><strong>Recommended outcome:</strong> LIGHT</p>
<p><strong>Reasoning:</strong> …</p>
<p><strong>Override if:</strong> …</p>
```

The `<h2>Triage Assessment</h2>` heading is how later runs detect that an opportunity has been triaged. Keep it exact.
