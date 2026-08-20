---
name: featbit-experimentation-skills
description: End-to-end FeatBit experimentation and release-decision workflow. Use when shaping product intent, writing a hypothesis, defining feature flags and rollout strategy, designing metrics and instrumentation, creating or analyzing experiment runs, deciding continue/pause/rollback/inconclusive, or capturing learnings for the next iteration.
license: Apache-2.0
metadata:
  author: FeatBit
  version: "1.0.0"
  category: experimentation
---

# FeatBit Experimentation

This skill is the single entry point for the FeatBit release-decision loop:

```
intent -> hypothesis -> implementation -> exposure -> measurement -> interpretation -> decision -> learning -> next intent
```

It replaces the old multi-skill layout (`featbit-release-decision`, `intent-shaping`, `hypothesis-design`, `reversible-exposure-control`, `measurement-design`, `experiment-workspace`, `evidence-analysis`, `learning-capture`, and `project-sync`). Do not call those split skills from this workflow. Treat their former behavior as internal stages of this skill.

## Core Principles

- Do not let an available tool define the user's problem before the decision type is clear.
- Do not implement before the business intent and hypothesis are explicit.
- Do not expose a measurable change before reversibility and rollout rules are defined.
- Do not measure with multiple primary metrics. One metric decides; guardrails protect.
- Do not interpret experiment data before sufficiency, SRM, instrumentation, and window checks pass.
- Do not close a cycle without recording a reusable learning and a next hypothesis.
- Persist every stage transition through the configured FeatBit experimentation MCP server.

## MCP Contract

The FeatBit API database is the canonical source for experiment state. Read it on entry and write it before moving stages.

Required MCP tools:

| Tool | Purpose |
|---|---|
| `featbit_experiment_get_experiment` | Read experiment state, runs, messages, setup mode, and pasted input data |
| `featbit_experiment_update_experiment` | Write goal, intent, hypothesis, constraints, lastAction, lastLearning |
| `featbit_experiment_set_stage` | Set lifecycle stage |
| `featbit_experiment_update_metrics` | Write primary metric and guardrails |
| `featbit_experiment_create_run` | Create an experiment run |
| `featbit_experiment_update_run` | Update run setup, status, decision, and learning fields |
| `featbit_experiment_update_run_traffic` | Configure a run's experiment traffic assignment: analysis method, control/treatment roles, layer id/key, bucket slice, assignment unit, audience filters, allocation plan, and per-variation analysis sampling |
| `featbit_experiment_analyze_run` | Run server-side analysis and persist `inputData` / `analysisResult` |
| `featbit_experiment_list_layers` | List registered release-decision layers for the experiment environment |
| `featbit_experiment_create_layer` | Create a registered layer after explicit user approval |
| `featbit_experiment_update_layer` | Update a registered layer after explicit user approval |
| `featbit_experiment_archive_layer` | Archive a registered layer after explicit user approval |
| `featbit_experiment_get_feature_flag` | Read the real FeatBit flag, revision, variations, and targeting for the experiment environment |
| `featbit_experiment_create_feature_flag` | Create a FeatBit-managed feature flag after the exposure contract is complete and the user explicitly approves |
| `featbit_experiment_update_feature_flag_targeting` | Update flag targeting/rollout directly, or create a change request when `useChangeRequest` or reviewers are provided, after explicit user approval |
| `featbit_experiment_toggle_feature_flag` | Enable or disable the FeatBit flag after targeting is configured, or during pause/rollback execution, after explicit user approval |

The MCP client configuration must provide normal FeatBit auth headers: `Authorization`, `Organization`, and `Workspace`. Do not ask for or pass a per-experiment access token.

For the canonical tool inventory and step-by-step usage rules, read [references/mcp-tool-usage.md](references/mcp-tool-usage.md) before executing MCP-backed experiment work.

Feature flag mutation safety:

- Before calling `featbit_experiment_create_feature_flag`, `featbit_experiment_update_feature_flag_targeting`, or `featbit_experiment_toggle_feature_flag`, summarize the exact flag key, environment implied by the experiment, operation, rollout/toggle state, and rollback consequence, then ask the user for explicit approval.
- Set `confirmedByUser: true` in the MCP request only after the user clearly approves that exact operation. Do not infer approval from earlier setup discussion, a stage transition, or an analysis recommendation.
- If the user has not approved, stop before the MCP mutation and state the proposed operation awaiting approval.

