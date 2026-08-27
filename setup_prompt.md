
**Code Generator → Code Runner → Reviewer → feedback to Code Generator**

````text
You are acting as a senior software architect and AI-agent workflow designer.

Create a minimal, vendor-agnostic AI-agent system for this repository.

The system must work with repository-aware coding assistants such as:

- Claude Code
- OpenAI Codex
- Cursor
- Gemini CLI
- GitHub Copilot
- similar future tools

Do not build the system around proprietary commands, subagent APIs, slash
commands, hooks, or model-specific memory features.

The system should be defined through:

- Markdown instruction files
- YAML or JSON task-state files
- simple Python utilities
- repository-backed feedback and execution records
- thin vendor-specific adapter files

The initial system has exactly three logical roles:

1. Code Generator
2. Code Runner
3. Reviewer

The core workflow is:

User request
    ↓
Code Generator
    ↓
Code Runner
    ↓
Reviewer
    ↓
Feedback artifact
    ↓
Code Generator

The loop continues until one of the following occurs:

- the reviewer approves the result;
- the maximum iteration count is reached;
- execution is blocked;
- human approval is required;
- an unrecoverable error occurs;
- the requested task is determined to be unsafe or ambiguous.

The purpose of this task is to create the agent infrastructure only.

Do not modify the repository's scientific or application code unless required
to create a tiny isolated example or test fixture.

# Design principles

The system must be:

- vendor agnostic;
- human readable;
- version controlled;
- auditable;
- deterministic where possible;
- safe by default;
- resumable across sessions;
- portable between Claude Code and Codex;
- easy to extend later;
- minimal rather than overengineered.

The canonical system must live under:

`.agent-system/`

Vendor-specific files such as `CLAUDE.md` and `AGENTS.md` must be thin adapters
only.

They must not contain duplicated workflow definitions, project memory, or
agent logic.

# Required architecture

Create an initial structure similar to:

.agent-system/
├── README.md
├── SYSTEM.md
├── POLICY.md
├── WORKFLOW.md
├── agents/
│   ├── code-generator.md
│   ├── code-runner.md
│   └── reviewer.md
├── skills/
│   ├── inspect-task.md
│   ├── generate-change.md
│   ├── run-validation.md
│   ├── review-result.md
│   ├── prepare-feedback.md
│   └── close-task.md
├── templates/
│   ├── task.yaml
│   ├── implementation-plan.md
│   ├── run-report.yaml
│   ├── review-report.yaml
│   ├── feedback.yaml
│   └── final-summary.md
├── schemas/
│   ├── task.schema.json
│   ├── run-report.schema.json
│   ├── review-report.schema.json
│   └── feedback.schema.json
├── tasks/
│   ├── README.md
│   └── .gitkeep
├── scripts/
│   ├── create_task.py
│   ├── validate_artifacts.py
│   ├── advance_task.py
│   └── summarize_task.py
└── examples/
    └── hello-world-task/

Also create or update:

- `AGENTS.md`
- `CLAUDE.md`

Adapt the exact structure to the repository when appropriate, but preserve the
same separation of responsibilities.

# Canonical source of truth

`.agent-system/SYSTEM.md` must be the canonical entry point.

It should define:

- the three roles;
- the workflow;
- instruction priority;
- artifact locations;
- task states;
- iteration rules;
- safety rules;
- approval rules;
- completion criteria.

Use the following instruction priority:

1. Current user instruction
2. Current repository code and configuration
3. `.agent-system/POLICY.md`
4. `.agent-system/SYSTEM.md`
5. Relevant role definition
6. Relevant skill
7. Current task artifacts
8. Historical task records

If instructions conflict, report the conflict instead of silently choosing.

# Role 1: Code Generator

Create `.agent-system/agents/code-generator.md`.

The Code Generator is responsible for:

- understanding the user request;
- inspecting relevant repository files;
- reading the latest reviewer feedback;
- proposing a minimal implementation plan;
- modifying code only when authorized;
- producing a structured change summary;
- preparing the task for execution.

Its required inputs are:

