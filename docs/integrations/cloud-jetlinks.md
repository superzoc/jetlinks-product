# Integration: cloud.jetlinks

How to enable `jetlinks-product` inside the [cloud.jetlinks](https://github.com/jetlinks-v2/cloud.jetlinks) repository (and how to make Claude / Codex / Gemini pick it up automatically when you open the repo).

> **Last verified against cloud.jetlinks**: 2026-05-10. Branch `2.12`. If you find drift, file an issue (link at the bottom).

## Why a project-specific guide

`cloud.jetlinks/AGENTS.md` says **"no local skills, use the shared `jetlinks-develop-skills` repository"**. This guide explains how to plug `jetlinks-product` in without violating that rule:

- `jetlinks-product` is hosted standalone at https://github.com/superzoc/jetlinks-product (not vendored into `cloud.jetlinks`).
- `cloud.jetlinks` only references the skill in its `AGENTS.md` — no skill files committed locally.
- Installation happens at the **agent client** level (cc-switch / user-level `~/.claude/skills/`), not in the repo.

## Workspace shape (what cloud.jetlinks looks like)

`cloud.jetlinks` is a multi-stack Maven workspace with a Vue ops UI alongside it. Top-level areas, all real code locations:

| Area | What it is |
|---|---|
| `modules/` | Backend business modules (`jetlinks-account`, `jetlinks-payment`, `jetlinks-saas`, `jetlinks-tenant`, `authentication-manager`, `jetlinks-capability-marketplace`, plus several `.gitmodules`-linked capability repos) |
| `runtime/` | Runtime microservices (`api-gateway-service`, `iot-service`, `notify-service`, `rule-engine-service`, etc.) |
| `control/` | Control-plane services (`saas-control-service`) |
| `ui/` | Operator-side frontend workspace (`jetlinks-web-core` + `ui/modules/<feature>-manager-ui`) |
| `runtime-ui/` | Runtime frontend extension workspace |
| `jetlinks-parent/` | Shared Maven parent and dependency management |
| `docs/` | Repo docs; see `docs/plans/` for index, `docs/requirements/` for product blueprints (see below), `docs/tasks/` for per-delivery |

Two important shape facts the skill must accommodate:

1. **`AGENTS.md` is canonical**, `CLAUDE.md` is a thin bridge that imports it. Don't write rules in `CLAUDE.md` — write them in `AGENTS.md`.
2. **`docs/constitution/` does not exist in cloud.jetlinks** and is not planned. Engineering / design / service-layer rules live in `AGENTS.md`, in shared focused skills (`jetlinks-conventions`, `jetlinks-reactive`, `jetlinks-web`, etc.), and in module READMEs. Skill must work without `docs/constitution/`.

The skill core (`SKILL.md` + `references/document-layering.md` + `references/must-read-index-pattern.md`) accommodates both shape facts as of 2026-05-10. If you see assumptions that break here, file an issue.

## Step 1 — Install the skill on your machine

Pick **one** of the three options.

### Option A · cc-switch one-click (recommended)

```text
Open cc-switch → Skills tab → "Install from GitHub" →
paste: https://github.com/superzoc/jetlinks-product
```

cc-switch keeps the skill updated; works across Claude Code, Codex, Gemini CLI.

### Option B · User-level Claude Code skills directory

```bash
mkdir -p ~/.claude/skills
cd ~/.claude/skills
git clone https://github.com/superzoc/jetlinks-product.git
```

Updating later: `cd ~/.claude/skills/jetlinks-product && git pull`.

### Option C · Project-level skill (NOT recommended for cloud.jetlinks)

```bash
cd ~/projects/cloud.jetlinks
mkdir -p .claude/skills
cd .claude/skills
git clone https://github.com/superzoc/jetlinks-product.git
echo ".claude/skills/" >> ../../.gitignore
```

⚠️ This contradicts `cloud.jetlinks/AGENTS.md` ("no local skills"). Use only for short-lived experimentation; clean up before committing.

## Step 2 — Add a row to `cloud.jetlinks/AGENTS.md` Skill Routing

`cloud.jetlinks/AGENTS.md` already has a `## Skill Routing` section listing focused skills (`jetlinks-protocol`, `jetlinks-routing`, `jetlinks-crud`, `jetlinks-events`, `jetlinks-web`, `jetlinks-delivery`, etc.). The minimal integration is to add `jetlinks-product` as one more row:

```markdown
- 新功能 / 多阶段交付 / 任务包 / 高保真 mockup 沉淀：`jetlinks-product`（来自 https://github.com/superzoc/jetlinks-product，非共享仓库的扩展技能）
```

Place this row alongside the existing focused-skill list. Don't add long prose to `AGENTS.md` — the skill itself carries the protocol; `AGENTS.md` only needs the routing pointer.

If `cloud.jetlinks/CLAUDE.md` is still the canonical `@AGENTS.md` bridge (current state), you don't need to touch it — the routing flows through automatically.

### Auto-install hint (optional)

Below the routing row, you can add the same `$skill-installer` hint cloud.jetlinks already uses for `jetlinks-router` and friends:

```text
Use $skill-installer to install skill from https://github.com/superzoc/jetlinks-product
```

## Step 3 — Verify the skill triggers

Open a fresh Claude Code session in `~/projects/cloud.jetlinks` and try:

| Prompt | Expected behavior |
|---|---|
| "Let's design a new module called X" | Picks up `jetlinks-product`; proposes ≥2 alternatives; offers to build a complex task package |
| "Build a complex task package for feature Y" | Picks up `jetlinks-product`; copies template from `assets/complex-task-package/`; asks about goals / non-goals |
| "Just review this PR" | Does NOT pick up `jetlinks-product` (out of scope) |
| "Where is the device service in this repo?" | Picks up `jetlinks-routing`, NOT `jetlinks-product` |
| "Add a new module + design its IA + plan implementation" | Picks up BOTH skills |

If the first prompt does not trigger the skill, check:

1. The skill is installed in a directory the agent client knows about (`claude code skills list` should show `jetlinks-product`).
2. The frontmatter `description` field in `SKILL.md` is intact (`< 1024` chars, present).
3. You restarted the agent client after installing the skill.

## Step 4 — First real task

The first end-to-end run in cloud.jetlinks (verified 2026-05-10) sunk the requirements doc and built a complex task package for the **behavior analysis module**:

- Requirements: `cloud.jetlinks/docs/requirements/module-behavior-analysis.md`
- Task package: `cloud.jetlinks/docs/tasks/2026-05-10-0038-behavior-analysis-bring-up/`

That run validated the full flow:

1. Claude proposed alternatives (philosophy rule 1) — 3 questions in `AskUserQuestion` form; user signed off.
2. Requirements doc sunk to `docs/requirements/` (a new directory).
3. Task package created at `docs/tasks/<timestamp>-<slug>/` (a new directory; cloud.jetlinks didn't have `docs/tasks/` before this run — Claude created it without modification to AGENTS.md required, since the skill's protocol is the source of truth).
4. 5 stage docs filled (`README` / `01-requirements` / `02-ui-review` / `03-implementation-plan` / `04-verification` / `handoff`).
5. Cross-cutting decisions surfaced in `AskUserQuestion` form, one decision at a time, never bundled.
6. Naming convention table added to the requirements doc (mapping prototype-style names to cloud.jetlinks conventions like `jetlinks-<feature>` backend + `<feature>-manager-ui` frontend).

Use that run as the reference template for any subsequent module / feature you bring up.

## Coexistence map

`cloud.jetlinks` already uses several conventions; here's how `jetlinks-product` interacts with each:

| cloud.jetlinks convention | jetlinks-product behavior |
|---|---|
| `AGENTS.md` says "use shared skills" | ✅ Compatible — this skill is referenced via `AGENTS.md` skill routing, not vendored |
| `CLAUDE.md` is a thin `@AGENTS.md` bridge | ✅ Skill treats AGENTS.md as canonical entry doc; CLAUDE.md needs no edits |
| `jetlinks-router` / `jetlinks-routing` / `jetlinks-crud` / `jetlinks-events` / `jetlinks-web` / `jetlinks-conventions` / `jetlinks-delivery` etc. | ✅ Coexists; this skill provides the **delivery protocol**, focused skills provide the **how to write code** |
| No `docs/constitution/` | ✅ Skill works without it (as of SKILL.md v0.2). Must-read lists point at `AGENTS.md` sections + module READMEs instead |
| Multi-stack split (`modules/` Java + `ui/modules/` Vue) | ✅ Skill's "repo shapes" guidance covers this; task package `README.md` should list backend + frontend file scopes separately |
| `.gitmodules`-linked capability repos in `modules/` | ✅ List submodule directories under "do not touch" in task package; verify `git status` shows no submodule pointer changes |
| Frontend Scope Guardrails (`ui/` vs `runtime-ui/`) | ✅ Skill defers to `AGENTS.md` for which frontend area applies; task package picks one and sticks to it |
| `docs/plans/` (existing repo-level index) | ✅ Skill writes to `docs/requirements/` (blueprints) and `docs/tasks/` (per-delivery), not `docs/plans/`. `docs/plans/README.md` continues to be the repo-level index it already is |

## Skill ↔ shared focused skills routing inside cloud.jetlinks

When a feature flows through the skill's 5 stages, expect to use focused skills at specific phases:

| Stage | Likely focused skill(s) |
|---|---|
| 01 — requirements clarification | `jetlinks-router` (to confirm scope), `jetlinks-routing` (to pick owning module) |
| 02 — UI review | `jetlinks-web-style` (style selection), then `jetlinks-web` (page implementation) |
| 03 — implementation plan | `jetlinks-crud` / `jetlinks-events` / `jetlinks-reactive` / `jetlinks-conventions` per task |
| 04 — verification | `jetlinks-delivery` (commit, PR, test evidence) |
| 05 — handoff | n/a (in-task tracking only) |
| Post-task | `jetlinks-capture` (only when stable, reusable knowledge surfaced) |

This skill **does not replace** the focused skills. It orchestrates the delivery; focused skills do the work.

## Feedback loop

When you find a flow that doesn't fit `cloud.jetlinks`, file an issue at:

https://github.com/superzoc/jetlinks-product/issues

Include:

- The cloud.jetlinks-specific friction (what convention does the skill assume that doesn't hold here)
- A minimal repro prompt
- Whether you'd like the skill changed, or `cloud.jetlinks` adapted instead

## Future integrations

Each JetLinks repo gets its own file under [`docs/integrations/`](.):

- `docs/integrations/cloud-jetlinks.md` ← you are here
- `docs/integrations/runtime-ui.md` (TODO — when an integration run validates the runtime frontend workspace)
- `docs/integrations/jetlinks-ai-app.md` (TODO — for the original prototype repo where the patterns came from)

The skill itself stays project-agnostic; integration guides live here.
