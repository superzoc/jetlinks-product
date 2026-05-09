---
name: jetlinks-product
description: Use when the user starts a new feature, scopes a complex multi-stage delivery, onboards to a JetLinks-style multi-module repo (cloud.jetlinks, runtime-ui, manager modules), or asks the agent to "follow the design flow / propose before implementing / build a task package". Provides a five-rule collaboration philosophy (propose alternatives, wait for sign-off, stage by stage, never silently revert, never fabricate metrics), a four-layer doc taxonomy (entry / constitution / requirements / tasks), simple-vs-complex task package decision and templates, a high-fidelity HTML mockup sinking method that ties into the task package, and a must-read index pattern that resolves which long-term docs to read for the incoming task.
---

# JetLinks Design Flow

A reusable collaboration protocol for product design and engineering delivery in JetLinks-style multi-module repositories.

## When to invoke this skill

Invoke when any of the following is true:

- The user is starting a new feature or module ("design X", "build Y", "let's plan Z").
- The user is onboarding an agent to a repo that already uses a `CLAUDE.md` / `AGENTS.md` / `GEMINI.md` + `docs/constitution/` + `docs/requirements/` + `docs/tasks/` layout.
- The user explicitly asks to "follow the design flow", "propose before implementing", "build a task package", or "stage by stage".
- The work involves multiple collaboration stages (requirements → UI review → implementation → verification → handoff) or multi-model handoff.
- The user requests high-fidelity HTML mockups bound to a delivery plan.

Do not invoke for trivial single-file edits, single-fact lookups, or when the user has already laid out a different protocol that conflicts with this one.

## Hard rules (read these first)

Five collaboration rules apply to every interaction once this skill is active. Full text in `references/collaboration-philosophy.md` — load it before deviating from any rule.

1. **Propose at least two concrete alternatives** with tradeoffs whenever the work touches new functionality, IA changes, or new interaction patterns. Mark a recommendation. Wait for sign-off.
2. **Discussion ≠ permission to implement.** Until the user signs off, stay in proposal mode.
3. **Stage by stage.** Never bundle IA, visual rework, data changes, and doc edits into one big step.
4. **Never silently revert** the user's edits. Surface decision points proactively.
5. **Never fabricate metrics.** No invented accuracy, recall, latency, savings, etc. Use placeholders or omit.

## Four-layer document taxonomy

Every project an agent works in via this skill should follow (or be guided toward) this layout. Full mapping in `references/document-layering.md`.

| Layer | Path | Owns | Forbidden content |
|---|---|---|---|
| Entry | `CLAUDE.md` / `AGENTS.md` / `GEMINI.md` | Collab rules, must-read index, engineering quick-ref | Business rules |
| Long-term rules | `docs/constitution/` | Architecture, engineering, UI tokens, ADRs | Per-feature business design |
| Business blueprints | `docs/requirements/` | Per-feature specs, fields, flows | Coding conventions |
| Per-delivery | `docs/tasks/` | A single delivery's context, plan, verification, handoff | Long-term truth |

Cross-layer rule: stable conclusions migrate **upward** (tasks → requirements / constitution). Long-term rules never live inside a task package.

## Task package decision tree

Resolve the right form for any incoming work. Full protocol in `references/task-package-protocol.md`.

| Scope of change | Form | Template |
|---|---|---|
| 1–2 files, trivial bug, copy tweak, single-fact answer | No task doc — just do it and report back | — |
| 3–5 files, single stage, no IA / product model changes | **Simple task note** at `docs/tasks/YYYY-MM-DD-HHMM-<name>.md` | `assets/simple-task.md` |
| New feature, multi-stage, multi-model handoff, IA / data changes | **Complex task package** at `docs/tasks/YYYY-MM-DD-HHMM-<name>/` | `assets/complex-task-package/` |

**Naming**: `YYYY-MM-DD-HHMM-<name>` (4-digit local 24h time). Lexical sort = creation order.

**Archive**: when a task is verified and stable conclusions are written back to `requirements/` or `constitution/`, immediately `git mv` it to `docs/tasks/_archive/`. Keep the top of `tasks/` to current work only.

## Five collaboration stages (inside a complex task package)

Stages, not roles. Any AI or human can take any stage as long as the deliverable matches.

