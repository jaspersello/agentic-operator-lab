# Skill: Lead Qualification SOP

> This is the "company brain" for how an inbound lead gets scored and routed.
> It maps directly to the system prompt in `workflows/hello-agent.json`.

## Ideal Customer Profile (ICP)
- **Role:** marketing, growth, or RevOps leader (Manager level and up).
- **Company size:** 50–1000 employees.
- **Context:** B2B, running paid + content motions, cares about attribution/reporting.

## Tiers
**Tier A — route to an AE now**
- Clear ICP fit **and** at least one buying signal.
- Action: hand to sales with the drafted opener attached.

**Tier B — nurture**
- Partial fit (e.g. right role, unclear company size) or early-stage interest with no urgency.
- Action: add to nurture sequence; revisit in 30 days.

**Tier C — disqualify**
- Poor fit or a hard disqualifier present.
- Action: polite decline / no action; log the reason.

## Buying signals (any one can push toward Tier A)
- Mentions scaling the team or budget.
- Names an attribution, reporting, or measurement pain.
- Frustration with a current tool.
- Explicit demo / pricing / trial request.

## Disqualifiers (any one forces Tier C)
- Student or job seeker.
- Competitor.
- Company under 10 employees.
- Personal email with no company attached.
- Clearly outside B2B marketing.

## Required output (structured)
The agent must return **only** this JSON object:
```json
{
  "tier": "A | B | C",
  "reasoning": "1-2 sentence justification grounded in the lead's actual details",
  "suggested_opener": "2-sentence first-touch message in brand voice"
}
```

## Guardrail note (you build this for real in Week 2)
- The agent **drafts**; a human **approves** before anything is sent. This is the HITL boundary.
- If the lead data is missing or ambiguous, the agent should return Tier B and say what's missing — never guess.