Run traffic configuration safety:

- `featbit_experiment_update_run_traffic` changes how experiment evidence is read; it does not mutate the live feature flag or who sees a variation.
- On a draft run, configure traffic directly after explaining the intended analysis sample.
- Use `featbit_experiment_list_layers` before assigning a run to a layer. If the layer does not exist, create it only after the user explicitly approves.
- Use `sliceStart` and `sliceEnd` for layer bucket ranges such as `30` to `60`; `layerTrafficPercent` is the slice width. Avoid legacy `trafficPercent` / `trafficOffset` unless preserving older run behavior.
- If a run is already `collecting`, `analyzing`, or `decided`, summarize the exact run, control/treatment roles, layer eligibility, and sampling rates, then ask the user to approve before passing `confirmedByUser: true`.
- Choose `analysisSamplingPlan` include rates from the actual exposure distribution in the run window: `includeRate = desired analyzed users for that variation / observed served users for that variation * 100`, capped at `100`. If the live flag rollout changed after data was collected, start a new run window or collect fresh data instead of reusing the old distribution.

Layer mutation safety:

- Creating, updating, or archiving a registered layer can affect future mutual-exclusion assignments. Summarize the layer name, key, assignment unit selector, status, and active run impact before asking for approval.
- Set `confirmedByUser: true` only after approval for that exact layer operation.

Valid values:

| Field | Values |
|---|---|
| `stage` | `intent`, `hypothesis`, `implementing`, `measuring`, `learning` |
| `run.status` | `draft`, `collecting`, `analyzing`, `decided`, `archived` |
| `method` | `bayesian_ab`, `frequentist`, `bandit` |
| `decision` | `CONTINUE`, `PAUSE`, `ROLLBACK`, `INCONCLUSIVE` |
| `metricType` | `binary`, `continuous` |
| `metricAgg` | `once`, `count`, `sum`, `average` |
| primary `expectedDirection` | `increase_good`, `decrease_good` |
| guardrail `direction` | `increase_bad`, `decrease_bad` |

## Entry Protocol

At minimum, pass the experiment id:

```
/featbit-experimentation <experiment-id>
```

The web UI may also pass a stage hint and selected run:

```
/featbit-experimentation <experiment-id> --stage intent-hypothesis
/featbit-experimentation <experiment-id> --stage exposure
/featbit-experimentation <experiment-id> --stage measuring
/featbit-experimentation <experiment-id> --stage decision --run-id <run-id>
/featbit-experimentation <experiment-id> --stage learning
```

Parse `experiment-id`, optional `stage`, optional `run-id`, and optional unresolved UI item labels from the invocation. If `experiment-id` is missing, ask for it before proceeding.

Before asking or saying anything, call:

```python
state = MCP("featbit_experiment_get_experiment", experimentId=experiment_id)
```

If the MCP call fails because the server is unreachable, retry once. If the retry still fails, treat this as a blank new project and proceed without diagnosing the database to the user. If the user later asserts that the database is reachable, call the read tool again before saying otherwise.

After the MCP read resolves `featbitProjectKey`, use product-context MCP capabilities if available. If none are configured, skip product facts silently. Never hardcode local product URLs.

## UI Stage Prompts

When the user comes from the FeatBit Release Decision UI, do not require them to copy a long stage prompt. A short invocation is enough:

```text
Use featbit-experimentation for experiment <experiment-id> at stage <stage>.
```

If a UI stage is explicit, read [references/ui-stage-prompts.md](references/ui-stage-prompts.md) and apply that stage protocol. Treat unresolved UI item labels as private routing hints only; do not recite them to the user.

Supported UI stages:

| UI stage | Internal protocol |
|---|---|
| `intent-hypothesis` | CF-01 + CF-02 |
| `exposure` | CF-03 + CF-04 |
| `measuring` | CF-05 + CF-06 setup/run management |
| `decision` | CF-06 + CF-07 decision for a selected run |
| `learning` | CF-06 + CF-07 + CF-08 |

## First Response Rules

If the project is blank, ask only:

> What are you trying to improve or learn?

Do not enumerate empty fields, announce state loading, or explain the stage.

