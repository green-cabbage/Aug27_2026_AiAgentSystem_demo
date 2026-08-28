````text
Create and execute a production-scale feature-development task using the
workflow defined under `./agent-system/`.

Task ID:

`implement-met-phi-modulation-correction`

## Authorization

This is a full-scale production execution task, not a toy or educational test.

Set the task authorization fields to:

```yaml
execution_granted: true
production_granted: true
````

Ensure that `allowed_commands` includes the gateway option:

```yaml
allowed_commands:
  - "--use_gateway"
```

Also include the concrete command categories required by this task:

```yaml
allowed_commands:
  - "--use_gateway"
  - "pixi run *"
  - "bash stage1_loop_Improved.sh *"
  - "python run_plotter.py"
  - "git status --short"
  - "git diff *"
  - "find *"
  - "ls *"
  - "cat *"
  - "grep *"
```

Interpret these permissions narrowly. They authorize only the repository,
Dask, Stage 1, plotting, inspection, and validation operations required for
this task.

Do not commit, push, rewrite Git history, delete unrelated outputs, or modify
unrelated production configurations.

## Objective

Implement a production-quality MC scale-factor correction that improves the
Data/MC agreement and flatness of the MET phi distribution.

The correction must be integrated into the existing event-processing and
weight-management architecture, primarily through:

```python
from coffea.analysis_tools import Weights
```

in:

```text
src/copperhead_processor.py
```

The correction must apply to MC samples only. Data events must not receive an
MC scale-factor weight.

After implementation, run the requested production-scale Stage 1 workflow,
produce validation plots and text summaries, compare the corrected output
against the existing baseline, and iterate until the Reviewer approves or a
stopping condition is reached.

## Important year clarification

The prose description mentions `2016preVFP`, but the supplied execution
commands explicitly use:

```text
2016postVFP
```

For this task, treat the exact supplied commands as authoritative and run
`2016postVFP`.

Record this resolution in the task assumptions.

Do not silently switch to `2016preVFP`.

## Workflow

Use this role sequence:

1. Code Generator
2. Code Runner
3. Validation Reviewer
4. Code Generator, when changes are requested
5. Code Runner
6. Validation Reviewer
7. Repeat until convergence

A separate Results Agent is not required for this task because the Code Runner
will execute the existing plotter and record its outputs. The Reviewer must
inspect the generated text results and reference the PDF paths for later human
inspection.

Use no more than four implementation-and-validation iterations.

Do not weaken validation criteria merely to obtain convergence.

## Task artifact location

Create task records under:

```text
./agent-system/tasks/implement-met-phi-modulation-correction/
```

Use a separate directory for every iteration:

```text
./agent-system/tasks/implement-met-phi-modulation-correction/
├── task.yaml
├── current-state.yaml
├── assumptions.md
├── iterations/
│   ├── 001/
│   ├── 002/
│   ├── 003/
│   └── 004/
└── final-summary.md
```

Never overwrite evidence from an earlier iteration.

---

# Phase 1: Code Generator

Begin with:

`Role: Code Generator`

## Generator objective

Implement a MET phi scale-factor correction for MC samples using the existing
`coffea.analysis_tools.Weights` infrastructure in:

```text
src/copperhead_processor.py
```

The Data/MC MET phi histogram is currently not sufficiently flat. The
implementation should improve the agreement while preserving the behavior of
other observables and categories.

## Existing starting implementation

A partial minimum working example is available at:

```text
new_features/MET_phi_modulation/starting_code/met_phi_correction_mwe.py
```

Inspect this implementation carefully.

Do not copy it blindly into production code.

Determine:

* which parts are complete;
* which parts are placeholders;
* which assumptions are valid;
* which pieces must be adapted to the repository's production event model;
* how the scale factor should be calculated and represented;
* how nominal and uncertainty variations should be registered in `Weights`.

## Reference materials

Inspect all relevant files under:

```text
new_features/MET_phi_modulation/
```

with particular attention to:

```text
new_features/MET_phi_modulation/ref_material/
```

The pseudocode relevant to this implementation is located on slides 5 and 6
of:

```text
new_features/MET_phi_modulation/ref_material/260202_JMEGeneral_Type1METWithNano_Nurfikri.pdf
```

Read those slides and the surrounding context before deciding how to
implement the correction.

Also inspect any associated code, examples, configuration, text, ROOT, JSON,
YAML, or documentation files in the feature directory.

Use the repository materials as the primary source of truth.

## Missing implementation pieces

The partial implementation is missing or may incompletely handle:

* `Jet_jecL1`;
* `Jet_jecL1L2L3`;
* `CorrT1METJet_jecL1`;
* `CorrT1METJet_jecL1L2L3`;
* the correct construction of the corrected MET components;
* the derivation of the MET phi scale factor;
* uncertainty estimation for the scale factor;
* nominal, Up, and Down weight registration;
* edge cases involving empty or invalid jet collections;
* year-dependent behavior;
* sample-dependent behavior;
* integration with existing Stage 1 systematic handling;
* any additional requirements implied by the reference material.

The list above is not necessarily exhaustive.

The Generator must discover other required pieces through repository and
reference inspection.

## Required technical investigation

Before editing production code, inspect:

1. `src/copperhead_processor.py`;
2. existing uses of `coffea.analysis_tools.Weights`;
3. existing scale-factor implementations;
4. existing nominal/Up/Down weight naming conventions;
5. MET and Type-1 MET construction;
6. NanoAOD branch usage for:

   * jets;
   * corrected Type-1 MET jets;
   * raw and corrected MET;
   * JEC levels;
7. year-resolution and dataset-resolution logic;
8. treatment of data versus MC;
9. Stage 1 output columns used by `run_plotter.py`;
10. existing tests and validation utilities;
11. any existing MET phi correction or JME helper code.

Document the relevant code path before implementing the change.

## Physics and implementation requirements

The implementation must:

* apply only to MC;
* leave data event weights unchanged;
* use the repository's existing `Weights` object;
* follow existing weight naming and variation conventions;
* provide a nominal MET phi scale-factor weight;
* provide meaningful Up and Down uncertainty variations;
* avoid NaN or infinite event weights;
* define behavior for underflow, overflow, and unsupported regions;
* preserve event ordering and array shapes;
* work with awkward arrays and the repository's Dask/Awkward execution model;
* avoid eager conversion of full distributed arrays to NumPy;
* avoid unnecessary event-level Python loops;
* preserve existing nominal weights and systematics;
* avoid double-applying a correction;
* avoid applying the correction to data;
* remain reproducible;
* record any clipping, capping, interpolation, or smoothing behavior.

Do not invent uncertainty values without support.

Derive the uncertainty treatment from, in priority order:

1. supplied reference material;
2. existing uncertainty information in supplied correction inputs;
3. a documented statistical calculation from the Data/MC ratio;
4. a conservative method explicitly justified in the implementation report.

If the correct uncertainty prescription cannot be established, stop and
request human input rather than fabricating one.

## Jet and MET terms

Correctly resolve and use the required JEC quantities, including:

```text
Jet_jecL1
Jet_jecL1L2L3
CorrT1METJet_jecL1
CorrT1METJet_jecL1L2L3
```

Verify their precise definitions, units, shape, availability, and intended use
from the repository and NanoAOD/JME reference material.

Do not assume that similarly named fields are interchangeable.

If a field is not stored directly, determine whether it must be calculated
from available raw factors, correctionlib evaluators, or existing repository
helpers.

Document each formula and its source.

## Scale-factor representation

Determine the appropriate representation from the supplied materials and
existing architecture. This may involve:

* binned MET phi scale factors;
* interpolation;
* Fourier or harmonic parametrization;
* Data/MC ratio correction;
* correctionlib-compatible inputs;
* lookup objects already used by the repository.

Use the smallest production-suitable implementation consistent with the
reference material.

Avoid hard-coded unexplained arrays or constants in
`src/copperhead_processor.py`.

Place reusable correction data or configuration in an appropriate repository
location.

## Required Generator outputs

Before editing, write:

```text
iterations/<NNN>/implementation-plan.md
```

The plan must include:

* current behavior;
* relevant repository files;
* interpretation of slides 5 and 6;
* required MET/JEC formulas;
* proposed scale-factor calculation;
* proposed uncertainty calculation;
* intended `Weights` registration;
* expected files to modify;
* risks;
* validation plan;
* production rerun scope.

After implementation, write:

```text
iterations/<NNN>/
├── generator-report.md
├── changed-files.yaml
├── proposed-commands.yaml
└── implementation-assumptions.yaml
```

The Generator must never claim that execution or validation succeeded.

## Scope boundaries

The Generator is authorized to modify:

* `src/copperhead_processor.py`;
* relevant feature files under
  `new_features/MET_phi_modulation/`;
* narrowly relevant configuration;
* targeted tests;
* validation utilities;
* `run_plotter.py` only where necessary to support task-specific validation.

Avoid unrelated refactoring.

Do not change:

* event selection;
* category definitions;
* luminosities;
* dataset definitions;
* unrelated weights;
* unrelated systematics;
* Combine methodology.

If the implementation would require such a change, stop and request human
approval.

---

# Phase 2: Code Runner

Continue with:

`Role: Code Runner`

## Runner preflight

Before production execution:

1. inspect the Generator report and diff;
2. verify that the changed files remain within scope;
3. run lightweight syntax and import checks;
4. inspect the repository's Pixi configuration;
5. identify the correct production Pixi environment;
6. verify that the required environment supports `--use_gateway`;
7. verify that production execution is authorized;
8. record the current Git commit and dirty-worktree state;
9. check whether the proposed Stage 1 label would overwrite an existing
   output;
10. confirm how the output path will be identified.

Do not execute production commands if basic import or syntax checks fail.

## Dask cluster restart

After the implementation passes preflight, restart the Dask cluster.

Use the connected Dask-control capability or repository-supported cluster
management interface with this instruction:

```text
close existing dask cluster and start a new dask cluster with parameters
defined in dask_cluster_params.txt and scale to 100 workers
```

Use the gateway mode authorized by:

```text
--use_gateway
```

The Code Runner must verify:

* the previous cluster was closed or is no longer being used;
* the new cluster was created using `dask_cluster_params.txt`;
* the scheduler is reachable;
* the cluster scaled to 100 workers or reached the highest valid available
  count;
* worker failures or scaling shortfalls are recorded;
* the cluster belongs to this task.

Record:

* cluster identifier;
* scheduler address, redacted where appropriate;
* requested worker count;
* achieved worker count;
* cluster startup time;
* relevant warnings and errors.

Do not expose credentials or secret tokens in task artifacts.

If the cluster cannot be restarted or cannot reach a viable production worker
count, stop and report the issue.

## Stage 1 label

Create a new, clearly distinguishable Stage 1 label for the corrected output.

It must not overwrite the baseline label:

```text
Run2_NanoV15_forVBFChannel_July06_2026_jetUncRedo
```

Use a label that clearly identifies:

* NanoAODv15;
* Run 2;
* MET phi correction;
* the current implementation or task date/version.

Record the selected label in:

```text
iterations/<NNN>/run-parameters.yaml
```

Use the same corrected label consistently throughout the iteration.

## Stage 1 dataset scope

Run Stage 1 for:

* year: `2016postVFP`;
* MC: DY samples;
* data samples;
* exclude `dy_VBF_filter`.

The task description originally mentions `2016preVFP`, but the supplied
commands specify `2016postVFP`. Use `2016postVFP` unless the user explicitly
changes the instruction.

Before running, inspect how the repository limits samples to DY and data and
how `dy_VBF_filter` is excluded.

Do not assume that `-k` alone performs the requested sample filtering.

Verify the actual selected sample set from configuration, command output, or
the resolved task setup.

If additional task-specific configuration is needed, the Code Generator must
make the smallest transparent configuration change. The Runner must not
silently edit dataset selection logic.

## Stage 1 commands

Run through the repository's production Pixi environment with gateway support.

Use the repository-supported equivalent of:

```bash
bash stage1_loop_Improved.sh \
  -c configs/datasets/dataset_nanoAODv15_run2.yaml \
  -v 15 \
  -l "{stage1_label}" \
  -y "2016postVFP" \
  -m 1 \
  -k

