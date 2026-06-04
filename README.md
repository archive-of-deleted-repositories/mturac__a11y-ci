# a11y-ci

Claude Code plugin for automated accessibility audits. Runs static WCAG 2.1
checks on HTML, JSX/TSX, and Vue SFC components, annotates violations in-source
with line numbers, and suggests fixes with code snippets.

## Install

```bash
claude plugins install mturac/a11y-ci
```

Or copy the plugin directory into your project's `.claude/plugins/` folder.

## Commands

| Command | Description |
|---------|-------------|
| `/a11y-ci` | Audit all components in `src/` |
| `/a11y-ci-audit [file]` | Audit a specific file or directory |
| `/a11y-ci-fix` | Apply fixes interactively with before/after diffs |
| `/a11y-ci-baseline` | Create or compare an accessibility baseline |

## Example Violation Output

```
FILE:        src/components/Hero.tsx
LINE:        7
WCAG:        1.1.1
SEVERITY:    A
VIOLATION:   Image missing alt text
DESCRIPTION: <img> element has no alt attribute. Screen readers will
             announce the filename instead of meaningful content.
FIX:
  Before:  <img src="/images/hero-banner.png" className="hero">
  After:   <img src="/images/hero-banner.png" className="hero" alt="Hero banner showing the product dashboard">

FILE:        src/components/Toolbar.tsx
LINE:        42
WCAG:        4.1.2
SEVERITY:    A
VIOLATION:   Button without accessible text
DESCRIPTION: <button> contains only an SVG icon with no aria-label.
FIX:
  Before:  <button className="btn-close"><svg aria-hidden="true" .../></button>
  After:   <button className="btn-close" aria-label="Close"><svg aria-hidden="true" .../></button>

FILE:        src/pages/Contact.jsx
LINE:        18
WCAG:        1.3.1
SEVERITY:    A
VIOLATION:   Input without associated label
DESCRIPTION: <input type="email"> has no <label for=""> or aria-label.
FIX:
  Before:  <input type="email" name="email" />
  After:   <label htmlFor="email">Email address</label>
           <input id="email" type="email" name="email" />
```

## Violation Table (Example Audit)

| File | Line | WCAG | Level | Violation |
|------|------|------|-------|-----------|
| Hero.tsx | 7 | 1.1.1 | A | Image missing alt text |
| Toolbar.tsx | 42 | 4.1.2 | A | Button without accessible text |
| Contact.jsx | 18 | 1.3.1 | A | Input missing label |
| Layout.html | 1 | 3.1.1 | A | Missing lang attribute on `<html>` |
| App.tsx | 23 | 1.3.1 | A | Heading skips from h1 to h3 |
| theme.css | 14 | 2.4.7 | AA | outline:none without focus alternative |
| Card.tsx | 55 | 1.4.3 | AA | Contrast ratio 3.1:1 below 4.5:1 threshold |

## WCAG Criterion Reference

| Criterion | Level | Description |
|-----------|-------|-------------|
| 1.1.1 | A | Non-text Content — images need alt text |
| 1.3.1 | A | Info and Relationships — form inputs need labels; headings must be hierarchical |
| 1.4.3 | AA | Contrast (Minimum) — 4.5:1 for normal text, 3:1 for large text |
| 2.4.7 | AA | Focus Visible — interactive elements must show keyboard focus |
| 3.1.1 | A | Language of Page — `<html>` element must have lang attribute |
| 4.1.2 | A | Name, Role, Value — buttons/links need accessible names; ARIA roles must be valid |

## Baseline Tracking

Create a baseline snapshot to track accessibility progress over time:

```bash
# Create initial baseline
/a11y-ci-baseline

# After fixing some issues, compare
/a11y-ci-baseline
# → Shows: 3 resolved, 0 new, net -3 (improved)
```

The baseline file (`.a11y-baseline.json`) should be committed to your repo
so CI can detect regressions on every pull request.

## CI Integration

Add to your GitHub Actions workflow:

```yaml
- name: Accessibility audit
  run: claude /a11y-ci
```

Level A violations exit with code 1, blocking the build. Level AA/AAA
findings are reported but non-blocking by default.
