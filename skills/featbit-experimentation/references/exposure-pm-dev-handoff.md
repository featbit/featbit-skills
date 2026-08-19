---
name: PM to Dev Handoff for Reversible Exposure
description: Handoff template and required fields for passing a complete feature flag contract from experiment owner to the team that owns code and flag operations.
---

# PM to Dev Handoff for Reversible Exposure

Use this reference when the current user owns the release decision but does not own the codebase, the feature flag wrapper, or the production flag tooling.

The goal is not to tell engineering which command to run. The goal is to give engineering a complete and unambiguous implementation contract.

## What the handoff must answer

Every handoff should let the implementation team answer these questions without guessing:

1. What business change is being gated?
2. Where in the product or AI flow should the flag decision happen?
3. What is the flag called, and what stable key should be used?
4. What variants must exist, and what does each variant do?
5. Who should see the candidate first, and who must be protected?
6. How should traffic be assigned consistently?
7. What is the initial rollout, and what evidence allows expansion?
8. What signals require pause or rollback?
9. If multiple experiments share this flag, what is the traffic isolation strategy?
10. Who owns implementation, rollout approval, and emergency disable?

## Required fields

Include these fields in the ticket, spec, or handoff message.

### 1. Decision context

- Goal
- Hypothesis
- Primary metric
- Guardrails

### 2. Flag contract

- Flag name: human-readable label for dashboards and reviews
- Flag key: stable, environment-agnostic key used in code and flag tooling
- Flag type: boolean, string, number, or JSON
- Variants: control and candidate values, including exact keys returned by the wrapper or SDK
- Default behavior: what happens if evaluation fails or the flag is unavailable

### 3. Implementation point

- Product surface: page, endpoint, workflow step, or model/tool path affected
- Decision point: exact place where the wrapper or SDK should evaluate the flag
- Expected behavior per variant
- Any dependencies: config, segment data, user attributes, or prerequisite events

### 4. Exposure rules

- Protected audiences: users who must stay on control
- Initial audience: who should see the candidate first
- Dispatch key: the user attribute used for stable assignment, such as `userId` or `organizationId`
- Targeting logic: internal users, beta cohort, region, plan, or custom rules
- Initial rollout percentage

### 5. Expansion and rollback rules

- Expansion checkpoints: what evidence justifies 10% -> 25% -> 50% -> 100%
- Pause conditions: what uncertainty or operational issue blocks expansion
- Rollback triggers: what guardrail breach or incident requires disabling the candidate
- Rollback method owner: who is expected to execute the disable action

### 6. Ownership

- Spec owner
- Dev owner
- Flag operator owner
- Approval owner for expansion
- Monitoring owner during rollout

## Recommended handoff template

```md
# Feature Flag Handoff

## Decision context
- Goal:
- Hypothesis:
- Primary metric:
- Guardrails:

## Change summary
- What is changing:
- Why this needs to be reversible:

## Flag contract
- Flag name:
- Flag key:
- Flag type:
- Variants:
  - control:
  - candidate:
- Default behavior if flag evaluation fails:

## Implementation point
- Product surface:
- Evaluate the flag at:
- Behavior when control is returned:
- Behavior when candidate is returned:
- Existing internal wrapper or integration constraint:

## Exposure plan
- Protected audiences:
- Initial audience:
- Dispatch key:
- Targeting rules:
- Initial rollout:

## Expansion plan
- Move to 25% when:
- Move to 50% when:
- Move to 100% when:

## Multi-experiment coordination (if applicable)
- Number of planned experiments on this flag:
- Experiment sequence: [sequential / mutual exclusion / orthogonal]
- If sequential: predecessor experiment must reach decision before this starts
- If mutual exclusion: traffic partition method and bucket allocation
- Predecessor experiment slug (if any):
- Variants active in this experiment vs reserved for later:

## Pause / rollback plan
- Pause if:
- Roll back immediately if:
- Rollback operator:

## Ownership
- Spec owner:
- Dev owner:
- Flag operator:
- Expansion approver:
- Monitoring owner:
```

## Notes for FeatBit teams using an internal wrapper

If the engineering team wraps FeatBit behind its own abstraction:

- specify the desired flag contract, not raw FeatBit commands
- specify the variant keys that the wrapper should return
- specify the dispatch key and targeting intent the wrapper or flag platform must preserve
- specify whether the flag should be created manually, by pipeline, or by internal tooling

The PM or experiment owner does not need to dictate whether engineering uses FeatBit MCP, REST API, Web UI, or an internal platform.

## Minimal good handoff

At minimum, the handoff is good enough when engineering can implement it without asking:

- what the flag key should be
- where the flag should be evaluated
- what the control and candidate behaviors are
- who gets exposed first
- what condition allows expansion
- what condition forces rollback
