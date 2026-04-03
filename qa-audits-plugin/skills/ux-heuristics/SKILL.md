---
name: ux-heuristics
description: >
  Evaluate any web application's usability against Jakob Nielsen's 10
  heuristic principles using Playwright MCP. No documentation needed —
  the heuristics ARE the evaluation framework. Use when user says "UX
  audit", "usability review", "how usable is this", "UX check", "is
  this user-friendly", "Nielsen heuristics", "heuristic evaluation",
  "review the UX", "usability test", "is this intuitive", or "UX
  problems". Also trigger on "how would a user feel using this" or
  "would a normal person understand this".
---

# UX Heuristic Evaluation (Nielsen's 10)

You are a UX expert evaluating a web application against Jakob Nielsen's
10 usability heuristics. Each heuristic is a proven principle — violations
are usability problems regardless of what any spec says.

**First**, read: `qa-common/references/qa-reference.md (in this plugin)`

---

## Setup

1. **URL** (default: `http://localhost:3000`)
2. **Scope**: entire app or specific flow?
3. **User persona**: novice / intermediate / expert? (default: novice —
   harshest test of usability)

---

## Evaluation Process

Navigate every page via Playwright MCP. For each heuristic, evaluate
every screen and interaction you encounter.

### H1: Visibility of System Status
The system should keep users informed about what's going on.

Check:
- After clicking a button: is there immediate visual feedback?
  (loading spinner, button state change, toast notification)
- During data loading: skeleton screens, progress bars, or spinners?
- After form submission: clear success/failure message?
- Current location: is the active page highlighted in navigation?
  Breadcrumbs present on deep pages?
- Pending actions: does the UI show unsaved changes?
- Long operations: progress indication or estimated time?

**Common violations**: button looks the same after click, no loading state
during API calls, no confirmation after save, unclear which nav item is active.

### H2: Match Between System and Real World
Use language and concepts familiar to the user, not developer jargon.

Check:
- Labels and terminology: would a non-technical person understand every label?
- Icons: are they universally recognizable or ambiguous?
- Metaphors: do they match real-world expectations?
  (trash can = delete, magnifying glass = search)
- Data formats: dates, currencies, numbers in locale-appropriate format?
- Logical ordering: menus, options, steps in natural sequence?

**Common violations**: technical error messages ("Error 500", "null reference"),
ambiguous icons without labels, unnatural sort order, ISO date formats.

### H3: User Control and Freedom
Users need a clear "emergency exit" from unwanted states.

Check:
- Can you undo recent actions? (delete → undo, edit → cancel)
- Back button works correctly? (no broken history, no re-submission)
- Modals have a close button AND Escape key closes them?
- Multi-step flows have a "go back" option at each step?
- Can you cancel a process mid-way? (upload, checkout, wizard)
- No forced paths — can the user navigate away freely?

**Common violations**: no undo for delete, no cancel button on forms,
modal without close/Escape, multi-step wizard with no back option.

### H4: Consistency and Standards
Same actions, same appearance, same result throughout the product.

Check:
- Are similar actions named the same way? (Save/Submit/Confirm — pick one)
- Do buttons with the same role look the same across pages?
- Is the layout pattern consistent? (headers, sidebars, content areas)
- Do links look different from buttons? Buttons from plain text?
- Error messages use the same tone and format everywhere?
- Follow platform conventions: underlined links, pointer cursor on clickables,
  standard form patterns

**Common violations**: "Save" on one page, "Submit" on another; primary
button is blue on one page, green on another; inconsistent error display.

### H5: Error Prevention
Design to prevent errors rather than just handling them.

Check:
- Destructive actions require confirmation? (delete, discard, overwrite)
- Input fields have appropriate constraints? (type=email, type=number,
  maxlength, datepicker vs free text for dates)
- Disable submit button when form is invalid?
- Default values for common choices? (country, currency, timezone)
- Search with suggestions/autocomplete to prevent typos?
- Clear formatting hints near inputs? ("DD/MM/YYYY", "min 8 characters")

**Common violations**: delete without confirmation, free-text date field,
no input masks, submit enabled with empty required fields.

### H6: Recognition Rather Than Recall
Minimize memory load — show options, don't make users remember them.

Check:
- Are instructions visible when needed? (not hidden behind a tooltip)
- Search history / recent items visible?
- Form labels always visible? (not just placeholder that disappears)
- Complex actions have contextual help or examples?
- Dashboard shows relevant data without requiring navigation?
- Related actions grouped visually?

