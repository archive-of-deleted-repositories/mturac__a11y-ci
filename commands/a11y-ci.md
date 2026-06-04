---
name: a11y-ci
description: Audit all components in src/ for WCAG violations.
---

# /a11y-ci

Runs the `a11y-audit` skill on the entire `src/` directory.

## Usage

```
/a11y-ci
```

## Behavior

1. Discovers all `.html`, `.htm`, `.jsx`, `.tsx`, and `.vue` files under `src/`.
2. Runs the full `a11y-audit` skill on the discovered file set.
3. Prints per-violation records grouped by file, sorted by line number.
4. Prints the audit summary block at the end.

## Options

Append a path to override the default `src/` scope:

```
/a11y-ci components/
/a11y-ci src/pages/checkout/
```

## Exit Signal

If any **Level A** violations are found, the command signals failure so CI
pipelines can catch it. Level AA and AAA findings are reported but do not
block by default (configure in `.a11y-ci.config.json` if present).