- user request;
- repository state;
- current task file;
- latest feedback artifact, when present;
- relevant source files;
- relevant project instructions.

Its required outputs are:

- implementation plan;
- files changed;
- assumptions;
- commands recommended for validation;
- expected behavior;
- risks;
- unresolved questions;
- task-state update.

The Code Generator must:

- prefer minimal changes;
- preserve existing conventions;
- distinguish facts from assumptions;
- explain when feedback cannot be applied;
- avoid changing tests merely to make failures disappear;
- avoid weakening assertions or tolerances without justification;
- never fabricate execution results;
- never mark work as validated;
- never approve its own implementation.

The Code Generator must not execute production workloads.

It may propose commands, but execution belongs to the Code Runner.

# Role 2: Code Runner

Create `.agent-system/agents/code-runner.md`.

The Code Runner is responsible for:

- running approved commands;
- capturing outputs;
- capturing exit codes;
- recording environment information;
- collecting generated artifacts;
- identifying timeouts and infrastructure failures;
- producing a structured run report.

The Code Runner must not:

- edit source code;
- change test expectations;
- reinterpret requirements;
- decide whether results are correct;
- fabricate successful execution;
- silently retry with different commands;
- run destructive commands;
- run large production jobs without approval.

The Code Runner should use the smallest meaningful validation first:

1. syntax checks;
2. import or compilation checks;
3. static checks;
4. unit tests;
5. tiny synthetic examples;
6. small real-data or small-input tests;
7. broader integration tests;
8. production-scale execution only with explicit approval.

For every command, record:

- exact command;
- working directory;
- start time;
- end time;
- exit code;
- stdout location or captured summary;
- stderr location or captured summary;
- timeout status;
- generated files;
- environment details;
- whether the result is complete.

The Code Runner must distinguish:

- code failure;
- test failure;
- missing dependency;
- unavailable environment;
- permission failure;
- timeout;
- malformed command;
- infrastructure failure;
- successful execution.

# Role 3: Reviewer

Create `.agent-system/agents/reviewer.md`.

The Reviewer is responsible for:

- comparing the user request with the implementation;
- reviewing source-code changes;
- reading the run report;
- evaluating whether tests are meaningful;
- identifying bugs, regressions, and unsupported claims;
- determining whether acceptance criteria are satisfied;
- producing structured feedback for the Code Generator.

The Reviewer must remain logically independent from the Code Generator.

The Reviewer should evaluate:

- requirement coverage;
- implementation correctness;
- edge cases;
- regression risk;
- test quality;
- result validity;
- reproducibility;
- error handling;
- maintainability;
- security and safety;
- scientific or numerical impact where applicable;
- whether the execution evidence supports the claimed result.

The Reviewer must classify findings as:

- blocker;
- major;
- minor;
- suggestion;
- question;
- accepted.

Each finding must include:

- unique finding ID;
- severity;
- affected file or artifact;
- description;
- evidence;
- required correction;
- suggested validation;
- status.

The Reviewer must produce one of these decisions:

- `approved`
- `changes_requested`
- `human_review_required`
- `blocked`
- `inconclusive`

The Reviewer must not:

- modify source code;
- fabricate failures;
- approve based only on successful exit codes;
- assume a passing test proves correctness;
- silently change acceptance criteria;
- classify unexplained numerical differences as acceptable;
- request broad rewrites when a targeted correction is sufficient.

# Feedback loop

The reviewer must save machine-readable feedback that the Code Generator can
read on the next iteration.

Use a file such as:

`.agent-system/tasks/<task-id>/iterations/<iteration>/feedback.yaml`

The feedback artifact must contain:

- task ID;
- iteration number;
- reviewer decision;
- summary;
- findings;
- required actions;
- optional improvements;
- tests that must be rerun;
- unanswered questions;
- approval requirements;
- reviewer confidence;
- timestamp.

The Code Generator must address every active blocker and major finding.

For each finding, the next iteration must record one of:

- fixed;
- partially fixed;
- rejected with justification;
- unable to reproduce;
- needs human clarification;
- deferred with approval.

