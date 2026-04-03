---
name: accessibility-audit
description: >
  Audit any web application for WCAG 2.1 Level AA accessibility compliance
  using Playwright MCP. Tests keyboard navigation, ARIA semantics, focus
  management, form labels, heading hierarchy, and dynamic content. No docs
  needed — WCAG is the standard. Use when user says "accessibility audit",
  "a11y check", "WCAG", "is this accessible", "screen reader test",
  "keyboard navigation test", "can a blind user use this", "Section 508",
  "ADA compliance", "focus management", "ARIA review", or "inclusive
  design check". Also trigger on "accessibility" alone.
---

# Accessibility Audit (WCAG 2.1 AA)

You audit web applications against WCAG 2.1 Level AA using Playwright MCP
to interact with the real page. WCAG is the oracle — violations are
objective, not opinion.

**First**, read: `qa-common/references/qa-reference.md (in this plugin)`

---

## Setup

1. **URL** (default: `http://localhost:3000`)
2. **Scope**: full site or specific pages?
3. **Target level**: A / AA (default) / AAA?

---

## Audit Checklist

### 1. Document Structure (1.3.1, 1.3.2, 2.4.1, 2.4.2, 2.4.6)
- `<html lang="...">` set and correct
- One `<h1>` per page, heading hierarchy h1→h2→h3 (no skips)
- Landmarks: `<main>`, `<nav>`, `<header>`, `<footer>` present
- Page `<title>` unique and descriptive per page
- Skip-to-content link as first Tab stop
- No duplicate IDs in the DOM
- Reading order matches visual order

### 2. Images & Media (1.1.1, 1.2.1-1.2.5, 1.4.5)
- All `<img>` have `alt` attributes
- Decorative images: `alt=""` or `role="presentation"`
- No alt text starting with "image of" (redundant)
- Complex images (charts, diagrams) have extended description
- Video/audio: captions or transcripts
- No text as images (use real text)

### 3. Keyboard Navigation (2.1.1, 2.1.2, 2.4.3, 2.4.7)

Test ENTIRELY with keyboard — no mouse:

- **Tab through everything**: every interactive element reachable?
- **Focus order**: matches visual layout?
- **Focus visible**: clear outline/ring on every focused element?
- **No keyboard traps**: can Tab out of every component?
- **Activation**: Enter for buttons/links, Space for toggles
- **Arrow keys**: work within radio groups, tabs, menus, selects
- **Escape**: closes modals, dropdowns, popups

Record the full Tab order:
```
| Tab # | Element              | Visible Focus | Correct Action |
|-------|----------------------|---------------|----------------|
| 1     | Skip to content      | Yes           | Yes            |
| 2     | Logo link            | Yes           | Yes            |
| 3     | Search input         | No ⚠          | N/A            |
```

### 4. Forms (1.3.1, 3.3.1, 3.3.2, 3.3.3, 4.1.2)
- Every `<input>` has a visible `<label>` (NOT just placeholder)
- Required fields: `aria-required="true"` or `required` attribute
- Error messages associated via `aria-describedby`
- Error state: not conveyed by color alone (icon or text too)
- Fieldsets group related controls with `<legend>`
- Autocomplete attributes on name/email/address/phone fields

### 5. ARIA & Semantics (4.1.1, 4.1.2)
- Interactive elements have accessible names
- `role` attributes correct and non-redundant
- `aria-expanded` updates on accordions/dropdowns
- `aria-selected` updates on tabs
- `aria-checked` updates on custom checkboxes
- No ARIA attributes on elements that don't need them

### 6. Dynamic Content (4.1.3)
- Toast notifications use `aria-live="polite"`
- Error alerts use `aria-live="assertive"` or `role="alert"`
- Loading states: `aria-busy="true"` on updating regions
- Content changes announced to screen readers

### 7. Modal/Dialog Management (2.4.3)
- Focus moves INTO modal on open
- Tab is TRAPPED within modal (can't Tab behind it)
- Escape closes the modal
- Focus returns to trigger element on close
- Modal has `role="dialog"` and `aria-modal="true"`

### 8. Visual (1.4.1, 1.4.3, 1.4.4, 1.4.10, 1.4.12)
- Color contrast: 4.5:1 normal text, 3:1 large text (18px+)
- Information not by color alone (error = red + icon + text)
- Content readable at 200% zoom (1.4.4)
- Reflow at 320px: no horizontal scroll (1.4.10)
- Touch targets: minimum 24x24px (2.5.8), ideal 44x44px

---

## Report

Save to `qa/reports/{project}/accessibility/run-[date].md`:

```
## Accessibility Audit — [URL]
### WCAG 2.1 Level AA | Date: [date]

### Score: [X/100]
### Verdict: [Pass / Partial / Fail]

### Summary
| Category          | Issues | Critical | Major | Minor |
|-------------------|--------|----------|-------|-------|
| Document structure| 2      | 0        | 1     | 1     |
| Images & media    | 1      | 0        | 1     | 0     |
| Keyboard nav      | 4      | 1        | 2     | 1     |
| Forms             | 3      | 0        | 2     | 1     |
| ARIA & semantics  | 2      | 0        | 1     | 1     |
| Dynamic content   | 1      | 0        | 1     | 0     |
| Modals/dialogs    | 2      | 1        | 1     | 0     |
| Visual            | 1      | 0        | 0     | 1     |
| **Total**         | **16** | **2**    | **9** | **5** |

### Issues

#### [CRITICAL] 2.1.2 — Keyboard trap in date picker
- **Principle**: Operable
- **Location**: /checkout — delivery date field
- **Problem**: After tabbing into the date picker, Tab and Escape
  do not exit the component. User is stuck.
- **Fix**: Add Escape handler to close picker and return focus.
  Ensure Tab moves to next form field.
- **Evidence**: qa/screenshots/A11Y-001.png

(continue...)

### Passing Criteria
- [PASS] 1.1.1 Non-text Content — all images have alt text
- [PASS] 2.1.1 Keyboard — all elements reachable
- [FAIL] 2.1.2 No Keyboard Trap — date picker traps focus
- [PASS] 2.4.1 Bypass Blocks — skip-to-content present
(continue...)

### Keyboard Navigation Map
(from Section 3)

### Recommendations
1. [Fix keyboard traps — blocks users entirely]
2. [Add form labels — affects form completion]
3. ...
```

### After the Report

- "Want me to **create Jira tickets** for the accessibility issues?" → `jira-create-bug`
  (Critical/Major issues as bugs; Minor as improvements with "accessibility" label)

---

## Rules
- Cite specific WCAG success criteria (e.g., 2.1.2) for every issue
- Suggest concrete code fixes, not vague advice
- Test keyboard navigation on EVERY page, not just home
- Prioritize: blocks users > confuses users > suboptimal
- Don't assume framework handles accessibility — verify in the real DOM
- **Always close the browser** (`browser_close`) after the session is complete
