---
name: a11y-ci-baseline
description: Create or compare an accessibility baseline to track progress.
---

# /a11y-ci-baseline

Runs the `a11y-baseline` skill to create a snapshot of current violations or
compare current state against a previously saved snapshot.

## Usage

```
/a11y-ci-baseline
```

## Behavior

**First run (no `.a11y-baseline.json` exists):**
- Runs a full `a11y-audit` on `src/`.
- Saves results to `.a11y-baseline.json` in the project root.
- Reports total violations captured.

**Subsequent runs (`.a11y-baseline.json` exists):**
- Prompts: compare vs. overwrite.
- Compare mode: shows resolved, new, and unchanged violations with net change.
- Overwrite mode: replaces baseline with fresh audit results.

## CI Integration

Use the baseline in CI to catch regressions:

```yaml
- name: Check a11y regressions
  run: claude /a11y-ci-baseline --ci
```

In `--ci` mode (non-interactive): exits with code 1 if any **new** violations
are detected since the last committed baseline. Exits 0 if unchanged or improved.

## Output Files

- `.a11y-baseline.json` — machine-readable baseline (commit this to the repo).
