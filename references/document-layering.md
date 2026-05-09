# Document layering

Four layers of long-term content. Each layer owns specific kinds of writing; mixing kinds is the most common drift this skill exists to prevent.

## The four layers

| Layer | Path convention | Owns | Example file |
|---|---|---|---|
| **1. Entry** | `CLAUDE.md`, `AGENTS.md`, `GEMINI.md` (root) | Collaboration rules, must-read index, engineering quick-reference, dev troubleshooting | `CLAUDE.md` |
| **2. Long-term rules** | `docs/constitution/*.md` | Architecture, engineering conventions, UI tokens, service layer rules, ADRs | `docs/constitution/engineering.md` |
| **3. Business blueprints** | `docs/requirements/*.md` | Per-feature specs, fields, flows, IA, page layouts, copy rules | `docs/requirements/module-iot.md` |
| **4. Per-delivery** | `docs/tasks/YYYY-MM-DD-HHMM-*` | A single delivery's context, plan, verification, handoff | `docs/tasks/2026-05-08-1920-mod-X/` |

## What each layer is forbidden to contain

- **Entry**: no business rules, no specific implementation plans. Only protocol + index.
- **Constitution**: no per-feature business design. Only cross-cutting rules.
- **Requirements**: no coding conventions, no UI token definitions. Only "what does this product do".
- **Tasks**: no long-term truth. Only "what we're delivering now". Stable conclusions migrate up.

## Migration rules

Stable knowledge flows **upward**:

```text
tasks → requirements (when a delivery's product decisions stabilize)
tasks → constitution (when a delivery surfaces a cross-cutting rule)
requirements → constitution (rare; only for rules that span all features)
```

Volatile knowledge stays put or flows **downward** to a fresh task:

```text
requirements → new task (when implementing a long-known spec)
constitution → new task (when applying a rule to a specific delivery)
```

## Naming conventions

### Requirements docs

Use the project's existing convention. The most common is **menu-path-based**:

```text
<top-menu>-<side-menu>-<feature>.md

Examples:
  product-model.md
  application-marketplaces.md
  module-iot.md
  module-video.md
  developer-vision-training.md
```

Every requirements doc carries the 4-line frontmatter — see `assets/requirements-doc-header.md`.

### Constitution docs

Short slugs, one file per cross-cutting concern:

```text
docs/constitution/
├── engineering.md        # Stack, layout, store, routing, content, layouts
├── service-layer.md      # API contracts, mock, long task runner, side effects
├── design.md             # Visual, tokens, classes, shell, copy
└── decisions.md          # ADRs (decision IDs, dates, rationale)
```

### Task package directories

```text
docs/tasks/YYYY-MM-DD-HHMM-<task-slug>/
```

- `YYYY-MM-DD`: 4-2-2 digits, creation day
- `HHMM`: 24h local time, 4 digits
- `<task-slug>`: kebab-case, ≤4 words

Lexical sort = creation order. Two parallel sub-tasks at the same time → use `-A` / `-B` in the slug, not on the timestamp.

## How `CLAUDE.md` ties layers together

`CLAUDE.md` carries:

1. The four-layer taxonomy (this file's content, summarized).
2. A **must-read index table** — see `must-read-index-pattern.md`.
3. Engineering quick-reference (1-screen list of "what tech stack, what file lives where").
4. Common dev troubleshooting (the things that bite this repo specifically).

`CLAUDE.md` does **not** carry:

- Long passages of business rules.
- Detailed architecture diagrams (those live in `constitution/`).
- Per-feature specs (those live in `requirements/`).

## Anti-patterns this layering prevents

| Smell | Likely violation | Fix |
|---|---|---|
| `CLAUDE.md` >800 lines and growing | Mixed in business rules | Move business rules to `requirements/`, keep CLAUDE.md as index |
| `requirements/X.md` has TypeScript code blocks defining schemas | Mixed in coding conventions | Move schema rules to `constitution/`, keep field tables in requirements |
| `constitution/design.md` describes how the iot module's IA works | Mixed in business design | Move IA rules to `requirements/module-iot.md` |
| `tasks/2026-04-X/` is referenced 6 months later as the canonical spec | Skipped migration | Migrate stable conclusions back up to `requirements/`, archive task |

## Reading priority when working on a task

1. `CLAUDE.md` (always).
2. `docs/tasks/<current-task>/handoff.md` + `README.md` (always, if a task is in progress).
3. `docs/constitution/*` files matching the change (for engineering / UI / service work).
4. `docs/requirements/*` files matching the business scope (for feature work).
5. The actual code (for implementation).

The must-read index pattern automates step 3 + 4 — see `references/must-read-index-pattern.md`.
