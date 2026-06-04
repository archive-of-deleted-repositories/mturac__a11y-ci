---
name: a11y-ci-fix
description: Interactively apply accessibility fixes to components.
---

# /a11y-ci-fix

Runs the `a11y-fix` skill to guide you through applying WCAG fixes
interactively, one violation at a time.

## Usage

```
/a11y-ci-fix
/a11y-ci-fix src/components/Button.tsx
```

## Behavior

1. If audit findings are already in context, loads them directly.
2. Otherwise runs a fresh `a11y-audit` on the target scope.
3. Presents a numbered list of auto-fixable violations.
4. For each selected violation, shows a before/after diff and asks for
   confirmation before writing.
5. Non-fixable violations (structural, CSS-only) are listed at the end
   as requiring manual intervention.

## Fix Types Supported

| Violation | Fix Applied |
|-----------|-------------|
| Image missing alt | Suggests alt text from filename/context, user confirms |
| Input missing label | Generates `<label for="…">` element |
| Button no accessible text | Adds `aria-label` derived from class/context |

## Safety

No file is modified without explicit user confirmation per violation.
Use "n" to skip any fix. Use "all" to approve all suggestions at once
(not recommended for alt text — review each suggestion).
