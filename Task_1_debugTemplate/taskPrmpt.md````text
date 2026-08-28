````text
Create and execute a validation-and-repair task using the workflow defined
under `./agent-system/`.

Task ID:

`run2-v15-jetunc-validation-repair`

## Objective

Run the requested Stage 1 processing, generate Combine datacards and
significance results, validate the resulting samples, yields, and systematic
histograms, identify evidence-supported implementation defects, repair
confirmed defects, and repeat the workflow until validation converges or a
stopping condition is reached.

A defect is known to exist somewhere in the relevant implementation, but the
Reviewer must discover and substantiate it independently.

The statement that a defect exists is not evidence for:

- its location;
- its nature;
- its severity;
- its root cause;
- any particular suspicious output.

The Reviewer must not invent a finding merely because a defect is expected.
Every reported defect must be supported by execution evidence, output
comparisons, configuration, source-code inspection, or a combination of these.

Do not modify code solely because this task states that a defect exists.

## Workflow

Use the following role sequence:

1. Code Runner
2. Validation Reviewer
3. Results Agent
4. Code Generator, only when changes are requested
5. Code Runner
6. Validation Reviewer
7. Results Agent
8. Repeat until convergence or a stopping condition is reached

The first iteration begins with the Code Runner.

Do not run the Code Generator before the initial execution and validation.

Each role must write separate task artifacts.

## Iteration limit

Use at most four validation-and-repair iterations.

The task stops when:

- the Reviewer would approve and the task is ready for human inspection;
- the maximum iteration count is reached;
- execution cannot be completed;
- required inputs or environments are unavailable;
- the evidence remains inconclusive;
- a repair requires scientific judgment;
- a repair exceeds the authorized scope;
- repeated repairs fail to resolve the same finding.

Do not weaken validation criteria merely to obtain convergence.

## Task parameters

Use:

```bash
stage1_label="Run2_NanoV15_forVBFChannel_July06_2026_jetUncRedo"
years="2018,2017,2016postVFP,2016preVFP"
dataset_config="configs/datasets/dataset_nanoAODv15_run2.yaml"
nanoaod_version="15"
````

Create task records under:

```text
./agent-system/tasks/run2-v15-jetunc-validation-repair/
```

Use a separate directory for each iteration:

```text
./agent-system/tasks/run2-v15-jetunc-validation-repair/
├── task.yaml
├── current-state.yaml
├── iterations/
│   ├── 001/
│   ├── 002/
│   ├── 003/
│   └── 004/
└── final-summary.md
```

Never overwrite evidence from a previous iteration.

---

# Phase 1: Code Runner

Begin with:

`Role: Code Runner`

## Initial inspection

Before running commands, inspect the repository and determine:

* the repository root;
* the required working directory;
* the software environment required for Stage 1;
* the correct way to enter the Pixi environment named `combine`;
* whether the requested Stage 1 label already has outputs;
* whether any command may overwrite existing outputs;
* where modes `2`, `2p`, and `3` write their outputs;
* how to identify the Stage 3 output path;
* whether the Combine scripts expect the Stage 3 directory itself or a
  particular child directory;
* where the Combine scripts write datacards, workspaces, logs, and significance
  results;
* how to distinguish outputs from the current iteration from stale outputs.

Do not guess the environment or Stage 3 path.

Do not alter the functional command arguments without approval.

If an existing output would be overwritten, stop and report the conflict
unless repository behavior clearly makes the rerun safe and preserves prior
evidence.

## Stage 1 execution

Run these commands sequentially:

```bash
stage1_label="Run2_NanoV15_forVBFChannel_July06_2026_jetUncRedo"
years="2018,2017,2016postVFP,2016preVFP"

time bash stage1_loop_Improved.sh \
  -c configs/datasets/dataset_nanoAODv15_run2.yaml \
  -v 15 \
  -l "$stage1_label" \
  -y "$years" \
  -m 2 \
  -k

