---
name: a11y-audit
description: >
  Static accessibility audit on HTML, JSX/TSX, and Vue SFC files.
  Detects WCAG 2.1 violations, reports file and line number, assigns severity,
  and provides a suggested fix with a code snippet.
triggers:
  - audit accessibility
  - check wcag
  - a11y scan
  - run accessibility audit
---

# a11y-audit

Run a full static accessibility audit on one or more source files.

## Supported Input Formats

- Plain HTML files (`.html`, `.htm`)
- React JSX / TSX components (`.jsx`, `.tsx`)
- Vue Single-File Components (`.vue`)

## Steps

### 1. Accept Input

Collect the file list from the user. If no specific files are given, scan every
file matching `src/**/*.{html,htm,jsx,tsx,vue}`. For each file, read its full
contents and parse it into a line-indexed array so every finding can carry a
precise line number.

### 2. Detect WCAG Violations (Static Scan)

Run each check below against every file. Apply checks independently so a
single element can produce multiple findings.

#### 2.1 Images Without Alt — WCAG 1.1.1 (Level A)

Flag every `<img>` element that:
- Has no `alt` attribute at all.
- Has `alt=""` and does **not** carry `role="presentation"` or `role="none"`.

If the image is inside a `<figure>` with a `<figcaption>`, note that in the
suggestion — the alt text may duplicate the caption.

#### 2.2 Buttons and Links Without Accessible Text — WCAG 4.1.2 (Level A)

Flag every `<button>`, `<a>`, or element with `role="button"` or `role="link"`
that satisfies **all** of the following:
- No visible text content (ignoring whitespace).
- No `aria-label` attribute.
- No `aria-labelledby` attribute.
- No `<title>` child element.

For icon-only buttons (e.g. `<button><svg …/></button>`), suggest adding
`aria-label`.

#### 2.3 Form Inputs Without Associated Label — WCAG 1.3.1 (Level A)

Flag every `<input>`, `<textarea>`, and `<select>` that:
- Has no `<label>` element with a matching `for` attribute.
- Is not wrapped by a `<label>`.
- Has no `aria-label` or `aria-labelledby` attribute.

Exempt: `type="hidden"`, `type="submit"`, `type="button"`, `type="reset"`.

#### 2.4 Color Contrast (Inline Styles) — WCAG 1.4.3 (Level AA)

For every element with an inline `style` containing both `color` and
`background-color` / `background`:

1. If both values are concrete (`#rgb`, `#rrggbb`, `rgb()`, `rgba()`, named
   CSS colors), compute the luminance-based contrast ratio using:
   ```
   L = 0.2126*R + 0.7152*G + 0.0722*B  (linearized sRGB)
   ratio = (L_lighter + 0.05) / (L_darker + 0.05)
   ```
2. Flag if ratio < 4.5:1 (normal text) or < 3:1 (large text ≥18pt / ≥14pt bold).
3. If either value is a CSS variable or `inherit`/`currentColor`, flag for
   **manual review** — not an automatic violation.

#### 2.5 Missing `lang` Attribute — WCAG 3.1.1 (Level A)

Flag every `<html>` element that does not carry a `lang` attribute with a
valid BCP 47 language tag (e.g. `en`, `en-US`, `tr`, `fr-CA`).

In Vue SFCs check the `<template>` root; in JSX check the top-level `<html>`
in render output if present.

#### 2.6 Heading Hierarchy Violations — WCAG 1.3.1 (Level A)

Collect every heading (`<h1>`–`<h6>`) in document order. Flag a heading when
its level skips more than one step from the previous heading.

| Previous | Current | Result |
|----------|---------|--------|
| h1 | h3 | VIOLATION (skips h2) |
| h2 | h4 | VIOLATION (skips h3) |
| h3 | h2 | OK (going up is fine) |

#### 2.7 Interactive Elements Without Focus Styles — WCAG 2.4.7 (Level AA)

In `<style>` blocks or CSS-in-JS template literals, flag if:
- `outline: none` or `outline: 0` is set on `a`, `button`, `input`,
  `select`, `textarea`, or `[tabindex]` without a replacement visible
  focus style on `:focus` or `:focus-visible` (`box-shadow`, `border`, etc.).

In JSX/TSX also flag `style={{ outline: 'none' }}` or event handlers that
strip focus styles dynamically.

#### 2.8 ARIA Role Mismatches — WCAG 4.1.2 (Level A)

Flag elements with semantically inappropriate or redundant `role` values:

| Element | Invalid / Redundant Role |
|---------|--------------------------|
| `<a href="…">` | `role="button"` (warn: remove href or handle keyboard) |
| `<button>` | `role="link"` |
| `<input type="checkbox">` | `role="radio"` |
| `<input type="radio">` | `role="checkbox"` |
| `<nav>` | `role="banner"` |
| `<header>` | `role="navigation"` |
| `<main>` | `role="complementary"` |
| `<nav>` | `role="navigation"` (redundant) |
| `<button>` | `role="button"` (redundant) |

### 3. Per-Violation Output Format

For every finding emit a structured record:

```
FILE:        path/to/Button.tsx
LINE:        42
WCAG:        4.1.2
SEVERITY:    A
VIOLATION:   Button without accessible text
DESCRIPTION: <button> has no visible text, no aria-label, and no aria-labelledby.
             Screen readers will announce "button" with no context.
FIX:
  Before:  <button><svg aria-hidden="true" …/></button>
  After:   <button aria-label="Close dialog"><svg aria-hidden="true" …/></button>
```

Group findings by file. Sort within each file by line number ascending.

### 4. Summary Block

After all findings, emit:

```
=== A11Y AUDIT SUMMARY ===
Total issues:  12
  Level A:     8
  Level AA:    3
  Level AAA:   1
Most common:   Missing alt text (4 occurrences)
Files scanned: 7
Files clean:   2
```

If zero violations are found, output:
```
=== A11Y AUDIT SUMMARY ===
No violations found. All scanned files pass WCAG 2.1 Level AA.
```