| Stage | Output | When done |
|---|---|---|
| 1. Requirements clarification | `01-requirements.md` | Goals, non-goals, user paths, edges, open questions are explicit |
| 2. UI review | `02-ui-review.md` | Issue table, recommended layout, copy, accessibility checked |
| 3. Implementation plan | `03-implementation-plan.md` | File list, technical approach, step-by-step, risk control, verification list |
| 4. Verification | `04-verification.md` | Auto checks pass, manual checks pass, residual risks recorded |
| 5. Handoff | `handoff.md` | Current stage, status, decisions, next steps, "do-not-do" list |

`handoff.md` is updated at every stage transition. Anyone picking up the task reads `README.md` + `handoff.md` first.

## High-fidelity mockup sinking method (UI review stage only)

When designing UI inside a task package, sink high-fidelity prototypes as **self-contained HTML files** under `<task>/mockups/`. Full method in `references/mockup-sinking-method.md`.

Layout:

```
<task>/mockups/
├── README.md             # index + recommended walkthrough path
├── _styles.css           # shared design tokens, reset, flow-nav style
├── view-A.html
├── view-B.html
└── ...
```

Each mockup must:

1. Be **self-contained** (open with double-click, only depends on `_styles.css` + a CDN icon font).
2. Include a top **flow-nav** bar with all sibling mockups linked, current one with `class="active"`.
3. Wire **key buttons** (`onclick="location.href='other.html'"`) so a user can walk the entire flow without manual file-picking.

`02-ui-review.md` references the mockups directory and lists the design decisions ratified during review.

## Must-read index pattern

Before touching code on any incoming task, resolve which long-term docs the task hits. Full pattern in `references/must-read-index-pattern.md`.

The convention:

1. **`CLAUDE.md` maintains a "scope → required reads" mapping table** (per-business-domain).
2. On task intake, list the business scopes the task touches.
3. For each scope, read the corresponding `constitution/*.md` + `requirements/*.md` files in full (or the relevant sections if >1200 lines).
4. Only then propose alternatives.

Reading the index is non-negotiable for new features and IA changes. Skip it for trivial fixes (1–2 files).

## Quick start

### Create a complex task package

```bash
TASK="$(date +%Y-%m-%d-%H%M)-my-feature"
mkdir -p "docs/tasks/$TASK"
cp -r "$SKILL_DIR/assets/complex-task-package/." "docs/tasks/$TASK/"
# Then fill README.md → 01-requirements.md → handoff.md (stage = requirements)
```

### Add a new business requirements doc

1. Pick a name following the project's existing naming pattern (e.g. `module-<slug>.md`, `application-<area>.md`).
2. Copy `assets/requirements-doc-header.md` to the top of the new file.
3. Register the file in `CLAUDE.md` "scope → required reads" table so future tasks pick it up.

### Sink mockups into a task package

1. Inside the task dir create `mockups/` with `_styles.css` (design tokens + flow-nav style).
2. Author each view as a single HTML file with the flow-nav header.
3. Wire onclicks across views to make the flow walkable.
4. Reference the directory from `02-ui-review.md`.

## Files in this skill

| Path | Purpose | Load when |
|---|---|---|
| `references/collaboration-philosophy.md` | Full text of the 5 hard rules with examples | Before deviating from any rule |
| `references/document-layering.md` | 4-layer taxonomy details + migration rules | Setting up a new repo or moving content between layers |
| `references/task-package-protocol.md` | Decision tree, naming, archive, handoff conventions | Creating, transitioning, or archiving a task |
| `references/mockup-sinking-method.md` | HTML mockup conventions, flow-nav style, design tokens | Doing UI review stage |
| `references/must-read-index-pattern.md` | How `CLAUDE.md` maintains the index, how Claude resolves it on intake | Onboarding to a new repo or starting any feature task |
| `assets/simple-task.md` | Single-file simple task template | Creating a 3–5 file task |
| `assets/requirements-doc-header.md` | The 4-line frontmatter every requirements doc carries | Authoring a new requirements doc |
| `assets/complex-task-package/` | 6-file complex task package template | Creating a complex task |

## Out of scope

This skill does **not** prescribe:

- Coding conventions (live in `docs/constitution/engineering.md` per project)
- UI tokens / visual rules (live in `docs/constitution/design.md` per project)
- Service layer / data layer architecture (live in `docs/constitution/service-layer.md` per project)
- Specific business rules (live in `docs/requirements/*.md` per project)

Use this skill **alongside** the project's own constitution and requirements — it provides the protocol, not the content.