If `messages` is non-empty, emit no greeting or recap. The user can already see the conversation history. Wait for their next prompt.

If meaningful state exists and `messages` is empty, summarize only the non-empty fields needed for the next decision in at most two short sentences.

If `entryMode == "expert"`, the user already filled the setup wizard. Do not run blank intent, hypothesis, or measurement shaping unless explicitly asked. Acknowledge the configured method, metrics, priors, variants, window, and any `experimentRuns[*].inputData`. If `inputData` exists, route directly to analysis.

## Stage Routing

Infer the stage from state plus the user's message. Multiple signals can apply; choose the earliest stage that blocks a sound decision.

| Lens | Internal stage | Activate when |
|---|---|---|
| CF-01 | `shape_intent` | `goal` is empty/vague, user leads with a tactic, or says "improve", "increase adoption", "make it better" |
| CF-02 | `design_hypothesis` | Goal exists but no falsifiable causal claim ties change, metric, audience, and reason |
| CF-03 / CF-04 | `control_exposure` | Change needs a feature flag, rollout, targeting, protected audience, or traffic strategy |
| CF-05 | `design_measurement` | Hypothesis exists but primary metric, guardrails, or event instrumentation are incomplete |
| CF-05 / CF-06 | `manage_experiment` | Instrumentation is ready and the user wants to start, run, refresh, close, bandit, or holdout an experiment |
| CF-06 / CF-07 | `analyze_evidence` | Data exists and the user asks whether to continue, pause, rollback, or decide |
| CF-08 | `capture_learning` | A decision exists or the user wants to close the cycle and define the next iteration |

## Execution Procedure

```python
def on_session_start(argv, user_message):
    experiment_id = parse_experiment_id(argv)
    ui_stage = parse_stage_hint(argv, user_message)
    run_id = parse_run_id(argv, user_message)
    assert experiment_id, "experiment-id is required"
    state = read_state_with_one_retry(experiment_id)
    if ui_stage:
        stage_protocol = read("references/ui-stage-prompts.md")
        return route_ui_stage(ui_stage, experiment_id, run_id, state, stage_protocol)
    if state.status == "unavailable" or is_blank_project(state):
        ask_user("What are you trying to improve or learn?")
        return

    if state.featbitProjectKey and MCP.has_tool("featbit_product_context_get_facts"):
        _product_facts = MCP("featbit_product_context_get_facts", projectKey=state.featbitProjectKey)

    if state.messages:
        return  # no visible recap

    if state.entryMode == "expert":
        acknowledge_expert_setup(state)
        if any(run.inputData for run in state.experimentRuns):
            manage_experiment(experiment_id, "run analysis")
        return

    say(summarize_nonempty(state, max_sentences=2))
    ask_user("What would you like to work on next?")

def on_user_turn(experiment_id, user_message):
    state = MCP("featbit_experiment_get_experiment", experimentId=experiment_id)
    lens = infer_cf_lens(state, user_message)
    if lens == "CF-01": return shape_intent(experiment_id, user_message, state)
    if lens == "CF-02": return design_hypothesis(experiment_id, user_message, state)
    if lens in ("CF-03", "CF-04"): return control_exposure(experiment_id, user_message, state)
    if lens == "CF-05": return design_measurement(experiment_id, user_message, state)
    if lens in ("CF-05-RUN", "CF-06-RUN"): return manage_experiment(experiment_id, user_message, state)
    if lens in ("CF-06", "CF-07"): return analyze_evidence(experiment_id, user_message, state)
    if lens == "CF-08": return capture_learning(experiment_id, user_message, state)
    answer_directly(user_message, state)
```

## CF-01: Shape Intent

Use when the user has a desire or tactic but no measurable business outcome.

Read [references/intent-goal-extraction-patterns.md](references/intent-goal-extraction-patterns.md).

Actions:

- If the user leads with a solution, ask what outcome that solution should produce.
- Extract the specific behavior or metric, audience, baseline, and iteration scope.
- Ask one question at a time.
- Do not proceed to hypothesis or implementation until the goal is measurable.

Persist:

```python
MCP("featbit_experiment_update_experiment", experimentId=experiment_id, update={
    "goal": goal,
    "intent": original_user_framing,
    "lastAction": "Intent clarified",
})
MCP("featbit_experiment_set_stage", experimentId=experiment_id, stage="intent")
```

