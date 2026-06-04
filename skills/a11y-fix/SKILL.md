---
name: a11y-fix
description: >
  Interactively applies specific accessibility fixes to source files.
  Shows exact before/after diffs and requires user confirmation before
  writing any change to disk.
triggers:
  - fix accessibility issues
  - apply a11y fix
  - fix wcag violation
  - a11y-fix
---

# a11y-fix

Apply targeted accessibility fixes interactively, one issue at a time.

## Prerequisite

Run `a11y-audit` first (or provide audit findings). This skill operates on
the structured violation records produced by that audit.

## Steps

### 1. Load Violations

Accept violation records from a prior `a11y-audit` run, or re-run the audit
internally if no findings are available. Present the list of fixable
violations to the user:

```
Fixable violations found:

[1] Button.tsx:42  — 4.1.2 (A)  Button without accessible text
[2] Form.jsx:18   — 1.3.1 (A)  Input missing associated label
[3] Hero.tsx:7    — 1.1.1 (A)  Image missing alt text

Which issue would you like to fix? (enter number, "all", or "skip")
```

Non-fixable violations (e.g. color contrast requiring CSS changes outside
the component, or heading hierarchy requiring structural refactor) are listed
separately with a note that they require manual intervention.

### 2. Fix Type: Missing Alt Text — WCAG 1.1.1

When the violation is a missing or empty `alt` attribute on `<img>`:

1. Read the image `src` path and strip the file extension and path prefix to
   derive a candidate description. Examples:
   - `src="/images/hero-banner.png"` → candidate: `"Hero banner"`
   - `src="/icons/close.svg"` → candidate: `"Close"` (but prompt: is this decorative?)
2. Check surrounding context: `<figure>` with `<figcaption>`, parent element
   text, nearest heading. Include context in the suggestion.
3. Present the suggested alt text:
   ```
   FILE:    src/components/Hero.tsx
   LINE:    7
   Before:  <img src="/images/hero-banner.png" className="hero">
   After:   <img src="/images/hero-banner.png" className="hero" alt="Hero banner showing the product dashboard">

   Suggested alt: "Hero banner showing the product dashboard"
   Is this accurate? (y to apply / type replacement / n to skip)
   ```
4. If the user responds with replacement text, substitute it into the After
   line before applying.
5. Do not apply until the user confirms with "y" or a replacement string.

### 3. Fix Type: Missing Form Label — WCAG 1.3.1

When the violation is an `<input>`, `<textarea>`, or `<select>` without an
associated label:

1. Read the input's `name`, `id`, and `placeholder` attributes to infer
   a label string. If `id` exists but no label, generate the label element.
   If no `id` exists, generate one from the `name` attribute.
2. Present the fix:
   ```
   FILE:    src/components/ContactForm.jsx
   LINE:    18
   Before:
     <input type="email" name="email" placeholder="Enter email" />

   After:
     <label htmlFor="email">Email address</label>
     <input id="email" type="email" name="email" placeholder="Enter email" />

   Apply this fix? (y / type alternative label text / n to skip)
   ```
3. For React/JSX files use `htmlFor`; for HTML files use `for`.
4. If the input already has an `id` that matches a label elsewhere in the
   file, flag this as a mismatch rather than adding a duplicate label.

### 4. Fix Type: Missing aria-label — WCAG 4.1.2

When the violation is a button or link without accessible text:

1. Read the element's children. If an SVG `<title>` child exists, suggest
   promoting it to `aria-label`. If a class name hints at purpose (e.g.
   `btn-close`, `icon-search`), derive a label.
2. Present the fix:
   ```
   FILE:    src/components/Toolbar.tsx
   LINE:    42
   Before:
     <button className="btn-close">
       <svg aria-hidden="true" viewBox="0 0 24 24"><path d="M6 18L18 6M6 6l12 12"/></svg>
     </button>

   After:
     <button className="btn-close" aria-label="Close">
       <svg aria-hidden="true" viewBox="0 0 24 24"><path d="M6 18L18 6M6 6l12 12"/></svg>
     </button>

   Suggested aria-label: "Close"
   Apply? (y / type replacement / n to skip)
   ```

### 5. Apply the Fix

After user confirmation:
1. Read the target file.
2. Apply the minimal surgical change (only the flagged line and its
   immediate context — do not reformat surrounding code).
3. Write the file.
4. Confirm: `Fixed: src/components/Hero.tsx:7 — alt text added.`
5. Move to the next violation in the queue.

### 6. Session Summary

After all violations are processed:

```
=== FIX SESSION SUMMARY ===
Applied:  3
Skipped:  1
Remaining manual: 2 (see audit report for details)
```
