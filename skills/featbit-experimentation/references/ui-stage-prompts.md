---
name: ui-stage-prompts
description: Compact internal protocols that replace the long FeatBit Release Decision UI copy-paste prompts for each stage.
---

# UI Stage Prompts

Use this reference when the user arrives from the FeatBit Release Decision UI with a stage hint. The UI should only need to send `experiment-id`, `stage`, optional `run-id`, and optional unresolved item labels.

Do not recite private UI hints. Use them only to choose the next question.

## Common Rules

- Read the experiment through `featbit_release_decision_get_experiment` before doing stage work.
- If required FeatBit MCP tools are missing, ask the user to register the MCP from the Release Decision page before continuing.
- If a FeatBit MCP call fails because a token is expired or revoked, ask the user to create a new MCP token and restart or resume the agent with the new setup command.
- Act like a release-decision coach, not a batch job.
- Ask one short question at a time when information is missing.
- Before creating a feature flag, updating feature flag targeting, creating a flag change request, or toggling a feature flag, summarize the exact operation and get explicit user approval. Pass `confirmedByUser: true` only after that approval.
- Persist only concrete user-grounded or evidence-grounded fields. Do not write placeholders.
- Advance stage only when the stage completion criteria are satisfied.

## Stage: intent-hypothesis

User-facing task: turn a rough product idea into a clear business goal, concrete reversible change, and falsifiable hypothesis.

Apply CF-01 and CF-02.

If the experiment is blank, start with exactly:

> What are you trying to improve or learn?

As the user answers, extract:

- measurable business goal
- original intent
- specific reversible change
- falsifiable hypothesis
- audience
- expected metric direction
- causal reason
- open constraints or questions

Suggest a concise draft when enough information exists, then confirm before writing through MCP.

Completion criteria:

- Business goal is clear enough to decide what success means.
- Intent separates the outcome from the proposed solution.
- Change is specific and reversible.
- Hypothesis names audience, expected metric direction, and causal reason.
- Open questions or constraints are captured before exposure begins.

Do not ask for variants, rollout percentages, metric event keys, observed data, primary metric contract, guardrails, `metricType`, or `metricAgg`.

Persist through `featbit_release_decision_update_experiment`; keep expected metric direction inside hypothesis text only.

## Stage: exposure

User-facing task: make the change reversible with a FeatBit-managed flag, real flag variations, rollout plan, and rollback conditions.

Apply CF-03 and CF-04.

Use FeatBit flag MCP tools when available. Read existing flags and real variations with `featbit_release_decision_get_feature_flag`; if it returns `ResourceNotFound`, create the missing flag with `featbit_release_decision_create_feature_flag` only after the flag contract is complete and the user approves creation; read the created flag back; update rollout/targeting with `featbit_release_decision_update_feature_flag_targeting` only after the user approves the targeting change. Targeting updates do not enable the flag. If exposure should begin now, ask for approval to enable the flag, call `featbit_release_decision_toggle_feature_flag` with `isEnabled: true`, then read back and verify `isEnabled`.

If direct flag tooling is unavailable, define the concrete flag contract, variants, rollout, toggle state, and rollback rules, then tell the user what to create or bind in the FeatBit UI.

Completion criteria:

- A FeatBit-managed feature flag is bound to the experiment.
- Variation mapping comes from the actual FeatBit flag, not manual text.
- Audience, rollout percentage, or traffic pool is defined for the run.
- If exposure is meant to be live, the flag has been enabled through MCP and read back as enabled.
- Rollback triggers and protected audiences are explicit.
- The change can be paused or reverted without redeploying.

Do not ask for metric event keys or observed data.

Persist only concrete flag, audience, rollout, toggle, and rollback decisions through MCP. When calling `featbit_release_decision_update_feature_flag_targeting`, pass `confirmedByUser: true`, the latest flag `revision`, and a complete targeting object only after explicit user approval. When turning exposure on or off, use `featbit_release_decision_toggle_feature_flag` with `confirmedByUser: true` and verify with `get_feature_flag` only after explicit user approval. Direct update is the default; use change-request mode only if reviewer ids are known or approval is explicitly required.

Do not mark Exposure satisfied when the experiment only contains a proposed flag key in `constraints`. A flag is bound only after `get_feature_flag` succeeds or `create_feature_flag` succeeds and the flag is read back successfully.

## Stage: measuring

