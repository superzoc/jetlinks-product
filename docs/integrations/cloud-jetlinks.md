# Integration: cloud.jetlinks

How to enable `jetlinks-product` inside the [cloud.jetlinks](https://github.com/jetlinks-v2/cloud.jetlinks) repository (and how to make Claude / Codex / Gemini pick it up automatically when you open the repo).

## Why a project-specific guide

`cloud.jetlinks/AGENTS.md` says **"no local skills, use the shared `jetlinks-develop-skills` repository"**. This guide explains how to plug `jetlinks-product` in without violating that rule:

- `jetlinks-product` is hosted standalone at https://github.com/superzoc/jetlinks-product (not vendored into `cloud.jetlinks`).
- `cloud.jetlinks` only references the skill in its `AGENTS.md` — no skill files committed locally.
- Installation happens at the **agent client** level (cc-switch / user-level `~/.claude/skills/`), not in the repo.

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

## Step 2 — Add a reference in `cloud.jetlinks/AGENTS.md`

Append this section to `~/projects/cloud.jetlinks/AGENTS.md`:

```markdown
## Product design flow

When starting a new feature / module, scoping a complex multi-stage delivery,
or building a task package, Claude should follow the protocol defined by the
[jetlinks-product](https://github.com/superzoc/jetlinks-product) skill.

If the skill is not installed in your environment yet:

\`\`\`text
Use $skill-installer to install skill from https://github.com/superzoc/jetlinks-product
\`\`\`

This skill provides:
- Five collaboration rules (propose alternatives before implementing,
  wait for sign-off, stage by stage, never silently revert, never fabricate metrics)
- Four-layer doc taxonomy (CLAUDE.md / constitution / requirements / tasks)
- Simple vs complex task package templates
- High-fidelity HTML mockup sinking method
- Must-read index pattern that resolves which long-term docs to read

This skill coexists with `jetlinks-routing`:
- `jetlinks-routing` → where in the multi-module repo to find specific code
- `jetlinks-product` → how to organize the design / delivery flow itself
```

Mirror the same paragraph (or a 1-line pointer back to `AGENTS.md`) into:

- `cloud.jetlinks/CLAUDE.md` (so Claude Code picks it up)
- `cloud.jetlinks/.cursor/rules/develop.mdc` (if you use Cursor; optional)

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

## Step 4 — First real task to try

Recommended first task to validate the full flow end-to-end inside `cloud.jetlinks`:

> **Task**: design + plan a new minor feature (something already scoped in product backlog).

The flow should run:

1. Claude proposes ≥2 design alternatives (philosophy rule 1).
2. You sign off on one (philosophy rule 2).
3. Claude scaffolds a complex task package at `docs/tasks/YYYY-MM-DD-HHMM-<feature>/` (cloud.jetlinks doesn't have `docs/tasks/` yet — Claude should create it; the skill protocol is the source of truth).
4. Claude fills `01-requirements.md` and `handoff.md`.
5. You sign off; Claude moves to `02-ui-review.md` (if UI involved) or `03-implementation-plan.md`.
6. Implementation follows stage by stage (philosophy rule 3).
7. Verification recorded in `04-verification.md`; stable conclusions migrated upward to `docs/requirements/` if a long-term spec was clarified.

If steps don't follow this flow, capture a transcript and file an issue (see Feedback below).

## Coexistence map

`cloud.jetlinks` already uses several conventions; here's how `jetlinks-product` interacts with each:

| cloud.jetlinks convention | jetlinks-product behavior |
|---|---|
| `AGENTS.md` says "use shared skills" | ✅ Compatible — this skill IS the shared kind |
| `jetlinks-routing` skill (where-to-find-code) | ✅ Coexists; both run in parallel on full-stack tasks |
| `.prompt/*.md` files | Read on demand; `jetlinks-product` doesn't override them |
| `.claude/rules.md`, `.claude/advanced-rules.md` | Read on demand; layer below skill instructions |
| `.cursor/rules/develop.mdc` | Cursor-specific; jetlinks-product is agent-client-agnostic |
| `modules/` + `runtime/` + `runtime-ui/` + `ui/` four-area split | The skill's "must-read index pattern" maps cleanly: each area gets its own scope row in the index table when you build one for `cloud.jetlinks` |

## Recommended: build a `cloud.jetlinks/CLAUDE.md` must-read index

`cloud.jetlinks/CLAUDE.md` is currently a thin pointer to shared skills. To get the most out of `jetlinks-product`, extend it with a must-read index (see [`references/must-read-index-pattern.md`](../../references/must-read-index-pattern.md)). Suggested seed:

```markdown
## Required reads index

### Cross-cutting
| Task scope | Required reads |
|---|---|
| Any backend code | `.prompt/common-crud-rules.md`, `.prompt/module-creation-rules.md` |
| Backend module reference | `.prompt/module-list.md`, `.prompt/module-reference.md` |
| Cross-service call | `.prompt/cross-service-call-rules.md` |
| Event-driven work | `.prompt/event-driven-rules.md` |
| Realtime subscription | `.prompt/realtime-subscription-rules.md` |
| Any frontend code | (TBD — point at runtime-ui/CLAUDE.md or ui/CLAUDE.md) |

### Frontend boundary (per AGENTS.md "Frontend Scope Guardrails")
| Task scope | Default area |
|---|---|
| Operator-side full-stack | `modules/` + `ui/` |
| Runtime frontend extension | `runtime-ui/` |
```

This isn't part of `jetlinks-product` itself — it's project-specific content. But the skill's must-read index pattern tells you the **shape**; the table above is suggested **content**.

## Feedback loop

When you find a flow that doesn't fit `cloud.jetlinks`, file an issue at:

https://github.com/superzoc/jetlinks-product/issues

Include:

- The cloud.jetlinks-specific friction (what convention does the skill assume that doesn't hold here)
- A minimal repro prompt
- Whether you'd like the skill changed, or `cloud.jetlinks` adapted instead

## Future integrations

If we add other JetLinks repos (`runtime-ui`, manager modules, etc.), each gets its own file under [`docs/integrations/`](.):

- `docs/integrations/cloud-jetlinks.md` ← you are here
- `docs/integrations/runtime-ui.md` (TODO)
- `docs/integrations/jetlinks-ai-app.md` (TODO — for whatever replaces the current jetlinks-ai-app prototype repo)

The skill itself stays project-agnostic; integration guides live here.
