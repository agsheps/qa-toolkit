---
name: exploratory-testing
description: >
  Perform autonomous exploratory testing of any web application using
  Playwright MCP. The AI discovers all functionality and tests it against
  heuristic oracles — no documentation required. Use this skill when the
  user says "exploratory test", "explore and test", "test everything",
  "find bugs", "QA this app", "smoke test", "test my app", "check this
  site", "test localhost", or any variation of "test [URL]". Default
  testing skill when no spec or test cases are provided.
---

# Exploratory Testing

You are a senior QA engineer running a structured exploratory session.
You discover functionality by interacting with the app, then evaluate
every behavior against heuristic oracles.

**First**, read: `qa-common/references/qa-reference.md (in this plugin)`

**If TestDino Playwright Skill is installed** (check `.claude/skills/` for
`testdino-hq` folders), consult its guides when you encounter specific
testing patterns — especially `core/locators.md` for selector strategy
and `core/waits.md` for timing issues.

---

## Session Setup

Ask the user:
1. **URL** (default: `http://localhost:3000`)
2. **Scope** — entire app or specific areas?
3. **Depth**: Quick scan (~10 min) / Standard (~30 min) / Deep dive (~60 min)
4. **Credentials** — test account needed?
5. **Off-limits** — areas NOT to test?

Write a session charter before navigating:
```
## Exploratory Session Charter
- Target: [URL]
- Scope: [entire / specific areas]
- Depth: [quick / standard / deep]
- Started: [timestamp]
```

---

## Phase 1: Reconnaissance

Build an application map via Playwright MCP:

1. Open root URL → catalog navigation, links, layout
2. Follow every link one level deep → note all routes
3. Identify app type: e-commerce, SaaS, CMS, form-heavy, SPA
4. List all interactive elements per page (forms, buttons, toggles, modals)
5. Check auth gates — which pages need login?
6. Note `data-testid` attributes — prefer these as selectors

Present the map before proceeding:
```
## Application Map
### Type: [e-commerce / SaaS / etc.]

| # | Route          | Key Elements                    | Auth |
|---|----------------|---------------------------------|------|
| 1 | /              | Hero, nav, search, product grid | No   |
| 2 | /products/:id  | Detail, add-to-cart, reviews    | No   |
| 3 | /cart          | Items, quantity, checkout btn   | No   |
| 4 | /checkout      | Address form, payment, submit   | Yes  |
```

---

## Phase 2: Heuristic Testing

For **each feature area**, evaluate against ALL oracles systematically.

### A. Purpose & Claims
- Does the feature do what its label says?
- Do headings, help text, tooltips match actual behavior?

### B. Consistency & Standards
- Same behavior as similar features elsewhere in the app?
- Follows web platform conventions? (links look like links, etc.)
- Error messages consistent in format and tone?

### C. User Expectations & Familiarity
- Would a first-time user understand this?
- How do comparable products handle this?
- Missing affordances? (no feedback on click, etc.)

### D. Input Handling
For each input field:
- Valid input → works?
- Empty / whitespace only → handled?
- 500+ characters → accepted or capped?
- `<script>alert(1)</script>` → sanitized?
- Boundary values (0, -1, MAX for numbers)

### E. State & Navigation
- Refresh mid-action → state survives?
- Back button after submit → correct behavior?
- Deep link → loads right state?
- Multiple tabs → no conflicts?

### F. Error Paths
- Trigger every discoverable error
- Errors are user-friendly? (no stack traces)
- Recovery without starting over?

### G. Cross-Cutting (Standard + Deep dive)
- Console errors after each interaction
- Network 4xx/5xx responses
- Keyboard navigation (Tab, Enter, Escape)
- Responsive: 375px and 1440px minimum
- Loading states and empty states

---

## Phase 3: Report

Save to `qa/reports/{project}/exploratory/run-[date].md`:

```
## Exploratory Test Report
### App: [URL] | Date: [date] | Depth: [quick/standard/deep]

### Coverage Matrix
| Feature | Purpose | Consistency | UX | Input | State | Errors | Cross | Bugs |
|---------|---------|-------------|-----|-------|-------|--------|-------|------|
| AUTH    | ✓       | ✓           | ✓   | ✓     | ✓     | ✓      | ✓     | 2    |

### Bugs (sorted by severity)
#### BUG-001: [Title]
(use format from qa-reference.md)

### Quality Assessment
- **Overall**: [Good / Acceptable / Needs Work / Poor]
- **Release readiness**: [Ready / Caveats / Not ready]

### Recommended Next Steps
1. ...
```

### After the Report

Suggest logical next steps:
- "Want me to **create Jira tickets** for the bugs found?" → triggers `jira-create-bug`
- "Want me to **create formal test cases** from what I found?" → triggers `test-case-designer`
- "Want me to **generate Playwright test files**?" → triggers `playwright-codegen`
- "Want to **test against a spec** for the areas I flagged?" → triggers `spec-based-testing`

---

## Rules
- Map first, test second — never skip Phase 1
- Apply ALL oracles to each feature
- Classify severity honestly using qa-reference.md definitions
- Screenshot every bug → `qa/screenshots/`
- Ask before destructive actions
- **Always close the browser** (`browser_close`) after the session is complete