time bash stage1_loop_Improved.sh \
  -c configs/datasets/dataset_nanoAODv15_run2.yaml \
  -v 15 \
  -l "$stage1_label" \
  -y "$years" \
  -m 2p \
  -k

time bash stage1_loop_Improved.sh \
  -c configs/datasets/dataset_nanoAODv15_run2.yaml \
  -v 15 \
  -l "$stage1_label" \
  -y "$years" \
  -m 3 \
  -k
```

Run them in this order:

1. mode `2`;
2. mode `2p`;
3. mode `3`.

Stop after the first failed Stage 1 command unless continuing is demonstrably
safe and useful.

Record any decision to continue after a failure.

## Resolve the Stage 3 output path

After mode `3` completes, identify the exact Stage 3 output path produced by
the current iteration.

Call this resolved path:

```bash
stage3_output_path="..."
```

The Runner must establish the path using evidence such as:

* command output;
* repository path conventions;
* output timestamps;
* expected files;
* directory contents.

Do not use a path merely because it already exists.

Verify that the resolved path:

* exists;
* belongs to the requested `stage1_label`;
* contains outputs for the requested years;
* was produced or updated by the current iteration;
* is suitable as input to `produce_combine_cards.sh`.

Write the resolved path to `output-manifest.yaml` before running the Combine
commands.

If the Stage 3 path cannot be resolved confidently, stop and mark the task
blocked or incomplete.

## Combine environment

Run the datacard and significance commands inside the Pixi environment named:

```text
combine
```

Inspect the repository's Pixi configuration to determine the correct command
form.

Prefer the repository-supported form, such as:

```bash
pixi run -e combine <command>
```

or another documented equivalent.

Do not assume a specific Pixi invocation without checking the repository.

Record:

* the exact Pixi command;
* the selected environment;
* relevant environment information;
* the working directory.

## Produce Combine cards

Using the resolved `stage3_output_path`, run sequentially:

```bash
sh produce_combine_cards.sh "$stage3_output_path" 2018
sh produce_combine_cards.sh "$stage3_output_path" 2017
sh produce_combine_cards.sh "$stage3_output_path" 2016postVFP
sh produce_combine_cards.sh "$stage3_output_path" 2016preVFP
sh produce_combine_cards.sh "$stage3_output_path" Run2
```

Each command must run inside the `combine` Pixi environment.

Run the four individual years before `Run2`, because the Run 2 combination may
depend on the per-year outputs.

Stop after the first failed datacard command unless later commands are known to
be independent and continuing is useful for diagnosis.

Do not run significance for a year or group whose datacard production failed.

## Produce significance results

After successful datacard production, run:

```bash
bash produce_significance.sh "$stage3_output_path" 2018
bash produce_significance.sh "$stage3_output_path" 2017
bash produce_significance.sh "$stage3_output_path" 2016postVFP
bash produce_significance.sh "$stage3_output_path" 2016preVFP
bash produce_significance.sh "$stage3_output_path" Run2
```

Each command must run inside the `combine` Pixi environment.

Run the four individual years before `Run2`.

Capture both prefit and stat-only significance values when the script reports
them.

Do not infer success solely from exit code zero. Verify that the expected
significance output or log was produced and contains parseable results.

## Execution evidence

For every Stage 1, datacard, and significance command, record:

* command ID;
* exact command;
* environment activation or Pixi wrapper;
* working directory;
* Git commit;
* dirty-worktree status;
* start time;
* finish time;
* wall-clock duration;
* exit code;
* stdout path;
* stderr path;
* warnings;
* errors;
* produced output paths;
* output existence checks;
* output modification times;
* output sizes;
* partial or incomplete outputs.

Write at least:

```text
iterations/<NNN>/
├── run-report.yaml
├── output-manifest.yaml
├── significance-results.yaml
└── logs/
    ├── stage1-mode-2.log
    ├── stage1-mode-2p.log
    ├── stage1-mode-3.log
    ├── combine-cards-2018.log
    ├── combine-cards-2017.log
    ├── combine-cards-2016postVFP.log
    ├── combine-cards-2016preVFP.log
    ├── combine-cards-Run2.log
    ├── significance-2018.log
    ├── significance-2017.log
    ├── significance-2016postVFP.log
    ├── significance-2016preVFP.log
    └── significance-Run2.log