bash stage1_loop_Improved.sh \
  -c configs/datasets/dataset_nanoAODv15_run2.yaml \
  -v 15 \
  -l "{stage1_label}" \
  -y "2016postVFP" \
  -m compact \
  -k
```

Include `--use_gateway` in the actual execution path wherever required by the
repository interface.

Do not append `--use_gateway` blindly to a script that does not accept it.
Inspect the Stage 1 launcher and Pixi task definitions and use the supported
gateway invocation.

Run mode `1` before mode `compact`.

Stop after a failed mode `1`.

Do not run `compact` against incomplete mode `1` outputs.

## Stage 1 output manifest

After successful execution, identify the exact Stage 1 output path produced by
the corrected label.

Do not guess the path.

Verify:

* it exists;
* it belongs to the current label;
* it contains `2016postVFP`;
* it includes the intended DY and data outputs;
* it excludes `dy_VBF_filter`;
* its modification time corresponds to the current iteration;
* it is suitable as input to `run_plotter.py`.

Record the path in:

```text
iterations/<NNN>/output-manifest.yaml
```

Also record:

* selected samples;
* skipped samples;
* failed samples;
* mode `1` output paths;
* compact output paths;
* log paths;
* file counts;
* output sizes;
* Dask execution details.

## Plotter configuration

After Stage 1 completes, inspect `run_plotter.py`.

Update only the parameters required for this validation, including:

* corrected Stage 1 output path;
* baseline Stage 1 output path;
* year;
* relevant samples;
* relevant plot output directory;
* any task-specific label.

The baseline comparison label is:

```text
Run2_NanoV15_forVBFChannel_July06_2026_jetUncRedo
```

Prefer command-line arguments or a task-local configuration file if already
supported.

When direct source editing is required:

1. record the original values;
2. modify only the relevant parameters;
3. include the change in the task diff;
4. do not alter plotting or metric logic unless required by a separate
   Generator repair;
5. ensure the final report identifies the exact configuration used.

## Plotter execution

Run through the correct Pixi environment:

```bash
python run_plotter.py
```

Use the repository-supported Pixi command, for example:

```bash
pixi run python run_plotter.py
```

Do not assume the environment command without inspecting the Pixi setup.

Capture:

* exact command;
* environment;
* working directory;
* start and end times;
* exit code;
* stdout;
* stderr;
* generated PDF paths;
* generated TXT paths;
* file sizes;
* output timestamps.

Verify that the outputs are newly generated for the current iteration.

## Runner artifacts

Write:

```text
iterations/<NNN>/
├── run-report.yaml
├── run-parameters.yaml
├── dask-cluster-report.yaml
├── output-manifest.yaml
├── plotter-output-manifest.yaml
└── logs/
    ├── preflight.log
    ├── dask-cluster-restart.log
    ├── stage1-mode-1.log
    ├── stage1-mode-compact.log
    └── run-plotter.log