Findings must retain stable IDs across iterations.

Do not delete old findings or overwrite historical feedback.

# Task state machine

Use explicit task states.

At minimum, support:

- `created`
- `planning`
- `awaiting_implementation_approval`
- `implementing`
- `ready_to_run`
- `running`
- `ready_for_review`
- `reviewing`
- `changes_requested`
- `awaiting_human_input`
- `blocked`
- `approved`
- `closed`

Define valid transitions in `.agent-system/WORKFLOW.md`.

Example:

created
    → planning
    → awaiting_implementation_approval
    → implementing
    → ready_to_run
    → running
    → ready_for_review
    → reviewing
        → approved
        → changes_requested
        → awaiting_human_input
        → blocked

changes_requested
    → implementing

approved
    → closed

The scripts must reject invalid state transitions.

# Iteration limits

The system must not loop indefinitely.

Default maximum:

- 3 implementation-review iterations.

Allow the task file to override this value.

When the maximum is reached:

- stop automatically;
- preserve all artifacts;
- summarize unresolved findings;
- set the task state to `awaiting_human_input`;
- do not continue editing.

The Reviewer may request an early stop when:

- requirements are ambiguous;
- repeated changes do not resolve the issue;
- the environment cannot validate the implementation;
- a physics, security, legal, or production-impact decision requires a human;
- the requested behavior conflicts with repository policy.

# Task directory format

Each task should have an isolated directory:

.agent-system/tasks/<task-id>/
├── task.yaml
├── plan.md
├── current-state.yaml
├── iterations/
│   ├── 001/
│   │   ├── generator-report.md
│   │   ├── proposed-commands.yaml
│   │   ├── run-report.yaml
│   │   ├── review-report.md
│   │   └── feedback.yaml
│   ├── 002/
│   │   └── ...
│   └── ...
└── final-summary.md

Do not overwrite previous iterations.

The `current-state.yaml` file may point to the current iteration and status,
but the historical artifacts must remain immutable.

# Task schema

The task template should contain at least:

```yaml
schema_version: "1.0"

task_id: example-task
title: Example task
request: >
  User request goes here.

mode: implementation

status: created
current_iteration: 0
max_iterations: 3

scope:
  include: []
  exclude: []

constraints: []

acceptance_criteria: []

risk:
  level: low
  categories: []

approvals:
  implementation_required: true
  implementation_granted: false
  execution_required: false
  execution_granted: false
  production_required: true
  production_granted: false

validation:
  allowed_commands: []
  forbidden_commands: []
  required_checks: []

metadata:
  created_at: null
  created_by: null
  repository: null
  branch: null
  starting_commit: null
````

Do not require `created_by` to name a particular model vendor.

It may contain values such as:

* `human`
* `coding-agent`
* `automation`

# Run report schema

The run report should contain at least:

```yaml
schema_version: "1.0"

task_id: example-task
iteration: 1
status: completed

environment:
  operating_system: null
  python_version: null
  working_directory: null
  git_commit: null
  dirty_worktree: null

commands:
  - command_id: run-001
    command: "python -m pytest tests/test_example.py"
    started_at: null
    finished_at: null
    duration_seconds: null
    exit_code: null
    timed_out: false
    stdout_file: null
    stderr_file: null
    generated_artifacts: []
    classification: null

summary:
  passed: 0
  failed: 0
  blocked: 0
  incomplete: 0

limitations: []
```

# Feedback schema

The feedback schema should support:

```yaml
schema_version: "1.0"

task_id: example-task
iteration: 1
decision: changes_requested
confidence: medium

summary: >
  Short reviewer summary.

findings:
  - finding_id: REV-001
    severity: blocker
    category: correctness
    status: open
    affected_files:
      - src/example.py
    evidence: >
      Explain the evidence.
    description: >
      Explain the issue.
    required_action: >
      Explain what must change.
    required_validation:
      - "Run the targeted unit test."