```

## Output manifest

Write all discovered outputs to:

```text
iterations/<NNN>/output-manifest.yaml
```

Organize them by:

* Stage 1 mode;
* year or year group;
* sample, when discoverable;
* output type;
* producing command.

Use a structure similar to:

```yaml
schema_version: "1.0"

task_id: "run2-v15-jetunc-validation-repair"
iteration: 1

stage1_label: "Run2_NanoV15_forVBFChannel_July06_2026_jetUncRedo"
stage3_output_path: null

stage1:
  mode_2:
    status: null
    exit_code: null
    log_path: "logs/stage1-mode-2.log"
    output_paths: []

  mode_2p:
    status: null
    exit_code: null
    log_path: "logs/stage1-mode-2p.log"
    output_paths: []

  mode_3:
    status: null
    exit_code: null
    log_path: "logs/stage1-mode-3.log"
    output_paths: []

combine_cards:
  "2018":
    status: null
    log_path: "logs/combine-cards-2018.log"
    datacard_paths: []
    workspace_paths: []
  "2017":
    status: null
    log_path: "logs/combine-cards-2017.log"
    datacard_paths: []
    workspace_paths: []
  "2016postVFP":
    status: null
    log_path: "logs/combine-cards-2016postVFP.log"
    datacard_paths: []
    workspace_paths: []
  "2016preVFP":
    status: null
    log_path: "logs/combine-cards-2016preVFP.log"
    datacard_paths: []
    workspace_paths: []
  "Run2":
    status: null
    log_path: "logs/combine-cards-Run2.log"
    datacard_paths: []
    workspace_paths: []

significance:
  "2018":
    status: null
    log_path: "logs/significance-2018.log"
    output_paths: []
  "2017":
    status: null
    log_path: "logs/significance-2017.log"
    output_paths: []
  "2016postVFP":
    status: null
    log_path: "logs/significance-2016postVFP.log"
    output_paths: []
  "2016preVFP":
    status: null
    log_path: "logs/significance-2016preVFP.log"
    output_paths: []
  "Run2":
    status: null
    log_path: "logs/significance-Run2.log"
    output_paths: []

outputs_by_year:
  "2018": []
  "2017": []
  "2016postVFP": []
  "2016preVFP": []
  "Run2": []

unresolved_outputs: []
```

For every output path:

1. verify that it exists;
2. record whether it is a file or directory;
3. record its modification time;
4. determine whether it belongs to the current iteration;
5. record which command produced it.

Do not merely copy paths printed in logs.

## Significance result artifact

Write parsed values to:

```text
iterations/<NNN>/significance-results.yaml
```

Use a structure similar to:

```yaml
schema_version: "1.0"
iteration: 1

results:
  "2018":
    prefit_significance: null
    stat_only_significance: null
    source_log: "logs/significance-2018.log"
    parse_status: null

  "2017":
    prefit_significance: null
    stat_only_significance: null
    source_log: "logs/significance-2017.log"
    parse_status: null

  "2016postVFP":
    prefit_significance: null
    stat_only_significance: null
    source_log: "logs/significance-2016postVFP.log"
    parse_status: null

  "2016preVFP":
    prefit_significance: null
    stat_only_significance: null
    source_log: "logs/significance-2016preVFP.log"
    parse_status: null

  "Run2":
    prefit_significance: null
    stat_only_significance: null
    source_log: "logs/significance-Run2.log"
    parse_status: null
