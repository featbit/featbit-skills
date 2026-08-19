---
name: FeatBit Experimentation MCP Tool Usage
description: Canonical MCP tool inventory and step-by-step best practices for the FeatBit experimentation skill.
---

# FeatBit Experimentation MCP Tool Usage

Use this reference whenever the workflow will read or mutate FeatBit experiment state, feature flags, runs, traffic scope, analysis, decisions, or learning.

## Canonical Tool Inventory

These tools are exposed by the FeatBit API MCP implementation:

| Tool | Use |
|---|---|
| `featbit_experiment_get_experiment` | Read experiment state, runs, activities, setup mode, metrics, and analysis data. |
| `featbit_experiment_update_experiment` | Patch experiment-level fields such as goal, intent, hypothesis, change, constraints, lastAction, and lastLearning. |
| `featbit_experiment_set_stage` | Move the experiment framework stage. Valid stages are `intent`, `hypothesis`, `implementing`, `measuring`, and `learning`. |
| `featbit_experiment_update_metrics` | Persist the primary metric contract and guardrails. Do not write primary metric fields through `update_experiment`. |
| `featbit_experiment_create_run` | Create a run under an existing experiment. |
| `featbit_experiment_update_run` | Patch run metadata, status, observation window, method, metric snapshot fields, decision fields, and learning fields. |
| `featbit_experiment_update_run_traffic` | Configure the Experiment Traffic Assignment panel: method, control/treatment roles, assignment unit, layer id/key, bucket slice start/end, audience filters, allocation plan, and analysis sampling. It does not mutate the live feature flag. |
| `featbit_experiment_analyze_run` | Run server-side analysis and persist refreshed `inputData` and `analysisResult` onto the run. |
| `featbit_experiment_list_layers` | List registered release-decision layers in the experiment environment before assigning a run to a layer. |
| `featbit_experiment_create_layer` | Create a registered layer after explicit user approval. |
| `featbit_experiment_update_layer` | Update a registered layer after explicit user approval. |
| `featbit_experiment_archive_layer` | Archive a registered layer after explicit user approval. |
| `featbit_experiment_get_feature_flag` | Read the bound FeatBit feature flag, revision, enabled state, variations, and targeting from the experiment environment. |
| `featbit_experiment_create_feature_flag` | Create a feature flag in the experiment environment after explicit user approval. |
| `featbit_experiment_update_feature_flag_targeting` | Apply targeting/rollout directly or create a change request after explicit user approval. Requires latest `revision`. |
| `featbit_experiment_toggle_feature_flag` | Enable or disable the feature flag after explicit user approval. Targeting updates do not toggle flags. |

There is no current `featbit_experiment_add_message` MCP tool. Do not call it. Durable notes should be written through `update_experiment.lastAction`, `update_experiment.lastLearning`, or run decision/learning fields as appropriate.

## Global Practices

1. Start every MCP-backed turn with `featbit_experiment_get_experiment`.
2. Treat the FeatBit API database as the source of truth. Do not create local experiment files or sync scripts.
3. Use feature-flag tools only after the experiment resolves an environment through `experimentId`; do not ask the user for `envId`.
4. Ask for explicit approval immediately before any production-impacting flag mutation: create flag, update targeting, create change request, enable, or disable.
5. Set `confirmedByUser: true` only for the exact operation the user approved. Do not carry approval across unrelated mutations.
6. Read the feature flag again after create, targeting update, or toggle when subsequent steps depend on revision, variation ids, or enabled state.
7. Use `update_run_traffic` for evidence scope. Use feature-flag targeting tools for live exposure. Keep those concepts separate.
8. Use layer tools for the layer registry. Use `update_run_traffic` to attach a run to a layer and choose the bucket slice.

## Step Practices

### Intent and Hypothesis

Use:

- `featbit_experiment_get_experiment`
- `featbit_experiment_update_experiment`
- `featbit_experiment_set_stage`

Best practice:

- Persist `goal`, `intent`, `hypothesis`, `change`, `variants`, and `lastAction` only after the user confirms the meaning.
- Do not write metric contract fields here. Metrics belong to the measurement step.

### Exposure

Use:

- `featbit_experiment_get_experiment`
- `featbit_experiment_get_feature_flag`
- `featbit_experiment_create_feature_flag`
- `featbit_experiment_update_feature_flag_targeting`
- `featbit_experiment_toggle_feature_flag`
- `featbit_experiment_update_experiment`
- `featbit_experiment_set_stage`

