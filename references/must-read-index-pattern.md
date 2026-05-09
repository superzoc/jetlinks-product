# Must-read index pattern

How `CLAUDE.md` lets Claude resolve "which long-term docs do I need to read for this task" automatically.

## The convention

`CLAUDE.md` carries a **scope → required reads** mapping table. When Claude picks up a task, it:

1. Identifies the **business scopes** the task hits (a task may hit multiple).
2. Looks up each scope in the table.
3. Reads the listed `constitution/*.md` and `requirements/*.md` files (or the relevant sections if a file is huge).
4. Only then proposes alternatives.

Reading the index is **non-negotiable** for new features, IA changes, or anything that touches business semantics. Skip only for trivial 1–2 file fixes.

## Example index in `CLAUDE.md`

```markdown
## Required reads index

### Cross-cutting (every task)

| Task scope | Required reads |
|---|---|
| Any code change | `docs/constitution/engineering.md`, `docs/constitution/service-layer.md` |
| Any UI / visual / copy change | `docs/constitution/design.md` |
| Historical decision lookup | `docs/constitution/decisions.md` |

### Business scopes

| Task scope | Required reads |
|---|---|
| Product model, IA, top nav | `docs/requirements/product-model.md` |
| IoT module, devices, device groups | `docs/requirements/module-iot.md` + `docs/requirements/project-workspace.md` |
| Video center, video resources | `docs/requirements/module-video.md` + `docs/requirements/project-workspace.md` |
| Behavior analysis module | `docs/requirements/module-behavior-analysis.md` |
| Patrol module | `docs/requirements/module-patrol.md` + `docs/requirements/module-video.md` + `docs/requirements/project-workspace.md` |
| Notifications, channels, recipients | `docs/requirements/product-notifications.md` + `docs/requirements/project-workspace.md` |
| Developer center IA, `/developer/*` | `docs/requirements/developer-center.md` + `docs/requirements/product-capability-layering.md` |
| Vision training assistant | `docs/requirements/developer-vision-training.md` |
```

When a task hits multiple scopes, **all matching rows are required reads** (deduplicate).

## How to identify scope on intake

Walk the user's request and tag scopes:

| User says | Scope hit |
|---|---|
| "add a new module X" | `product-model.md` (top-level IA) + the matching `module-*.md` |
| "redesign the iot module's device list" | `module-iot.md` + `project-workspace.md` + `design.md` |
| "fix the alarm rule editor copy" | `design.md` (UI / copy) + `module-alarm.md` (if exists) |
| "refactor the long task runner" | `service-layer.md` |
| "add a new persona" | usually maps to a specific module; trace from the persona's data |

When ambiguous, ask the user before reading dozens of files.

## Reading strategy for large files

Some requirements docs (legacy or comprehensive) can be >1200 lines. Don't load them entirely — that wastes context.

Strategy:

1. Read the **table of contents** or first ~200 lines (overview, decisions table, frontmatter).
2. **Grep** for the specific term the task is about (field name, decision ID, page name).
3. Use **Read** with `offset` + `limit` to load the relevant section (typically 50–200 lines).
4. Only escalate to "read full file" if the task genuinely spans many sections.

## Where the index lives

- **Primary**: `CLAUDE.md` — single source of truth.
- **Mirror**: a project's `AGENTS.md` and `GEMINI.md` should point back to `CLAUDE.md`, not duplicate the index.
- **Stale check**: every quarter, scan whether `requirements/*.md` files exist that aren't in the index. Add them.

## Maintaining the index

When a new requirements doc is added:

1. Decide what business scope it covers.
2. Add a row to the index in `CLAUDE.md`.
3. If the new doc supersedes an old one, leave the old row pointing at the new file or remove the old row (depending on archive status).

When a requirements doc is **renamed**, run a quick check:

```bash
rg "old-name.md" docs CLAUDE.md --glob '!docs/**/_archive/**'
```

…and update every reference, including the index.

## Anti-patterns

| Smell | Fix |
|---|---|
| Claude reads "everything in `requirements/`" on every task | Use the index; it should narrow to 1–3 files per task |
| Claude only reads task-package files and ignores requirements | Force a check: "Did I look up the scope in the index?" |
| Index references files that don't exist | Audit and prune; broken index = trust collapse |
| Two scopes share required-reads but the table has each listed separately | Refactor: extract the shared reads into a "cross-cutting" sub-table |

## Test it

A quick test that the index is healthy:

> Pick a feature description from the codebase. Without reading the code, ask: "What requirements + constitution files would I need to read to start work?" The answer should fall out of the index in <30 seconds.

If you can't answer in 30 seconds, the index needs work.