```

Preserve the raw logs even when parsing succeeds.

---

# Phase 2: Validation Reviewer

Continue with:

`Role: Validation Reviewer`

## Reviewer inputs

Read:

* the task specification;
* the current iteration's `run-report.yaml`;
* `output-manifest.yaml`;
* `significance-results.yaml`;
* all relevant execution logs;
* Stage 1 outputs;
* generated ROOT files;
* generated datacards and workspaces;
* relevant source code;
* relevant dataset and analysis configuration;
* nominal, Up, and Down histograms;
* previous findings and repairs, when applicable.

Do not edit code during the Reviewer phase.

## Independent defect discovery

A defect is known to exist, but the Reviewer must locate and substantiate it
independently.

The Reviewer must not:

* assume that the largest discrepancy is the defect;
* assume that every red flag is a code bug;
* select a source file merely because it appears related;
* report a finding without reproducible evidence;
* reinterpret expected physics differences as bugs without justification;
* fabricate a plausible defect to satisfy the task description.

The fact that this prompt says a defect exists must never appear as evidence.

Distinguish among:

* `confirmed_bug`;
* `likely_bug`;
* `suspicious_behavior`;
* `expected_difference`;
* `configuration_issue`;
* `execution_issue`;
* `insufficient_evidence`;
* `human_judgment_required`.

## Review area A: execution integrity

Verify that:

* all three Stage 1 commands used the requested arguments;
* the Stage 1 modes ran in the intended order;
* all requested years were processed;
* the Stage 3 path was resolved correctly;
* datacard and significance commands used that exact path;
* Combine commands ran in the `combine` Pixi environment;
* all requested per-year and Run 2 commands ran;
* significance was not run against failed or stale datacards;
* outputs belong to the current iteration;
* no stale output was treated as newly generated;
* output manifests are complete;
* no command failure was ignored;
* no output belongs to an unrelated Stage 1 label.

Report execution-integrity problems before drawing conclusions from the
outputs.

## Review area B: MC sample-name consistency

Extract the MC sample names from the authoritative generated output for each
group:

* 2018;
* 2017;
* 2016postVFP;
* 2016preVFP.

Datacard process names may be used when they are the authoritative
representation, but distinguish between:

* raw dataset names;
* internal sample names;
* grouped datacard process names.

Compare like with like.

Report:

* common MC samples;
* missing samples by year;
* extra samples by year;
* duplicated samples;
* likely aliases;
* intentional groupings;
* samples that cannot be compared safely;
* configuration-supported explanations.

Do not require data sample names to match across years.

### Run 2 versus Run 3 comparison

Compare Run 2 and Run 3 MC sample naming only when an authoritative Run 3
reference is available from:

* repository configuration;
* a verified prior Run 3 output;
* an explicit task input.

Do not invent a Run 3 sample list from memory.

If no authoritative Run 3 reference is available, report:

```text
Run 3 sample-name comparison was not performed because no authoritative
Run 3 reference was available.
```

## Review area C: datacard yield scaling

For every comparable MC process or sample, extract the yield represented in
the generated datacards.

Record:

* datacard path;
* process name;
* channel or category;
* year;
* yield;
* whether the value is a raw sample yield or a grouped process yield.

Do not compare a raw sample in one year with a grouped datacard process in
another.

Locate integrated luminosities from authoritative repository configuration or
documentation.

Do not hard-code luminosities from memory.

For each useful year pair, calculate:

```text
yield_ratio = yield_year_A / yield_year_B

lumi_ratio = lumi_year_A / lumi_year_B

relative_ratio_difference =
    yield_ratio / lumi_ratio - 1