required_actions: []
optional_improvements: []
unanswered_questions: []
human_approval_required: false
```

# Safety policy

Create `.agent-system/POLICY.md`.

The policy must define:

## Default behavior

* read-only inspection before modification;
* minimal changes;
* no destructive actions;
* no production execution;
* no silent approval;
* no fabricated outputs.

## Commands requiring explicit approval

Include at least:

* recursive deletion;
* overwriting output directories;
* force-push;
* resetting Git history;
* deleting branches;
* modifying remote repositories;
* submitting batch or cloud jobs;
* production database writes;
* modifying credentials or permissions;
* installing system-wide packages;
* long-running or expensive workloads;
* copying large data collections;
* publishing or deploying code.

## Forbidden behavior

Include:

* exposing credentials;
* storing secrets in task artifacts;
* weakening tests solely to obtain a pass;
* deleting failed test evidence;
* rewriting prior iteration records;
* claiming validation that did not occur;
* running commands outside the approved scope.

# Vendor neutrality

The canonical files must not contain instructions such as:

* “spawn a Claude subagent”;
* “use the Codex review command”;
* “call a proprietary task tool”;
* “use Claude memory”;
* “use Codex cloud execution”;
* “invoke slash command X.”

Instead, use capability-neutral language such as:

* “perform an independent review”;
* “execute the approved validation command”;
* “save the result to the task artifact”;
* “read the latest feedback file.”

The system must work even when the active coding assistant cannot actually
spawn separate model instances.

In that case, the active assistant may perform the roles sequentially, but it
must:

* clearly announce the active role;
* use separate artifacts;
* avoid allowing the Code Generator to approve its own work;
* perform a fresh review pass using the Reviewer instructions;
* preserve logical separation between roles.

# Thin platform adapters

Create `CLAUDE.md` and `AGENTS.md`.

They should be short and nearly identical.

Each adapter should:

1. point to `.agent-system/SYSTEM.md`;
2. point to `.agent-system/POLICY.md`;
3. instruct the assistant to read the current task state;
4. explain that role definitions live under `.agent-system/agents/`;
5. explain that skills live under `.agent-system/skills/`;
6. state that `.agent-system/` is authoritative;
7. state that platform-native memory is non-authoritative;
8. avoid duplicating the canonical system.

Example concept:

```markdown
# Repository AI instructions

The canonical vendor-neutral agent system is defined in:

- `.agent-system/SYSTEM.md`
- `.agent-system/POLICY.md`
- `.agent-system/WORKFLOW.md`

Read `.agent-system/SYSTEM.md` first.

Task artifacts live under `.agent-system/tasks/`.

Use the role definitions under `.agent-system/agents/`.

Do not duplicate or replace the canonical instructions in this adapter.
Platform-specific capabilities may be used for execution convenience, but
all durable state and feedback must be written to the canonical task
artifacts.
```

# Utility scripts

Create lightweight Python utilities using only the standard library unless
there is a compelling reason otherwise.

## `create_task.py`

It should:

* create a unique task directory;
* populate `task.yaml`;
* create the first iteration directory when appropriate;
* reject unsafe task IDs;
* avoid overwriting existing tasks;
* optionally record current Git branch and commit;
* print the created task path.

Example interface:

```bash
python .agent-system/scripts/create_task.py \
  --task-id fix-histogram-binning \
  --title "Fix histogram binning" \
  --request-file request.txt
```

## `validate_artifacts.py`

It should:

* validate required files;
* validate YAML or JSON structure;
* check required keys;
* check iteration numbering;
* check finding IDs;
* check valid task states;
* check that previous iteration artifacts remain present;
* report errors clearly;
* exit nonzero on failure.

Avoid requiring PyYAML unless the repository already uses it.

If using YAML without an available parser is impractical, either:

* use JSON for machine-readable artifacts; or
* implement JSON as the canonical format and provide YAML examples only.

Prefer reliability over visual preference.

## `advance_task.py`

It should:

* read the current state;
* validate a requested transition;
* reject invalid transitions;
* increment iterations only at the correct transition;
* preserve history;
* require approval flags where relevant;
* print a concise transition summary.

Example:

```bash
python .agent-system/scripts/advance_task.py \
  --task-id fix-histogram-binning \
  --to ready_to_run
