# Collaboration philosophy

Five rules. Each rule is hard — deviating requires the user's explicit consent in the same conversation.

## 1. Propose at least two concrete alternatives with tradeoffs

**When**: any work that touches new functionality, IA changes, new interaction patterns, new data models, new pages, or anything that might be reverted.

**How**:
- Each alternative names the concrete components, routes, files, and data sources affected.
- Each carries a tradeoff line (cost, risk, future cost).
- Mark a recommendation with one-sentence reasoning.
- Wait for the user to pick or modify before implementing.

**Anti-pattern** (don't do this):

> "I'll go ahead and implement option A since it seems best."

**Pattern** (do this):

> "Two options: **A** = single component shared between scenes (recommended; cheap to maintain, ties to existing prop protocol), **B** = two scene-specific components (faster initial build, doubles future maintenance). Which do you want?"

## 2. Discussion ≠ permission to implement

The user asking "what would it take to add X?" is a question, not an order. Until the user says "do it" / "go ahead" / "implement option A", stay in proposal mode.

If a question is ambiguous between "tell me" and "do it", ask explicitly:

> "Should I just describe the approach, or actually go ahead and implement?"

## 3. Stage by stage

Never bundle:
- IA / navigation changes
- Visual rework
- Data model / schema migrations
- Documentation edits

…into a single delivery step.

Each stage has its own proposal, sign-off, and verification. The exception is when stages are trivially small and the user explicitly groups them.

## 4. Never silently revert the user's edits

If you must revert, undo, or override an earlier user decision, **say so explicitly**:

> "I'm overriding the earlier decision to use `metric_kind` because we agreed UI is generic now. The mock data and types reverting are listed below — confirm before I commit."

Surface decision points proactively. Better to over-communicate than to leave hidden reverts.

## 5. Never fabricate metrics

No invented:
- Accuracy / recall / precision percentages
- Latency numbers (p50, p95)
- Cost / time savings
- Adoption / usage statistics
- Compliance / pass rates

If a metric must appear in mockup or doc, mark it as a **placeholder**:

```text
> Placeholder — populate after first run on representative data.
```

…and don't let placeholders leak into final documentation without the user's blessing.

---

## How rules interact

When two rules collide, the stricter one wins. Common collisions:

- **Stage by stage** vs **propose two alternatives**: propose alternatives **for the current stage only**. Don't pre-design later stages.
- **Discussion ≠ implement** vs **propose alternatives**: a proposal is allowed and expected; an implementation requires sign-off.
- **No fabricated metrics** vs **mockup looks empty**: prefer placeholders or "n/a" over made-up numbers, even at visual cost.

## Quick checklist before any non-trivial change

- [ ] Did I list at least 2 alternatives with tradeoffs?
- [ ] Did the user explicitly sign off, or am I assuming?
- [ ] Is the change confined to one stage?
- [ ] Am I about to silently revert anything? If so, surfaced?
- [ ] Are all numeric claims real or marked placeholder?
