---
name: a11y-baseline
description: >
  Creates and compares accessibility baselines. Saves audit results to
  .a11y-baseline.json and on subsequent runs reports new violations,
  resolved issues, and net change.
triggers:
  - create a11y baseline
  - compare accessibility baseline
  - track wcag progress
  - a11y-baseline
---

# a11y-baseline

Track accessibility progress over time by saving and comparing audit snapshots.

## Steps

### 1. Determine Mode

Check whether `.a11y-baseline.json` exists in the project root:

- **File does not exist:** run in **Create** mode.
- **File exists:** run in **Compare** mode. Ask the user:
  ```
  A baseline already exists (created: 2026-05-01T10:30:00Z, 12 issues).
  Options:
    [1] Compare current state against baseline
    [2] Overwrite baseline with fresh audit
  ```

### 2. Run Audit

Invoke the `a11y-audit` skill internally on the full `src/` directory (or the
scope the user specifies). Collect the structured violation records.

### 3a. Create Mode — Save Baseline

Write `.a11y-baseline.json` to the project root:

```json
{
  "version": 1,
  "createdAt": "2026-05-01T10:30:00Z",
  "scope": "src/**/*.{html,htm,jsx,tsx,vue}",
  "totalIssues": 12,
  "violations": [
    {
      "file": "src/components/Hero.tsx",
      "line": 7,
      "wcag": "1.1.1",
      "severity": "A",
      "violation": "Image missing alt text",
      "fingerprint": "Hero.tsx:7:img-missing-alt"
    },
    {
      "file": "src/components/Button.tsx",
      "line": 42,
      "wcag": "4.1.2",
      "severity": "A",
      "violation": "Button without accessible text",
      "fingerprint": "Button.tsx:42:button-no-text"
    }
  ]
}
```

The `fingerprint` field is computed as `{filename}:{line}:{violation-type-slug}`.
It is used for matching violations across runs even if line numbers shift
slightly.

Output:
```
Baseline created: .a11y-baseline.json
Total violations captured: 12
  Level A:  8
  Level AA: 4
Run /a11y-ci-baseline again after fixing issues to see your progress.
```

### 3b. Compare Mode — Diff Against Baseline

Load `.a11y-baseline.json`. Run the current audit. Compute three sets:

- **Resolved:** violations in baseline that are NOT in current audit (fingerprint match).
- **New:** violations in current audit that are NOT in baseline.
- **Unchanged:** violations present in both.

Build the diff report:

```
=== A11Y BASELINE COMPARISON ===
Baseline date:  2026-05-01T10:30:00Z
Compared at:    2026-06-05T09:00:00Z

Resolved (fixed since baseline):  3
  - Hero.tsx:7      1.1.1 (A)  Image missing alt text
  - Form.jsx:18     1.3.1 (A)  Input missing label
  - Nav.tsx:5       3.1.1 (A)  Missing lang attribute

New (introduced since baseline):  1
  - Modal.tsx:31    4.1.2 (A)  Button without accessible text

Unchanged:  9

NET CHANGE: -2 violations (improved)
```

If regressions exist (New > 0), print a warning:
```
WARNING: 1 new violation introduced. Address before merging.
```

If the project is fully clean:
```
All baseline violations resolved. No new issues. Project is WCAG 2.1 Level AA clean.
```

### 4. Persist Updated Baseline (Optional)

After Compare mode, ask:
```
Update baseline to current state? (y / n)
```

If "y", overwrite `.a11y-baseline.json` with the current audit results and
new timestamp. If "n", keep the original baseline for future comparisons.

### 5. Baseline File Location

Always place `.a11y-baseline.json` in the project root (same level as
`package.json` or `index.html`). If the user wants a different path, honor
it and record the custom path in the baseline's own `"scope"` field.

Add `.a11y-baseline.json` to `.gitignore` only if the user requests it.
Committing the baseline is recommended so CI can detect regressions.