Best practice:

- Read the target flag first. If it does not exist, create it only after the flag contract is complete and the user approves.
- Use the read-back flag variation ids and revision. Do not rely on manually typed variation ids when the API can provide them.
- Configure targeting before enabling. Enable only when exposure should start now or a collecting run requires active exposure.
- Use change-request mode only when reviewers are supplied, the user asks for it, or local policy requires it.

### Measurement

Use:

- `featbit_experiment_get_experiment`
- `featbit_experiment_update_metrics`
- `featbit_experiment_update_experiment`
- `featbit_experiment_set_stage`

Best practice:

- Persist exactly one primary metric with `metricName`, `metricEvent`, `metricType`, `metricAgg`, and `expectedDirection`.
- Persist guardrails as guardrails, not competing primary metrics.
- Confirm instrumentation event shape before marking measurement ready.

### Run Setup and Collection

Use:

- `featbit_experiment_get_experiment`
- `featbit_experiment_create_run`
- `featbit_experiment_list_layers`
- `featbit_experiment_create_layer`
- `featbit_experiment_update_layer`
- `featbit_experiment_archive_layer`
- `featbit_experiment_get_feature_flag`
- `featbit_experiment_toggle_feature_flag`
- `featbit_experiment_update_run_traffic`
- `featbit_experiment_update_run`
- `featbit_experiment_update_experiment`
- `featbit_experiment_set_stage`

Best practice:

- Reuse an existing run for the same hypothesis/window when appropriate; do not create duplicates by default.
- Read the bound feature flag and use actual served variation values for control and treatment.
- Use `list_layers` before assigning layer eligibility. Create or update a registered layer only after explicit approval and only when the requested layer key/assignment unit does not already exist or is wrong.
- Configure the right-side Experiment Traffic Assignment panel through `update_run_traffic`:
  - Control and treatments: `controlVariant`, `treatmentVariant`, and `analysisSamplingPlan` role entries.
  - Layer eligibility: `layerId` or `layerKey`, `assignmentUnitSelector`, `sliceStart`, `sliceEnd`, and `layerTrafficPercent`.
  - Analysis sampling: `analysisSamplingPlan[].includeRate`.
  - Audience filters: `audienceFilters`.
- Prefer `sliceStart` and `sliceEnd` for non-overlapping layer slices such as `0-30` and `30-60`. Avoid legacy `trafficPercent` and `trafficOffset` except when preserving an old run.
- Do not use layer assignment to decide served variations. The FeatBit flag still decides served variation; the layer only gates which assignment units are eligible for analysis.
- If collection should start and the flag is disabled, ask for approval, toggle it on, read it back, then set `observationStart` no earlier than that enablement.
- Choose `analysisSamplingPlan.includeRate` from observed served variation counts in the run window. Do not force the live flag to 50/50 only to make analysis balanced.
- Use `update_run` for run status, observation window, metric snapshot fields, minimum sample, priors, and user-facing run metadata.

### Analysis and Decision

Use:

- `featbit_experiment_get_experiment`
- `featbit_experiment_analyze_run`
- `featbit_experiment_update_run`
- `featbit_experiment_update_experiment`
- Feature-flag tools only if the user asks to execute the rollout decision.

Best practice:

- Trigger analysis with `forceFresh: true` when data, window, traffic scope, or metric configuration changed.
- After analysis, read the refreshed experiment and inspect run `inputData` and `analysisResult`.
- Do not hand-write `analysisResult`.
- Persist the decision with `update_run` using `status: "decided"` and one of `CONTINUE`, `PAUSE`, `ROLLBACK`, or `INCONCLUSIVE`.
- If executing rollout changes after a decision, read the latest flag revision, summarize the exact change, ask for approval, then call targeting/toggle tools.

### Learning

Use:

- `featbit_experiment_get_experiment`
- `featbit_experiment_update_experiment`
- `featbit_experiment_set_stage`
- `featbit_experiment_update_run`

Best practice:

- Capture what changed, what happened, what was confirmed or refuted, why it likely happened, and the next hypothesis.
- Use `update_experiment.lastLearning` for the reusable summary.
- Archive the run with `update_run.status: "archived"` only after the learning is captured.
