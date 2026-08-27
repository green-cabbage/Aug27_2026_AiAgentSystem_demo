# Aug27_2026_AiAgentSystem_demo

Empty github repo containing `setup_prompt.md`, which has instructions to setup a minimal AI-agent system. To setup the AI-agent system, please give this chat prompt to your Codex/Claude chat:
```
Using `setup_prompt.md` as your guidance, setup ai agent system for me on this repo
```

The output implementation will look something like this:
```
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
├── tests/
│   └── test_agent_system.py
└── examples/
    └── hello-world-task/
        ├── README.md
        ├── example.py
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
        │   └── 002/
        │       ├── generator-report.md
        │       ├── proposed-commands.yaml
        │       ├── run-report.yaml
        │       ├── review-report.md
        │       └── feedback.yaml
        └── final-summary.md

AGENTS.md
CLAUDE.md
```