User-facing task: define the metric contract, start or inspect a run, and use FeatBit evaluation plus metric event data as the evidence source.

Apply CF-05 and CF-06 setup/run management.

Define exactly one deciding metric, a small set of guardrails, event instrumentation, and run window. Prefer event names that already exist in FeatBit when MCP can discover them.

Use FeatBit-managed flag evaluation data and FeatBit metric event data as the evidence source. Third-party API evidence is only planned and must not be used for actual analysis yet.

When the primary metric name, event, type, aggregation, expected better direction, or guardrails are confirmed, write them with `featbit_release_decision_update_metrics`.

Completion criteria:

- Exactly one primary success metric exists with an expected better direction.
- Guardrails exist and have clear degradation direction.
- Required event instrumentation is named and mapped to FeatBit metric events.
- Run window, traffic mode, and minimum sample expectation are defined.
- When a run is moved to collection, the bound feature flag is read back as `isEnabled: true`; use `featbit_release_decision_toggle_feature_flag` if it is off.
- Evidence comes from FeatBit flag evaluation data and FeatBit metric events.

Do not ask the user to manually enter variants or observed data.

## Stage: decision

User-facing task: produce the first rollout decision for a selected run.

Apply CF-06 and CF-07. Requires `run-id`; if missing, select the only active/analyzable run or ask which run to decide.

Procedure:

1. Read the experiment.
2. Inspect selected run, current `analysisResult`, observation window, primary metric, guardrails, minimum sample, SRM result, and risk values.
3. If analysis is missing or clearly unusable, call `featbit_release_decision_analyze_run` with `forceFresh=true`, then read the refreshed experiment.
4. Do not replace a usable Bayesian/Bandit `analysisResult` with stats-ready/raw stats. Usable Bayesian analysis includes SRM, sample check, primary metric rows with `p_win`/risk, and guardrails. If refreshed analysis is only raw stats, stop and report analyzer mismatch.
5. Pick exactly one API decision value: `CONTINUE`, `PAUSE`, `ROLLBACK`, or `INCONCLUSIVE`. If reasoning says `ROLLBACK CANDIDATE`, persist `ROLLBACK`.
6. Write `decision`, `decisionSummary`, `decisionReason`, and `status="decided"` to the run.
7. Write `lastAction="Decision: <category>"` to the experiment. Do not move stage to `learning` unless learning capture is explicitly requested.
8. If the user asks to execute the rollout action and feature-flag MCP tools are available, read the latest flag revision, summarize the exact flag operation, get explicit user approval, then call `featbit_release_decision_update_feature_flag_targeting` with `confirmedByUser: true`. Use `featbit_release_decision_toggle_feature_flag` as needed with `confirmedByUser: true`: `isEnabled: true` for launch/expansion when the flag is off, `isEnabled: false` for immediate pause/rollback that should stop exposure. Do not silently mutate the flag as part of analysis.

Guardrail inverse mapping: `increase_bad` means inverse true; `decrease_bad` means inverse false.

`decisionSummary` must start with concrete feature-flag action:

- `CONTINUE`: move treatment to 100% or expand gradually.
- `PAUSE`: hold the current rollout.
- `ROLLBACK`: route users back to control/default.
- `INCONCLUSIVE`: keep observing or fix measurement.

`decisionReason` must cite primary metric, guardrails, SRM/sample health, and rollout risk. Do not mention tools, MCP, analyzer internals, schemas, JSON fields, or whether existing analysis was reused.

After writing through MCP, tell the user which fields were updated so the UI can refresh.

## Stage: learning

User-facing task: frame the release decision, record the learning, and feed the next iteration with evidence instead of memory.

Apply CF-06, CF-07, and CF-08.

If evidence is missing or insufficient, explain the smallest next evidence action instead of forcing a decision.

If a decision exists, capture:

- what changed
- what happened
- why it likely happened
- what should be tried next

Completion criteria:

- Evidence is sufficient or explicitly marked insufficient.
- Decision is one of `CONTINUE`, `PAUSE`, `ROLLBACK CANDIDATE`, or `INCONCLUSIVE`.
- Decision rationale ties back to hypothesis and guardrails.
- Learning records what changed, what happened, why, and what to try next.
- If data is insufficient, the process pauses at evidence sufficiency instead of forcing a decision.

Do not ask the user to manually enter variants or observed data.
