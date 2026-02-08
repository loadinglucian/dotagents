---
name: prd-writer
description: Write or improve comprehensive Product Requirements Documents (PRDs), product specs, feature briefs, or project requirement docs for full projects or features. Use when turning notes into a PRD, drafting a greenfield product PRD, expanding incomplete PRDs, or standardizing requirements for product + engineering audiences.
---

# PRD Writer

## Overview

Produce a complete, decision-ready PRD in Markdown for product + engineering audiences. Use clear structure, measurable success criteria, explicit assumptions, and concrete dates.

## Workflow

1. Confirm intent and scope, then run a short user interview before drafting.
2. Fill remaining gaps using the intake checklist.
3. Synthesize context, stakeholders, and alternatives before locking requirements.
4. Draft the PRD using the canonical template below.
5. Enforce the quality bar and call out open questions explicitly.

## User Interview (Ask Before Writing)

Ask concise questions to clarify features, user flow, and success. Prefer 5–8 questions max; if time is short, start with the first 4.

- What is the core user journey (happy path) from entry to success?
- Which key features are must-haves for v1, and which can wait?
- What are the most important screens or touchpoints in the flow?
- Where are the biggest risks, uncertainties, or unknowns?
- What outcomes define success (business and user), and by when?
- Are there any hard constraints (tech stack, budget, legal, timeline)?
- What should explicitly be out of scope?
- Who are the primary stakeholders/approvers?

## Intake Checklist (Ask or Infer)

- Define the problem and why it matters now.
- List goals and non-goals.
- Identify target users and primary use cases.
- Capture constraints (technical, legal, timeline, budget).
- Define success metrics and baselines if known.
- Set scope boundaries (in/out).
- Note dependencies and required stakeholders.

If missing information blocks decisions, ask concise questions. If not, proceed with reasonable assumptions and list them.

## Research and Synthesis

- Identify key stakeholders and decision owners.
- Summarize existing alternatives and why they fall short.
- Capture primary user pains and evidence where available.
- Note ecosystem dependencies or platform constraints.

Keep this section short and decision-oriented; avoid deep market essays.

## Canonical PRD Template

Use this structure and keep headings in this order unless the user specifies otherwise.

```markdown
# <Product/Feature Name> — PRD

**Version:** <x.y>
**Date:** <Month Day, Year>
**Status:** Draft | In Review | Approved
**Owner:** <Name/Team>

---

## 1. Problem Statement
<What is broken or missing? Who is impacted? Why now?>

## 2. Goals and Non-Goals
**Goals**
- ...

**Non-Goals**
- ...

## 3. Target Users and Use Cases
- **Primary users:** ...
- **Secondary users:** ...
- **Top use cases:** ...

## 4. User Journey / Flows
- <Step-by-step flow or bullets>
- <Key moments or decision points>

## 5. Requirements
### 5.1 Functional Requirements
- [P0] ...
- [P1] ...

### 5.2 Non-Functional Requirements
- Performance: ...
- Reliability: ...
- Security/Privacy: ...
- Accessibility: ...
- Compliance: ...

## 6. UX / Design Considerations
- ...

## 7. Data, Analytics, and Success Metrics
**Primary KPIs**
- ...

**Instrumentation**
- Events: ...
- Properties: ...

**Baseline and Targets**
- Baseline: ...
- Target: ...

## 8. Dependencies and Assumptions
**Dependencies**
- ...

**Assumptions**
- ...

## 9. Risks and Mitigations
- Risk: ...
  - Mitigation: ...

## 10. Milestones and Rollout
- Milestone 1 (Date): ...
- Milestone 2 (Date): ...
- Rollout plan: ...

## 11. Open Questions
- ...
```

## Quality Bar

- Make requirements testable and unambiguous.
- Tie each major decision to a goal or risk.
- Use explicit dates (for example, "February 4, 2026"), not relative dates.
- Quantify success criteria and include baselines when possible.
- Call out tradeoffs and out-of-scope items clearly.

## Edge Cases and Pitfalls

- Over-scoping: Split into phases if the PRD is too large.
- Vague success metrics: Add measurable targets or propose a discovery task.
- Missing constraints: List assumptions and flag for confirmation.

## Output Example Snippet

```markdown
## 7. Data, Analytics, and Success Metrics
**Primary KPIs**
- Activation rate: 35% → 50% within 90 days of launch
- Time-to-first-success: median 20 minutes → 10 minutes

**Instrumentation**
- Event: `project_created` (properties: source, template_id)
- Event: `first_success` (properties: time_elapsed_seconds)

**Baseline and Targets**
- Baseline: 35% activation rate (January 15, 2026 cohort)
- Target: 50% activation rate by June 1, 2026
```