```

Report:

* process or sample;
* category or combined yield;
* years compared;
* yields;
* yield ratio;
* luminosities;
* luminosity ratio;
* relative ratio difference;
* diagnostic status;
* explanation or finding reference.

Use these initial diagnostic thresholds:

* absolute difference below 20%: generally consistent;
* 20% to 40%: warning;
* above 40%: red flag.

These thresholds are diagnostic, not proof of correctness.

Account for legitimate year-dependent effects, including:

* trigger efficiency;
* detector conditions;
* correction weights;
* selection efficiency;
* filtering;
* generator differences;
* finite MC statistics;
* category migration;
* sample composition;
* production-version differences.

A discrepancy above threshold must be explained, investigated, or flagged. It
must not be silently accepted.

## Review area D: nominal versus Up/Down histograms

For every comparable combination of:

* year;
* MC sample;
* category or channel;
* variable;
* systematic variation;

identify:

* nominal histogram;
* Up histogram;
* Down histogram.

Validate:

* histogram existence;
* expected naming;
* compatible binning;
* matching bin edges;
* finite contents;
* finite variances;
* noncorrupt ROOT files;
* expected category coverage;
* nonunexpected empty histograms.

Calculate at least:

```text
up_integral_shift =
    (integral_up - integral_nominal) / integral_nominal

down_integral_shift =
    (integral_down - integral_nominal) / integral_nominal
```

Also calculate one or more shape-sensitive metrics:

* maximum binwise fractional shift;
* mean absolute fractional shift;
* weighted L1 distance;
* weighted L2 distance;
* chi-square-like shape difference.

Use a documented policy for nominal bins with zero or near-zero content.

Compare each systematic against peer variations for the same:

* year;
* sample;
* category;
* variable.

Use robust comparison statistics where practical:

* median;
* median absolute deviation;
* robust z-score;
* percentile rank.

Raise a red flag when one variation is substantially farther from nominal than
peer variations, unless an explicit and supported reason explains it.

Also investigate:

* Up and Down moving far in the same direction;
* one side being extreme while the other is nearly nominal;
* Up and Down being unexpectedly identical;
* one variation being identical to nominal across all bins;
* swapped Up and Down labels;
* sign inversions;
* missing variations;
* incorrect year or sample routing;
* unexpected normalization changes;
* NaNs or infinities;
* isolated extreme bins;
* inconsistent behavior across years.

A large systematic effect is not automatically a bug.

## Review area E: significance sanity checks

Review the generated significance values for:

* 2018;
* 2017;
* 2016postVFP;
* 2016preVFP;
* Run2.

Check:

* values are present and finite;
* scripts report successful Combine execution;
* Run 2 inputs correspond to the generated per-year cards;
* stat-only significance is not unexpectedly below the corresponding
  full-systematic or prefit value without explanation;
* Run 2 significance is reasonably related to the per-year results;
* values are not stale copies from an earlier run;
* output paths and logs correspond to the current iteration.

Do not require exact quadrature agreement unless that relationship is valid
for the specific statistical setup.

Flag:

* missing or unparseable significance values;
* identical values copied across distinct years;
* clear inconsistencies between logs and parsed values;
* unexpected zero or nonfinite values;
* evidence that the wrong datacard or workspace was used.

## Source-code inspection

Use observed anomalies to guide inspection of code involving:

* systematic resolution;
* field resolution;
* year-dependent configuration;
* sample selection;
* nominal, Up, and Down mapping;
* histogram naming;
* output routing;
* datacard generation;
* process grouping;
* yield extraction;
* significance input selection.

Do not perform unrelated speculative refactoring.

## Reviewer outputs

Write:

```text
iterations/<NNN>/
├── validation-report.md
├── validation-findings.yaml
├── metrics/
│   ├── sample-name-comparison.yaml
│   ├── datacard-yield-comparison.yaml
│   ├── systematic-shift-metrics.yaml
│   └── significance-review.yaml
└── plots-request.yaml
```

Each finding must resemble:

```yaml
finding_id: "VAL-001"
severity: "blocker"
classification: "confirmed_bug"
status: "open"

title: "Short descriptive title"

affected_files: []
affected_functions: []
affected_outputs: []
affected_years: []
affected_samples: []
affected_categories: []
affected_variables: []
affected_systematics: []

evidence:
  - type: "execution"
    description: "Concrete evidence"
    source: "path or command"
  - type: "output_comparison"
    description: "Concrete evidence"
    source: "path or metric table"
  - type: "source_code"
    description: "Concrete evidence"
    source: "file and line or function"