When the user asks what next: tell them to mark CF-01 satisfied in the UI, then continue with CF-02 in the same Intent & Hypothesis stage.

## CF-02: Design Hypothesis

Use when a goal exists but the causal claim is missing or vague.

Read [references/hypothesis-template.md](references/hypothesis-template.md).

Template:

> We believe **[change X]** will **[move metric Y in direction Z]** for **[audience A]**, because **[causal reason R]**.

Actions:

- Require change, metric, direction, audience, and causal reason.
- Ask: "Under what conditions would we conclude this hypothesis was wrong?"
- Do not persist `primaryMetric` here. A complete primary metric belongs to CF-05.
- Do not proceed to exposure planning until all five components are present.

Persist:

```python
MCP("featbit_experiment_update_experiment", experimentId=experiment_id, update={
    "hypothesis": hypothesis,
    "change": change,
    "variants": variants,
    "lastAction": "Hypothesis formed",
})
MCP("featbit_experiment_set_stage", experimentId=experiment_id, stage="hypothesis")
```

When the user asks what next: mark Intent & Hypothesis satisfied, move to Exposure, then run CF-03/04.

## CF-03 / CF-04: Control Exposure

Use when a change needs reversibility, flag contract, targeting, rollout, rollback, or traffic allocation rules.

Read:

- [references/exposure-pm-dev-handoff.md](references/exposure-pm-dev-handoff.md)
- [references/exposure-mcp-lifecycle.md](references/exposure-mcp-lifecycle.md)
- [references/exposure-rollout-patterns.md](references/exposure-rollout-patterns.md)
- [references/exposure-multi-experiment-traffic.md](references/exposure-multi-experiment-traffic.md)
- [references/exposure-tool-featbit-webui.md](references/exposure-tool-featbit-webui.md) only when the user needs concrete FeatBit UI operation guidance.

Default output is a concrete feature flag and exposure contract: flag name, stable key, type, variants, evaluation point, variant behavior, target audience, protected users, initial rollout, expansion checkpoints, stop conditions, rollback triggers, and metric-event requirements.

Rules:

- Confirm a hypothesis exists before flag work.
- Do not ask who should create the flag; define the contract and the UI/automation next step.
- Treat FeatBit as the source of truth for flag binding. Experiment `constraints` may propose a flag key, but the flag is not bound until `featbit_experiment_get_feature_flag` succeeds or `featbit_experiment_create_feature_flag` succeeds and is read back.
- Do not create or mutate a flag until the required contract fields are known: flag name, key, type, variants, default/off variation, target audience, protected audience, rollout split, rollback trigger, and dispatch key when rollout is percentage-based.
- If feature-flag MCP tools are available, follow [references/exposure-mcp-lifecycle.md](references/exposure-mcp-lifecycle.md): read existing flags with `featbit_experiment_get_feature_flag`; if the tool returns `ResourceNotFound`, create the missing flag with `featbit_experiment_create_feature_flag`; read the created flag back; configure rollout/targeting with `featbit_experiment_update_feature_flag_targeting`; then call `featbit_experiment_toggle_feature_flag` only when exposure should begin now or a run is moving to `collecting`.
- Targeting updates do not enable a flag. If the user asks to start exposure, launch, collect, expand, pause, or rollback and the toggle tool is available, operate the flag toggle through `featbit_experiment_toggle_feature_flag`; do not send the user to the FeatBit UI only to switch the flag on or off.
- Create, targeting update, and toggle are production-impacting mutations. Ask for explicit user approval immediately before each mutation and pass `confirmedByUser: true` only for the approved call.
- Direct targeting update is the default MCP path. Use change-request mode only when the user supplies reviewer ids, explicitly asks for approval/change-request mode, or the local operating policy requires it.
- If a required flag field is missing or invalid, ask for that field before calling the flag MCP tool. Do not invent reviewer ids, user segments, or production targeting rules.
- Do not set Exposure as satisfied, do not say the flag is bound, and do not advance from CF-03/04 when `get_feature_flag` still returns `ResourceNotFound`.
- Never start at 100% unless protected-audience targeting is explicit.
- For multiple experiments on the same flag or surface, classify as sequential, mutual-exclusion, or orthogonal. Sample size must use the actual traffic slice.