```

The plotter output manifest must include:

```yaml
baseline_stage1_label: "Run2_NanoV15_forVBFChannel_July06_2026_jetUncRedo"
corrected_stage1_label: null
corrected_stage1_output: null

pdf_outputs: []
txt_outputs: []

validation_scope:
  year: "2016postVFP"
  categories:
    - "nocat"
    - "ggh"
    - "vbf"
  primary_variable: "MET phi"
```

Do not claim that the correction is successful. Only report execution facts.

---

# Phase 3: Validation Reviewer

Continue with:

`Role: Validation Reviewer`

## Reviewer inputs

Read:

* task specification;
* implementation plan;
* Generator report;
* source diff;
* reference-material interpretation;
* Dask cluster report;
* Stage 1 run report;
* output manifest;
* plotter output manifest;
* generated TXT files;
* generated PDFs when accessible;
* baseline output associated with:
  `Run2_NanoV15_forVBFChannel_July06_2026_jetUncRedo`;
* corrected output from the current iteration.

Do not edit code during review.

## Review area A: implementation correctness

Review whether the implementation:

* applies only to MC;
* leaves data weights unchanged;
* uses `coffea.analysis_tools.Weights` correctly;
* registers nominal, Up, and Down variations consistently;
* uses the intended MET/JEC terms;
* handles:

  * `Jet_jecL1`;
  * `Jet_jecL1L2L3`;
  * `CorrT1METJet_jecL1`;
  * `CorrT1METJet_jecL1L2L3`;
* follows the pseudocode and formulas supported by the references;
* avoids duplicated correction application;
* handles empty collections and invalid values;
* preserves Awkward/Dask compatibility;
* avoids large eager materialization;
* uses a defensible uncertainty prescription;
* preserves unrelated weights and systematics.

Distinguish between:

* confirmed bug;
* likely bug;
* suspicious implementation;
* expected behavior;
* insufficient evidence;
* human judgment required.

## Review area B: execution integrity

Verify:

* the Dask cluster was restarted;
* the production run used the intended gateway mode;
* 100 workers were requested;
* the achieved worker count is recorded;
* mode `1` completed;
* mode `compact` used the new mode `1` output;
* the selected year was `2016postVFP`;
* only DY and data were included;
* `dy_VBF_filter` was excluded;
* the corrected output is not stale;
* the baseline and corrected outputs are distinct;
* the plotter used the intended paths;
* PDFs and TXT files belong to the current iteration.

Do not assess physics results from stale or mismatched outputs.

## Review area C: MET phi flatness

Inspect the generated TXT outputs and the metric definitions used by
`run_plotter.py`.

Compare the corrected output against the baseline:

```text
Run2_NanoV15_forVBFChannel_July06_2026_jetUncRedo
```

Evaluate MET phi Data/MC agreement in the configured Z-peak region for:

* `nocat`;
* `ggh`;
* `vbf`.

The primary acceptance category is:

```text
nocat
```

The `ggh` and `vbf` categories are secondary diagnostics and may have larger
statistical fluctuations.

Do not invent a Z-peak mass range.

Use the Z-peak region defined by the plotting or analysis configuration. If
the region is not defined unambiguously, return
`human_review_required`.

Use the quantitative metrics available in the TXT output. Where sufficient
information exists, evaluate at least:

* Data/MC ratio deviation from 1;
* RMS or spread of the Data/MC ratio;
* chi-square or chi-square per degree of freedom against a flat ratio;
* maximum absolute deviation;
* fitted slope, Fourier amplitude, or other modulation metric when provided;
* statistical uncertainties;
* number of populated bins.

The corrected output should show a clear improvement for `nocat`.

A conclusion of improvement must account for uncertainties and cannot be
based only on one visually favorable bin.

For each category, report:

* baseline metric;
* corrected metric;
* absolute change;
* relative change;
* statistical interpretation;
* decision.

If the plotter TXT does not provide sufficient quantitative information, use
the underlying histograms or request a targeted validation addition from the
Code Generator.

## Review area D: regressions in other variables

Inspect the TXT comparisons for all other plotted variables.

Check that the correction does not:

* significantly worsen Data/MC agreement;
* introduce an artificial slope;
* generate oscillatory behavior;
* create isolated bin spikes;
* produce discontinuities;
* distort category-specific distributions;
* change data histograms;
* unexpectedly alter yields;
* create NaN or infinite ratios;
* create large changes outside variables that depend on the new event weight.

Compare baseline and corrected metrics using the statistical uncertainties
reported by the validation workflow.

Do not treat small statistical fluctuations as regressions.

Raise a finding when another variable shows a material and unexplained
degradation.

## PDF plots and human inspection

Record every relevant PDF path in the Reviewer report.

Prioritize the paths for:

1. MET phi, `nocat`;
2. MET phi, `ggh`;
3. MET phi, `vbf`;
4. variables with the largest apparent degradation;
5. plots associated with any blocker or major finding.

The Reviewer may inspect PDFs when supported, but automated approval must be
grounded primarily in reproducible metrics and implementation evidence.

Human inspection remains required before final closure.

## Reviewer findings

Write:

```text
iterations/<NNN>/
├── validation-report.md
├── validation-findings.yaml
├── validation-metrics.yaml
└── human-plot-review.yaml
```

Each finding must resemble:

```yaml
finding_id: "METPHI-VAL-001"
severity: "major"
classification: "confirmed_bug"
status: "open"

