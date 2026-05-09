# Mockup sinking method

How to author high-fidelity HTML mockups that live **inside the task package** and tie into the UI review stage.

## When to use

- The task package is a complex one and includes UI review (stage 2).
- The user explicitly asks for high-fidelity prototypes.
- Multiple views need to be reviewed as a flow, not in isolation.

Don't use for: trivial visual fixes, copy-only changes, or single-component tweaks (sketches in chat are enough).

## Layout

```
docs/tasks/<task>/mockups/
├── README.md             # index + recommended walkthrough path
├── _styles.css           # shared design tokens, reset, button / chip / table base, flow-nav style
├── view-A.html
├── view-B.html
├── view-C.html
└── ...
```

Each `<view>.html` is **self-contained**: open by double-click in any browser, no build step, only depends on `_styles.css` and a CDN icon font (Tabler Outline is a good default).

## File template

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Module · View Name</title>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@tabler/icons-webfont@2.47.0/dist/tabler-icons.min.css">
<link rel="stylesheet" href="./_styles.css">
</head>
<body>

<div class="mockup-banner">
  <i class="ti ti-device-desktop"></i>
  desktop 1280 px · <view description>
</div>

<nav class="mockup-flow-nav">
  <span class="flow-label">Flow</span>
  <a href="view-A.html">① View A</a>
  <a href="view-B.html" class="active">② View B</a>
  <a href="view-C.html">③ View C</a>
  <span class="index-link"><a href="README.md">Index</a></span>
</nav>

<div class="mockup-canvas">
  <!-- Mockup body -->
</div>

</body>
</html>
```

## `_styles.css` essentials

The shared stylesheet should at minimum provide:

1. **Design tokens as CSS variables**:
   - `--color-text-primary / -secondary / -tertiary / -info / -success / -warning / -danger`
   - `--color-background-primary / -secondary / -tertiary / -info / -success / -warning / -danger`
   - `--color-border-tertiary / -secondary / -primary`
   - `--border-radius-md / -lg / -xl`
   - `--font-sans / -mono`
2. **Reset + base styles** for `body`, `button`, `input`, `select`, `a`, `h1-h3`, `table`.
3. **`.mockup-banner`** (top status bar with icon + description).
4. **`.mockup-canvas`** (main content wrapper, max-width 1280, centered).
5. **`.mockup-flow-nav`** (the cross-mockup navigation bar — see below).

### `.mockup-flow-nav` style

```css
.mockup-flow-nav {
  background: #fafaf6;
  border-bottom: 0.5px solid var(--color-border-tertiary);
  padding: 7px 14px;
  font-size: 11px;
  display: flex;
  gap: 6px;
  flex-wrap: wrap;
  align-items: center;
  max-width: 1280px;
  margin: 0 auto;
  color: var(--color-text-secondary);
}
.mockup-flow-nav .flow-label { font-weight: 500; color: var(--color-text-primary); margin-right: 4px; }
.mockup-flow-nav a { color: var(--color-text-secondary); padding: 3px 9px; border-radius: 999px; text-decoration: none; border: 0.5px solid transparent; }
.mockup-flow-nav a:hover { background: var(--color-background-secondary); color: var(--color-text-primary); }
.mockup-flow-nav a.active { background: var(--color-background-info); color: var(--color-text-info); border-color: rgba(24, 95, 165, 0.3); font-weight: 500; }
.mockup-flow-nav .sep { color: var(--color-text-tertiary); padding: 0 4px; }
.mockup-flow-nav .index-link { margin-left: auto; }
```

This makes it possible to walk the entire flow in any order, with the current view always highlighted.

## Wire key actions across views

In each view, wire the buttons / cards that represent flow transitions:

```html
<!-- A "view detail" button on a list view -->
<button onclick="location.href='view-detail.html'">
  <i class="ti ti-arrow-right"></i> View detail
</button>

<!-- A clickable card on a profile view -->
<div class="card"
     onclick="location.href='task-detail.html'"
     style="cursor: pointer;">
  ...
</div>
```

The user should be able to:

1. Open any one mockup.
2. Click visible buttons / cards to walk the intended flow.
3. Use the top flow-nav to jump anywhere.

## `mockups/README.md` content

- A short "What this directory contains".
- A **recommended walkthrough**: numbered steps describing the canonical user journey across views.
- A **view list** with file path + one-line purpose + scenario.
- Notes on shared tokens / icons.
- (Optional) Notes about responsive variants / fallback views.

## Reference from `02-ui-review.md`

The UI review stage doc should:

- Link to `mockups/README.md`.
- List design decisions that were ratified during the review (these are the ones that go into `requirements/` later).
- List visual regressions or issues found during review (with mitigation).

The mockups themselves are the **artifact**; the review doc is the **decision record**.

## Implementation pointers (for stage 3)

- `mockup-canvas` width = 1280 px → desktop primary form.
- Use semantic HTML (button / nav / table / details), not div-soup, so devs can map cleanly to components.
- Keep inline styles concrete (don't use vague ones like `padding: 1rem`) — every value should be defensible against design tokens.
- SVG charts inline (no chart library inside the mockup) — devs can swap with the real chart lib at impl time.

## Anti-patterns

| Smell | Fix |
|---|---|
| Mockup loads from CDN libraries other than icon font | Inline everything; mockups should work offline after first cache |
| Different views import different style files | Use one `_styles.css` for the whole task |
| flow-nav order doesn't match the canonical user journey | Reorder; flow-nav is the user's mental map |
| Mockup has fabricated metrics in headline numbers | Use placeholders or "—"; per philosophy rule 5 |
| Mockup file is >2000 lines | Split sub-views into separate files; cross-link via flow-nav |