Persist:

```python
if MCP.has_tool("featbit_experiment_get_feature_flag"):
    flag = MCP("featbit_experiment_get_feature_flag", experimentId=experiment_id, key=flag_key)
    if flag.error == "ResourceNotFound":
        require_explicit_user_approval("create feature flag", flag_contract)
        flag_contract["confirmedByUser"] = True
        MCP("featbit_experiment_create_feature_flag", experimentId=experiment_id, request=flag_contract)
        flag = MCP("featbit_experiment_get_feature_flag", experimentId=experiment_id, key=flag_key)
    assert not flag.error, "feature flag must exist before Exposure can be satisfied"
    require_explicit_user_approval("update feature flag targeting", targeting)
    MCP("featbit_experiment_update_feature_flag_targeting", experimentId=experiment_id, key=flag.key, request={
        "confirmedByUser": True,
        "revision": flag.revision,
        "targeting": targeting,
        "comment": "Initial experiment rollout",
        # Optional only when approval is required:
        # "useChangeRequest": True,
        # "reviewers": reviewer_ids,
        # "reason": "Initial experiment rollout"
    })
    if exposure_should_start_now and MCP.has_tool("featbit_experiment_toggle_feature_flag"):
        require_explicit_user_approval("enable feature flag", {"key": flag.key, "isEnabled": True})
        MCP("featbit_experiment_toggle_feature_flag", experimentId=experiment_id, key=flag.key, request={
            "confirmedByUser": True,
            "isEnabled": True,
            "comment": "Enable flag for initial experiment exposure",
        })
        flag = MCP("featbit_experiment_get_feature_flag", experimentId=experiment_id, key=flag.key)
        assert flag.isEnabled, "feature flag must be enabled before collecting exposure data"
MCP("featbit_experiment_update_experiment", experimentId=experiment_id, update={
    "constraints": flag_contract_and_rollout_with_actual_flag_key_and_variations,
    "lastAction": "Exposure contract defined",
})
MCP("featbit_experiment_set_stage", experimentId=experiment_id, stage="implementing")
```

When the user asks what next: mark Exposure satisfied, create/configure the flag through FeatBit MCP when available, otherwise use the FeatBit UI using the contract, then continue to Measuring / CF-05.

## CF-05: Design Measurement

Use when the hypothesis exists but metrics, guardrails, or instrumentation are incomplete.

Read:

- [references/measurement-event-schema-design.md](references/measurement-event-schema-design.md)
- [references/measurement-tool-featbit-sdk.md](references/measurement-tool-featbit-sdk.md)
- [references/exposure-multi-experiment-traffic.md](references/exposure-multi-experiment-traffic.md) when traffic is split across experiments.

Rules:

- One primary metric only. Everything else is a guardrail or diagnostic.
- `metricName`, `metricEvent`, `metricType`, `metricAgg`, and `expectedDirection` are required.
- Primary `expectedDirection` is `increase_good` or `decrease_good`.
- Guardrail `direction` is `increase_bad` or `decrease_bad`.
- Keep guardrails to 2-3.
- Do not advance to measuring until instrumentation is confirmed.

Persist metrics through `featbit_experiment_update_metrics`, not `update_experiment`:

```python
MCP("featbit_experiment_update_metrics", experimentId=experiment_id, update={
    "metricName": primary_metric.name,
    "metricEvent": primary_metric.event,
    "metricType": primary_metric.metric_type,
    "metricAgg": primary_metric.metric_agg,
    "expectedDirection": primary_metric.expected_direction,
    "metricDescription": primary_metric.rationale,
    "guardrails": guardrails_json,
})
MCP("featbit_experiment_update_experiment", experimentId=experiment_id, update={
    "lastAction": "Metrics defined",
})
MCP("featbit_experiment_set_stage", experimentId=experiment_id, stage="measuring")
```

When the user asks what next: mark Measuring satisfied only after event contract and instrumentation are ready, then continue to run setup or analysis.

## CF-05 / CF-06: Manage Experiment

Use when instrumentation is ready and the user wants to create, collect, analyze, refresh, close, run a bandit, or plan holdout.

Read:

- [references/workspace-experiment-folder-spec.md](references/workspace-experiment-folder-spec.md)
- [references/workspace-data-source-guide.md](references/workspace-data-source-guide.md)
- [references/workspace-analysis-bayesian.md](references/workspace-analysis-bayesian.md)
- [references/workspace-analysis-bayesian-usage-patterns.md](references/workspace-analysis-bayesian-usage-patterns.md)
- [references/workspace-analysis-bayesian-decision-guide.md](references/workspace-analysis-bayesian-decision-guide.md)
- [references/workspace-analysis-bandit.md](references/workspace-analysis-bandit.md) for bandit mode
- [references/workspace-analysis-holdout.md](references/workspace-analysis-holdout.md) for long-term holdouts

Rules:

- Redirect upstream if hypothesis or primary metric is missing.
- If feature-flag MCP tools are available, read the bound flag before run setup and use the actual variation values for `controlVariant` and `treatmentVariant`; do not rely on manual text when the API can provide the source of truth.
- Configure experiment traffic through `featbit_experiment_update_run_traffic`, not the generic run update tool. The feature flag's actual served variation is the source of truth; layer eligibility only decides whether an exposure can enter this run, and analysis sampling is applied inside each served variation.
- For gradual rollouts, do not force a 50/50 flag split just to make analysis equal. Use the observed served-variation counts in the run window to choose per-variation include rates. If the flag split has changed, do not assume historical exposure events match the current flag configuration.
- When starting or resuming collection, verify the bound flag is enabled. If `isEnabled` is false and `featbit_experiment_toggle_feature_flag` is available, enable it, read the flag back, and set `observationStart` no earlier than the enable time. If the toggle tool is missing, stop and ask the user to register the latest MCP tools instead of pretending collection has started.
- Starting collection by enabling a flag still requires explicit user approval. Do not set `confirmedByUser: true` unless the user has approved that toggle in this turn or an immediately preceding approval response.
- Resume an existing run for the same hypothesis instead of creating duplicates.
- The database record is the experiment. No local sync scripts are needed.
- Analysis must run through `featbit_experiment_analyze_run`; do not inline-compute and write `analysisResult`.
- Validate `inputData` after analysis: `k <= n`, variants match, no zero `n`.
- If SRM p < 0.01, stop before evidence interpretation.
- Valid run statuses are only `draft`, `collecting`, `analyzing`, `decided`, `archived`.

Start a run:

```python
state = MCP("featbit_experiment_create_run", experimentId=experiment_id)
run_id = newest_run_id(state)
flag = MCP("featbit_experiment_get_feature_flag", experimentId=experiment_id, key=flag_key)
if not flag.isEnabled:
    require_explicit_user_approval("enable feature flag before collection", {"key": flag_key, "isEnabled": True})
    MCP("featbit_experiment_toggle_feature_flag", experimentId=experiment_id, key=flag_key, request={
        "confirmedByUser": True,
        "isEnabled": True,
        "comment": "Enable flag before starting experiment collection",
    })
    flag = MCP("featbit_experiment_get_feature_flag", experimentId=experiment_id, key=flag_key)
    assert flag.isEnabled, "feature flag must be enabled before the run can collect data"
MCP("featbit_experiment_update_run_traffic", experimentId=experiment_id, runId=run_id, request={
    "method": "bayesian_ab",
    "controlVariant": control,
    "treatmentVariant": treatment,
    "assignmentUnitSelector": "user.keyId",
    "layerKey": layer_key_or_null,
    "layerTrafficPercent": layer_traffic_percent_or_100,
    "analysisSamplingPlan": json.dumps([
        {"variation": control, "role": "control", "includeRate": control_include_rate},
        {"variation": treatment, "role": "treatment", "includeRate": treatment_include_rate},
    ]),
    "audienceFilters": audience_filters_json_or_null,
})
MCP("featbit_experiment_update_run", experimentId=experiment_id, runId=run_id, update={
    "slug": slug,
    "status": "collecting",
    "hypothesis": hypothesis,
    "primaryMetricEvent": primary_metric_event,
    "primaryMetricType": primary_metric_type,
    "primaryMetricAgg": primary_metric_agg,
    "guardrailEvents": guardrail_csv,
    "minimumSample": minimum_sample,
    "priorProper": prior_proper,
    "priorMean": prior_mean,
    "priorStddev": prior_stddev,
    "observationStart": observation_start,
})
MCP("featbit_experiment_update_experiment", experimentId=experiment_id, update={
    "lastAction": f"Created experiment {slug}",
})
MCP("featbit_experiment_set_stage", experimentId=experiment_id, stage="measuring")
```