title: "Short evidence-based finding title"

affected_files: []
affected_functions: []
affected_outputs: []
affected_categories: []
affected_variables: []

evidence:
  - type: "source_code"
    description: "Concrete evidence"
    source: "file and function or line range"
  - type: "txt_metric"
    description: "Concrete metric comparison"
    source: "TXT output path"
  - type: "plot"
    description: "Relevant visual output"
    source: "PDF path"

alternative_explanations_considered: []

root_cause:
  status: "confirmed"
  description: "Evidence-supported root cause"

impact: >
  Effect on MET phi correction or other distributions.

required_action: >
  Smallest correction required.

required_rerun:
  restart_dask_cluster: true
  stage1_years:
    - "2016postVFP"
  stage1_samples:
    - "DY"
    - "data"
  exclude_samples:
    - "dy_VBF_filter"
  stage1_modes:
    - "1"
    - "compact"
  rerun_plotter: true

required_validation: []
```

## Reviewer decision

Return exactly one:

* `approved`;
* `changes_requested`;
* `human_review_required`;
* `blocked`;
* `inconclusive`.

Use `changes_requested` when:

* MET phi flatness is not improved sufficiently;
* the improvement is limited to an artifact;
* another important variable materially worsens;
* the uncertainty implementation is unsupported;
* the correction is applied incorrectly;
* a reproducible implementation defect exists.

When changes are requested, provide evidence and a minimum rerun scope.

Do not prescribe an arbitrary implementation solely from visual preference.

---

# Repair loop

When the Reviewer returns `changes_requested`, continue with:

`Role: Code Generator`

The Generator must read:

* latest validation report;
* structured findings;
* validation metrics;
* TXT outputs;
* PDF paths;
* current implementation;
* current diff;
* reference materials.

For each finding:

1. independently verify the evidence;
2. identify the smallest responsible code path;
3. determine whether the issue is in:

   * JEC/MET construction;
   * correction calculation;
   * binning or interpolation;
   * uncertainty calculation;
   * `Weights` registration;
   * year or sample routing;
   * plotting configuration;
   * validation logic;
4. propose the smallest safe repair;
5. avoid unrelated changes;
6. update targeted tests;
7. define the required production rerun.

Write:

```text
iterations/<NNN>/
├── repair-plan.md
├── finding-responses.yaml
├── generator-report.md
└── proposed-rerun.yaml
```

For every finding, record one response:

* `fixed`;
* `partially_fixed`;
* `rejected_with_evidence`;
* `unable_to_reproduce`;
* `needs_human_clarification`;
* `deferred_with_approval`.

The Generator must not:

* directly edit output histograms;
* edit TXT metrics to conceal a failure;
* alter data to improve agreement;
* weaken validation criteria;
* tune the correction against random noise without uncertainty support;
* remove problematic bins without justification;
* approve its own change.

After repair, start a new iteration.

The Runner must:

1. rerun lightweight preflight checks;
2. restart the Dask cluster;
3. run the corrected production Stage 1 scope;
4. run compact mode;
5. update the plotter inputs;
6. rerun `run_plotter.py`;
7. preserve all earlier iteration evidence.

---

# Convergence criteria

The task converges only when the Reviewer confirms:

* the implementation is consistent with the reference material;
* the required JEC and corrected-jet quantities are handled correctly;
* the correction applies only to MC;
* nominal, Up, and Down weights are valid;
* the uncertainty method is supported and documented;
* production Stage 1 execution completed;
* intended DY and data samples were processed;
* `dy_VBF_filter` was excluded;
* the corrected output is current and complete;
* MET phi Data/MC flatness improves clearly in `nocat`;
* `ggh` and `vbf` results are acceptable or statistically inconclusive rather
  than materially worse;
* no significant shape artifacts or regressions appear in other variables;
* all blocker and major findings are resolved;
* PDF and TXT output paths are recorded;
* evidence supports approval.

When the Reviewer would otherwise return `approved`, set the state to:

```text
ready_for_human_review
```

Present:

* implementation summary;
* formulas used;
* uncertainty method;
* files changed;
* Stage 1 label and output path;
* Dask cluster details;
* baseline versus corrected MET phi metrics;
* other-variable regression summary;
* remaining warnings;
* all TXT output paths;
* the most important PDF paths.

Do not mark the task closed until human plot inspection is acknowledged.

---

# Production safety boundaries

This task has:

```yaml
execution_granted: true
production_granted: true
```

This authorizes the specified full-scale run, Dask cluster restart, Stage 1
processing, compaction, and plotting.

It does not authorize:

* Git commits or pushes;
* force operations;
* deleting previous output labels;
* overwriting baseline outputs;
* modifying unrelated datasets;
* changing unrelated physics selections;
* launching unrelated production jobs;
* exposing credentials;
* modifying the core agent-system framework.

The orchestration framework under `./agent-system/` may only be modified
through normal task records under:

```text
./agent-system/tasks/implement-met-phi-modulation-correction/
```

If a framework defect blocks execution, stop and propose a separate
framework-development task.

Begin now with:

`Role: Code Generator`

Inspect the repository, partial implementation, and reference materials, then
write the implementation plan before modifying production code.

```
```