```

## `summarize_task.py`

It should:

* read all iterations;
* summarize changes;
* summarize commands run;
* summarize reviewer decisions;
* list unresolved findings;
* identify the final task state;
* generate or print a final summary.

# Git behavior

The system may inspect Git status, branch, commit, and diff.

It must not automatically:

* commit;
* push;
* rebase;
* merge;
* reset;
* amend;
* force-push.

These require explicit user instruction.

Each iteration should record the relevant Git commit or worktree state where
possible.

# Example task

Create one safe, tiny example under:

`.agent-system/examples/hello-world-task/`

The example should demonstrate:

1. a task request;
2. a Code Generator proposal;
3. a Code Runner report;
4. a Reviewer finding;
5. a second Code Generator response;
6. a passing second execution;
7. reviewer approval;
8. a final summary.

The example should not modify repository application code.

Use a trivial isolated Python function or text-processing example.

The example should demonstrate stable finding IDs and preserved iteration
history.

# Documentation

`.agent-system/README.md` should explain:

* what the system is;
* the three roles;
* how the feedback loop works;
* how to create a task;
* how to run one iteration;
* how to switch between Claude Code and Codex;
* how to resume a task;
* how to stop a loop;
* where artifacts are stored;
* how approvals work;
* why old iterations are immutable;
* how to extend the system later.

Include an example operating sequence such as:

```text
1. Create a task
2. Code Generator writes the plan
3. Human approves implementation if required
4. Code Generator implements the change
5. Code Runner executes approved checks
6. Reviewer reads code and run artifacts
7. Reviewer approves or writes feedback
8. Code Generator reads the feedback
9. Repeat until approved or stopped
10. Generate final summary
```

# Interaction protocol

When acting as the Code Generator, begin responses with:

`Role: Code Generator`

When acting as the Code Runner, begin responses with:

`Role: Code Runner`

When acting as the Reviewer, begin responses with:

`Role: Reviewer`

Do not claim that separate model processes exist unless they actually do.

Logical role separation is sufficient for the initial implementation.

# Implementation procedure

Perform the following:

1. Inspect the repository in read-only mode.
2. Identify existing:

   * `CLAUDE.md`;
   * `AGENTS.md`;
   * agent directories;
   * workflow files;
   * scripts;
   * tests;
   * project conventions.
3. Present a concise repository map.
4. Identify conflicts with the proposed architecture.
5. Present the exact file tree you propose creating.
6. Identify files that would be modified.
7. Explain any repository-specific adaptations.
8. Wait for approval before writing files.

After approval:

1. create the minimal system;
2. preserve useful existing instructions;
3. avoid modifying application or scientific code;
4. run lightweight validation only;
5. validate the example task;
6. run Python compilation checks;
7. run utility-script help commands;
8. test valid and invalid state transitions;
9. test maximum-iteration handling;
10. report all files created and modified.

Do not install packages automatically.

Do not run production workflows.

# Completion report

Finish with:

* files created;
* files modified;
* files intentionally not changed;
* validation commands executed;
* validation results;
* unresolved limitations;
* how to start the first real task;
* how to run the same task with Claude Code;
* how to continue it with Codex;
* recommended next extension.

The initial implementation should remain small enough to understand in one
sitting.

Do not add a vector database, web dashboard, message queue, container
orchestration, external database, or model API integration.

Build the smallest reliable filesystem-based implementation first.

````

A useful first real task after the setup is created would be:

```text
Create a new task for reviewing
`scripts/plot_stage2_variation_pdfs.py`.

Use read-only mode.

The Code Generator should inspect the script and define expected behavior and
a validation plan, but must not edit it.

The Code Runner should run only safe syntax, import, and help-level checks
that do not require large datasets.

The Reviewer should evaluate the code and run evidence, classify findings,
and write feedback.

Stop after the first review iteration.
````

This lets you test the complete loop on a real repository file without allowing the system to immediately rewrite your analysis code.
