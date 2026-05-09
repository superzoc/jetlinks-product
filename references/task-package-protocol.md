# Task package protocol

How to choose, create, transition, and archive a task in `docs/tasks/`.

## Decision tree

| Scope | Form | Why |
|---|---|---|
| 1–2 file change, fix, copy tweak, simple Q&A | **No task doc** — implement, run necessary checks, report in chat | Overhead exceeds value |
| 3–5 files, single stage, no IA / data changes, can finish in one session | **Simple task note** at `docs/tasks/YYYY-MM-DD-HHMM-<name>.md` | Captures intent + verification, no multi-stage overhead |
| New feature, multi-stage delivery, multi-model handoff, IA / data / many-file changes | **Complex task package** at `docs/tasks/YYYY-MM-DD-HHMM-<name>/` | Persists context across stages and people |

When uncertain between simple and complex, default to **simple** and upgrade if you discover hidden complexity. Never default to "no task" for a multi-file change unless the user explicitly says so.

## Naming

```text
YYYY-MM-DD-HHMM-<task-slug>
```

- 4-digit local 24h time. Choose at creation time and don't rename.
- `<task-slug>`: lowercase, kebab-case, ≤4 words.
- Two parallel sub-tasks: append `-A` / `-B` to the slug, not the timestamp.

Lexical sort = creation order. Helps everyone find "what's currently in flight".

## Simple task note

Single file, ~30–60 lines. Template at `assets/simple-task.md`. Sections:

- Goal
- Non-goals
- Files in scope
- Implementation notes
- Verification
- Handoff (if not finished in this session)

## Complex task package

Directory of 6 files. Template at `assets/complex-task-package/`. Files:

```
docs/tasks/YYYY-MM-DD-HHMM-<task>/
├── README.md            # Goal, non-goals, file scope, required reads, deliverables, risks
├── 01-requirements.md   # Requirements clarification stage output
├── 02-ui-review.md      # UI review stage output
├── 03-implementation-plan.md  # Implementation plan stage output
├── 04-verification.md   # Verification stage output
└── handoff.md           # Cross-stage handoff (current stage, status, next steps, do-not-do)
```

### Required reads in `README.md`

`README.md` lists what the **next executor must read before touching any file**. At minimum:

1. The repo's `AGENTS.md` / `CLAUDE.md`.
2. This task package's `handoff.md`.
3. The task package's other stage files.
4. Long-term docs the task hits — resolved via the must-read index pattern.

If a constitution / requirements file exists for the task's scope, list it explicitly. Don't make the next executor guess.

### `handoff.md` is the heartbeat

Updated at every stage transition. Required sections:

- **Current stage** (one of the five)
- **Status** (in progress / blocked / done)
- **Last executor** (Claude / Codex / Gemini / human)
- **Done** (bullet list of completed items)
- **Key conclusions** (decisions ratified, can be referenced from other docs)
- **Next steps** (what to do next, named files / commands)
- **Risks** (open items the next executor should not be surprised by)
- **Do-not-do** (explicit guardrails — what NOT to touch in this delivery)

## Five collaboration stages

Stages, not roles. Any AI or human takes any stage. Stage 5 (handoff) runs continuously alongside stages 1–4.

| Stage | Output | Done criteria |
|---|---|---|
| 1. Requirements clarification | `01-requirements.md` | Goal, non-goals, user paths, edges, open questions are explicit and signed off |
| 2. UI review | `02-ui-review.md` | Layout, components, copy, accessibility, responsive behavior decided; mockups (if any) sunk into `mockups/` |
| 3. Implementation plan | `03-implementation-plan.md` | File list, technical approach, step-by-step plan, risk control, verification list ready |
| 4. Verification | `04-verification.md` | Auto checks pass, manual checks pass, residual risks documented, write-back targets identified |
| 5. Handoff | `handoff.md` | Updated at every transition |

Stages run in order. Skipping a stage is allowed only when the user explicitly waives it.

## Archive convention

When a task is **verified** and stable conclusions are written back to `requirements/` or `constitution/`:

```bash
git mv docs/tasks/2026-05-08-1920-mod-X/ docs/tasks/_archive/2026-05-08-1920-mod-X/
```

Then commit:

```text
chore(tasks): archive 2026-05-08-1920-mod-X
```

The top of `docs/tasks/` should only contain currently-running tasks. Archived tasks are read-only history; don't reference them as canonical.

### When to archive

Archive when **any** of:

- Task passed verification + stable conclusions are written back upstream.
- Task is abandoned / reversed by a product decision (note this in `handoff.md` before archiving).
- Task is stale (≥4 weeks no progress) and no restart plan — confirm with user before archiving.

### Who archives

The last executor to close out the task (whoever ran stage 4) archives. If unsure, leave a "suggest archive" note in `handoff.md` for the user to confirm.

### Reading archived tasks

Default: **don't read archived tasks**. They reference old IA, fields, and decisions that are no longer canonical. Read only when the user explicitly asks "how was X done historically" or when an active task's `README.md` cites a specific archived task as a known dependency.

## Migration upward

After verification, before archiving, identify what to migrate:

- **Stable product conclusions** → write back to `docs/requirements/*.md`.
- **Stable cross-cutting rules** → write back to `docs/constitution/*.md`.
- **One-off implementation steps** → leave in the task package, not migrated.
- **Open questions still unanswered** → carry forward to the next task or `requirements/_open-questions.md`.

The task package's `04-verification.md` should explicitly list "items written back" so reviewers can see migration was done.

## Anti-patterns

| Smell | Fix |
|---|---|
| Task package has been "in progress" for >4 weeks with no commits | Either restart with a new package or archive (with reason) |
| `handoff.md` says "done" but `04-verification.md` is empty | Block archiving until verification is real |
| Two task packages overlap on the same files | Merge or split with explicit `-A` / `-B` slugs |
| `requirements/X.md` cites a task package path | Migration was skipped; fix the citation by writing the conclusion into requirements |
