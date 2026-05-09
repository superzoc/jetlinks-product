# jetlinks-product

A Claude Agent Skill that codifies a "propose-before-implement, stage-by-stage" product design and engineering delivery flow used across JetLinks-style multi-module repositories.

## What this skill provides

When the user starts a new feature, scopes a complex multi-stage delivery, or onboards to a JetLinks-style project, this skill teaches Claude to:

1. Hold a **collaboration philosophy** (propose alternatives before implementing, wait for user sign-off, stage by stage, never silently revert, never fabricate metrics).
2. Apply a **four-layer doc taxonomy** — `CLAUDE.md` (entry) / `docs/constitution/` (long-term rules) / `docs/requirements/` (business blueprints) / `docs/tasks/` (per-delivery context).
3. Choose between **simple task notes** and **complex task packages** with five collaboration stages (requirements / UI review / implementation plan / verification / handoff).
4. Sink high-fidelity **mockups** into the task package as self-contained HTML with a flow-nav header.
5. Resolve a **must-read index** for any incoming task — read the constitution and requirements docs that the task hits before touching code.

## Installation

### With cc-switch (recommended)

```text
Open cc-switch → Skills tab → "Install from GitHub" → paste this repo URL.
```

### Manual

Copy the folder to one of:

- `~/.claude/skills/jetlinks-product/` (user-level, all projects)
- `<project>/.claude/skills/jetlinks-product/` (project-level)
- An Anthropic skills directory recognized by your Claude client.

## Usage

Once installed, just describe the task naturally — Claude will pick this skill up when triggered (see frontmatter description in `SKILL.md`). Common triggers:

- "Start designing module X"
- "Build a complex task package for feature Y"
- "Onboard me to this repo's design flow"
- "Create a high-fidelity mockup that ties into the task"

## Layout

```
jetlinks-product/
├── SKILL.md                              # entry, < 500 lines, links downward
├── references/
│   ├── collaboration-philosophy.md       # the 5 hard rules
│   ├── document-layering.md              # the 4-layer doc taxonomy
│   ├── task-package-protocol.md          # simple vs complex, archive rules
│   ├── mockup-sinking-method.md          # high-fidelity HTML mockups inside tasks
│   └── must-read-index-pattern.md        # how to resolve which long-term docs to read
└── assets/
    ├── simple-task.md                    # template for 3-5 file tasks
    ├── requirements-doc-header.md        # frontmatter template for requirements docs
    └── complex-task-package/             # template directory copied per new task
        ├── README.md
        ├── 01-requirements.md
        ├── 02-ui-review.md
        ├── 03-implementation-plan.md
        ├── 04-verification.md
        └── handoff.md
```

## Origin

Distilled from a real run-through in `jetlinks-ai-app` (a high-fidelity prototype repo) where the flow was used to take a "behavior analysis module" from idea → 9 mockups → complete `module-behavior-analysis.md` requirements doc → bring-up task package. The patterns generalized cleanly to the broader JetLinks family of repos (`cloud.jetlinks`, `runtime-ui`, manager modules).

## Status

v0.1 draft. Verified once on a real module (`mod-behavior-analysis`); not yet packaged for cc-switch installation. See [SKILL.md](./SKILL.md) for the full content.

## License

MIT (or whatever the parent skill repo uses).