**Common violations**: placeholder-only labels that vanish on focus,
complex filters with no saved/recent options, no contextual help.

### H7: Flexibility and Efficiency of Use
Accommodate both novice and expert users.

Check:
- Keyboard shortcuts for frequent actions?
- Search/filter for long lists instead of scrolling?
- Bulk actions available? (select all, batch delete)
- Shortcuts visible but non-intrusive for novices?
- Customizable or configurable where appropriate?
- Touch-friendly on mobile + precise on desktop?

**Common violations**: no keyboard shortcuts, no search in long dropdowns,
no bulk operations, one-size-fits-all interface.

### H8: Aesthetic and Minimalist Design
Every extra element competes with relevant information.

Check:
- Is there visual clutter? (too many CTAs, excessive borders, dense text)
- Is the visual hierarchy clear? (what to look at first, second, third)
- White space used effectively?
- Only relevant information on screen? (no "just in case" content)
- Visual noise: excessive decorations, animations, competing colors?
- Information density appropriate for the context?

**Common violations**: overloaded dashboards, walls of text, competing
call-to-action buttons, unnecessary decorative elements.

### H9: Help Users Recognize, Diagnose, Recover from Errors
Error messages should be clear, specific, and constructive.

Check:
- Error messages in plain language? (not codes or technical jargon)
- Error messages say what went wrong AND what to do about it?
- Error messages appear near the problem? (inline vs page-top banner)
- Form errors highlight the specific field?
- Error state is visually distinct? (red border, icon, not just color)
- After an error, is user input preserved? (form not cleared)

**Common violations**: "Something went wrong", "Error 422", error banner
far from the field, form cleared after validation failure.

### H10: Help and Documentation
Even if the system is self-explanatory, help should be available.

Check:
- Help/FAQ accessible from the interface?
- Tooltips on complex features?
- Onboarding or first-use guidance?
- Empty states have helpful guidance? ("No items yet — click + to add")
- Error recovery instructions available?
- Contact/support option visible?

**Common violations**: no empty state guidance (just blank page), no
tooltips on icons, no onboarding, no help link.

---

## Scoring

Rate each heuristic: 0-4 (Nielsen severity scale)

| Score | Meaning |
|-------|---------|
| 0 | Not a usability problem |
| 1 | Cosmetic — fix if time allows |
| 2 | Minor — low priority fix |
| 3 | Major — important to fix, high priority |
| 4 | Catastrophe — must fix before release |

---

## Report

Save to `qa/reports/{project}/ux-heuristics/run-[date].md`:

```
## UX Heuristic Evaluation — [URL]
### Date: [date] | Persona: [novice/intermediate/expert]

### Scorecard
| # | Heuristic                           | Score | Issues |
|---|-------------------------------------|-------|--------|
| 1 | Visibility of system status         | 2     | 3      |
| 2 | Match with real world               | 1     | 1      |
| 3 | User control and freedom            | 3     | 4      |
| 4 | Consistency and standards            | 2     | 5      |
| 5 | Error prevention                    | 3     | 3      |
| 6 | Recognition over recall             | 1     | 2      |
| 7 | Flexibility and efficiency          | 2     | 2      |
| 8 | Aesthetic and minimalist design     | 1     | 1      |
| 9 | Error recognition and recovery      | 3     | 4      |
| 10| Help and documentation              | 2     | 2      |
|   | **Average**                         | **2.0**|**27** |

### Overall UX Rating: [Excellent / Good / Needs Work / Poor]

### Issues (sorted by severity)

#### [Severity 4] H3 — No undo after delete
- **Heuristic**: User control and freedom
- **Location**: /todos — delete button
- **Problem**: Clicking delete immediately removes the item
  with no undo option and no confirmation dialog
- **Impact**: Users who accidentally click delete lose data permanently
- **Recommendation**: Add confirmation dialog or undo toast (5s window)
- **Evidence**: qa/screenshots/UX-H3-001.png

(continue for each issue...)

### Top 5 Improvements (highest impact)
1. ...
```

### After the Report

- "Want me to **create Jira tickets** for the UX issues found?" → `jira-create-bug`
  (issues with severity 3-4 mapped as bugs; severity 1-2 as improvements)

---

## Rules
- Evaluate as the specified persona — novice finds more issues
- Score honestly — don't inflate or deflate
- Every issue needs a concrete recommendation
- Screenshot each issue
- Compare across pages — inconsistency is itself a violation (H4)
- **Always close the browser** (`browser_close`) after the session is complete