alternative_explanations_considered:
  - explanation: "Possible alternative"
    disposition: "Why it was accepted or rejected"

root_cause:
  status: "confirmed"
  description: "Evidence-supported root cause"

impact: >
  Describe the effect on outputs, datacards, or significance.

required_action: >
  Describe the smallest correction required.

required_rerun:
  stage1_modes: []
  years: []
  samples: []
  regenerate_combine_cards: []
  rerun_significance: []

required_validation: []
```

For `confirmed_bug`, require:

* reproducible evidence;
* a supported root cause;
* affected source or configuration;
* affected outputs;
* the minimum correction;
* the minimum rerun scope;
* post-repair validation criteria.

When evidence indicates a bug but does not establish root cause, classify it as
`likely_bug`.

Return exactly one decision:

* `approved`;
* `changes_requested`;
* `human_review_required`;
* `blocked`;
* `inconclusive`.

---

# Phase 3: Results Agent

Continue with:

`Role: Results Agent`

Read:

* `output-manifest.yaml`;
* `validation-report.md`;
* `validation-findings.yaml`;
* `plots-request.yaml`;
* systematic metric tables.

Inspect the existing Python nominal/Up/Down plotting script.

Determine its correct invocation from:

* source;
* `--help`;
* documentation;
* existing examples.

Do not guess the arguments.

Generate plots for:

1. every blocker finding;
2. every red-flag finding;
3. every warning finding;
4. representative nonflagged systematics for comparison.

Where supported, include:

* nominal;
* Up;
* Down;
* Up/Nominal;
* Down/Nominal;
* bin-by-bin differences.

Do not:

* modify analysis code;
* alter histogram contents;
* normalize unless explicitly requested;
* rebin unless explicitly requested;
* suppress suspicious plots;
* approve or reject findings.

Save results under:

```text
iterations/<NNN>/results/
```

Write:

```text
iterations/<NNN>/results/
├── plots-manifest.yaml
└── results-summary.md
```

Each plot entry must contain:

* plot path;
* source ROOT file;
* year;
* sample;
* category;
* variable;
* systematic;
* nominal histogram path;
* Up histogram path;
* Down histogram path;
* associated finding ID;
* exact plotting command;
* exit code.

Missing or failed plots must remain visible.

---

# Phase 4: Code Generator

Run the Code Generator only when the Reviewer returns:

`changes_requested`.

Continue with:

`Role: Code Generator`

Read:

* the task specification;
* the latest run report;
* output manifest;
* significance results;
* validation report;
* structured findings;
* metric tables;
* generated plots;
* relevant code and configuration;
* prior repairs.

For each open blocker or major finding:

1. independently verify the finding where practical;
2. identify the smallest responsible code or configuration path;
3. propose the smallest safe repair;
4. preserve unaffected nominal and systematic behavior;
5. avoid unrelated refactoring;
6. add targeted regression checks where practical;
7. define the minimum required rerun scope;
8. never claim validation before the Runner reruns the workflow.

For every finding, record one response:

* `fixed`;
* `partially_fixed`;
* `rejected_with_evidence`;
* `unable_to_reproduce`;
* `needs_human_clarification`;
* `deferred_with_approval`.

Do not:

* edit generated ROOT files;
* edit datacards to conceal a source-code defect;
* directly alter significance output;
* suppress systematic variations;
* weaken thresholds merely to pass review;
* rename outputs to hide inconsistencies;
* broadly refactor unrelated code;
* approve your own changes.

Write:

```text
iterations/<NNN>/
├── repair-plan.md
├── generator-report.md
├── finding-responses.yaml
└── proposed-rerun.yaml
```

The repair plan must include:

* finding IDs;
* evidence reviewed;
* root cause;
* files to change;
* intended repair;
* expected effect;
* risks;
* minimum Stage 1 rerun scope;
* which datacards must be regenerated;
* which significance commands must be rerun;
* required post-repair validation.

## Scientific-change boundary

Stop and request human approval before changing:

* event selection;
* physics weights;
* luminosity values;
* intended systematic definitions;
* intended systematic magnitudes;
* category definitions;
* sample grouping;
* datacard statistical interpretation;
* Combine methodology;

unless authoritative repository evidence makes the intended correction
unambiguous and demonstrates that the current behavior is an implementation
bug.

---

# Rerun and dependency policy

After a repair, start a new iteration.

Rerun the smallest sufficient Stage 1 scope, but respect dependencies.

Possible Stage 1 scopes include:

* mode `3` only;
* modes `2p` and `3`;
* modes `2`, `2p`, and `3`;
* selected years;
* all requested Run 2 years;
* selected samples;
* all samples.

After Stage 1 reruns:

* regenerate datacards for every affected year;
* regenerate the Run 2 datacard when any constituent year is affected;
* rerun significance for every regenerated year;
* rerun Run 2 significance when any constituent year or the Run 2 card is
  affected.

Do not reuse old datacards or significance results for affected outputs.

The Runner must verify that each downstream artifact was regenerated after its
upstream dependency.

Do not overwrite prior iteration evidence.

---

# Convergence criteria

The task converges only when the Reviewer confirms:

* required Stage 1 commands completed;
* required datacard commands completed;
* required significance commands completed;
* outputs belong to the current repaired implementation;
* the Stage 3 output path is correct;
* the output manifest is complete;
* MC sample-name comparisons are understood;
* Run 2 versus Run 3 naming was validated or explicitly marked unavailable;
* cross-year datacard yields roughly follow luminosity scaling or have
  documented explanations;
* no unexplained systematic variation remains an extreme outlier;
* significance results are present and internally consistent;
* all blocker and major findings are resolved;
* repaired outputs and downstream artifacts were regenerated;
* required plots are available for human inspection;
* no unrelated behavior changed;
* evidence supports approval.

Warnings may remain only when:

* they are documented;
* their cause is understood;
* they do not indicate a correctness defect;
* required human follow-up is recorded.

---

# Human inspection

When the Reviewer would otherwise return `approved`, set the task state to:

`ready_for_human_review`

Present:

* the final Reviewer decision;
* the discovered defect;
* evidence and root cause;
* files changed;
* Stage 1 commands rerun;
* datacards regenerated;
* significance commands rerun;
* before-and-after validation metrics;
* before-and-after significance values;
* yield-comparison summary;
* remaining warnings;
* plot manifest;
* paths to the most relevant nominal/Up/Down plots.

Do not mark the task `closed` until human inspection is acknowledged.

---

# Permissions

Authorized:

* run the specified Stage 1 commands;
* run the datacard and significance scripts in the `combine` Pixi environment;
* inspect source code and configuration;
* inspect ROOT files, datacards, workspaces, and logs;
* run lightweight validation and plotting utilities;
* modify source code required to repair evidence-supported defects;
* add targeted tests or validation scripts;
* rerun the minimum required Stage 1 and Combine scope;
* write task artifacts, metrics, manifests, and plots.

Not authorized:

* commit or push;
* rewrite Git history;
* delete prior outputs;
* overwrite prior task evidence;
* edit generated outputs to conceal defects;
* change datasets;
* submit unrelated jobs;
* alter unrelated analysis behavior;
* weaken acceptance criteria without approval;
* modify the core agent-system framework.

## Agent-system protection

Do not modify the orchestration framework under `./agent-system/`, except for
normal task records under:

```text
./agent-system/tasks/run2-v15-jetunc-validation-repair/
```

If a framework defect blocks the workflow:

1. stop;
2. document the defect;
3. propose a separate framework-development task;
4. request explicit approval before modifying the framework.

Begin now with:

`Role: Code Runner`

```
```


NOTE: for pixi env, use the local pixi toml (saved in /work/users/yun79/sideHustle2/copperheadV2)