Run or refresh analysis:

```python
MCP("featbit_experiment_analyze_run", experimentId=experiment_id, runId=run_id, forceFresh=True)
```

Expert mode with `inputData` uses the stored pasted data automatically when live stats are not available. Do not ask the user to paste data again.

## CF-06 / CF-07: Analyze Evidence

Use when data exists and a decision is being considered.

Read:

- [references/evidence-decision-framing-guide.md](references/evidence-decision-framing-guide.md)
- [references/evidence-tool-featbit-abtesting.md](references/evidence-tool-featbit-abtesting.md) when the user asks for FeatBit dashboard interpretation.

Sufficiency checks before deciding:

- Both variants measured over the same time window.
- Sample per variant is at least `minimumSample`.
- Posterior risk is not still wide (`risk[trt]` and `risk[ctrl]` both high).
- No known external contamination.
- Instrumentation is verified for both variants.
- SRM check passes.

Decision categories:

- `CONTINUE`: primary P(win) >= 95%, treatment risk is low, guardrails acceptable.
- `PAUSE`: primary P(win) is 80-95%, possible guardrail harm, SRM/instrumentation concern, or named risk needs investigation.
- `ROLLBACK`: guardrail crosses strong harm threshold or primary P(win) <= 5%.
- `INCONCLUSIVE`: sample is below floor, risk has not converged, or primary remains uncertain after the window.

Respect analyzer output:

- Do not quote metrics that do not exist.
- Do not override `verdict`, `p_harm`, or inverse-handled outputs silently.
- If a guardrail direction looks misconfigured, ask the user to confirm and re-run after configuration changes.
- Persist the decision first. If the user asks the agent to execute the rollout decision and feature-flag MCP tools are available, read the latest flag revision and call `featbit_experiment_update_feature_flag_targeting` to expand, hold, or rollback the rollout. Use `featbit_experiment_toggle_feature_flag` to enable a launch/expansion when the flag is off, or to disable exposure for an immediate pause/rollback. Use change-request mode only when reviewer ids are supplied or approval is required.

Persist:

```python
MCP("featbit_experiment_update_experiment", experimentId=experiment_id, update={
    "lastAction": f"Decision: {category}",
})
MCP("featbit_experiment_update_run", experimentId=experiment_id, runId=run_id, update={
    "status": "decided",
    "decision": category,
    "decisionSummary": summary,
    "decisionReason": reason,
})
```

Stage stays `measuring`; it advances to `learning` only after CF-08.

## CF-08: Capture Learning

Use when a decision exists or the cycle is ending.

Read [references/learning-iteration-synthesis-template.md](references/learning-iteration-synthesis-template.md).

A complete learning contains:

1. What changed
2. What happened
3. Confirmed or refuted
4. Why it likely happened
5. Next hypothesis

Rules:

- INCONCLUSIVE cycles still produce learnings.
- Do not turn learning into a postmortem; it must feed the next iteration.
- Do not set experiment status to `completed`, `finished`, or any invented value.

Persist:

```python
MCP("featbit_experiment_update_experiment", experimentId=experiment_id, update={
    "lastLearning": learning.summary,
    "lastAction": "Learning captured",
})
MCP("featbit_experiment_set_stage", experimentId=experiment_id, stage="learning")
MCP("featbit_experiment_update_run", experimentId=experiment_id, runId=run_id, update={
    "status": "archived",
    "whatChanged": learning.what_changed,
    "whatHappened": learning.what_happened,
    "confirmedOrRefuted": learning.confirmed_or_refuted,
    "whyItHappened": learning.why,
    "nextHypothesis": learning.next_hypothesis,
})
```

Then route the next cycle back to CF-01 / CF-02 inside this same skill.

## Deprecated Project Sync

The old `project-sync -> sync.ts -> web API -> database` bridge is retired. New flows use:

```
featbit-experimentation -> configured FeatBit experimentation MCP server -> FeatBit API -> database
```

Do not call `sync.ts`, do not set `SYNC_API_URL`, and do not require a per-experiment access token in the slash command.
