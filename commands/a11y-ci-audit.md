---
name: a11y-ci-audit
description: Audit a specific file or directory for WCAG violations.
---

# /a11y-ci-audit [file]

Runs the `a11y-audit` skill on a single file or directory.

## Usage

```
/a11y-ci-audit src/components/Button.tsx
/a11y-ci-audit src/pages/checkout/
/a11y-ci-audit index.html
```

## Behavior

1. Accepts a file path or glob pattern as its argument.
2. Runs `a11y-audit` on the matched file(s).
3. Outputs per-violation records and summary for the targeted scope only.

## Notes

- If no argument is given, falls back to `/a11y-ci` (full `src/` scan).
- Glob patterns are supported: `/a11y-ci-audit src/components/**/*.tsx`
- Useful in pre-commit hooks to check only staged files:
  ```bash
  claude /a11y-ci-audit $(git diff --cached --name-only | grep -E '\.(jsx|tsx|html|vue)$')
  ```
