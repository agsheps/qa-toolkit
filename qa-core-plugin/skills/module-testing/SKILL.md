---
name: module-testing
description: >
  Deep-test a single module or feature area of a web application after
  changes, without documentation. Focuses all effort on one area: full
  functional check, input validation, edge cases, state management, and
  then verifies integration boundaries with adjacent modules haven't
  broken. Use when user says "test this module", "test the cart",
  "check the login page", "test just the checkout", "we changed the
  search — test it", "test this feature", "verify this page", "test
  only [area]", "changes were made to [area] — check it", "regression
  on [module]", "smoke test [feature]", or names any specific page,
  flow, or component to test. Also trigger when user pastes a URL to
  a specific page (not root) and says "test this".
---

# Module Testing (Focused Deep-Dive)

You deeply test a single module or feature area of a web application.
All testing effort is concentrated on ONE area — but you also verify
that integration points with adjacent modules still work.

Think of it as: 80% deep on the target module, 20% smoke on its neighbors.

**First**, read: `qa-common/references/qa-reference.md (in this plugin)`

---

## Setup

1. **URL** to the module (e.g., `localhost:3000/cart`, `localhost:3000/settings`)
   If the user names a feature ("test the search"), navigate to find it.
2. **What changed?** — ask the user:
   - Specific changes? ("we updated the filter logic")
   - General changes? ("some work was done on search")
   - Unknown? ("just check it, something might have broken")
3. **Credentials** if the module requires auth

---

## Phase 1: Module Reconnaissance (fast, 2-3 min)

Navigate to the target module via Playwright MCP and build a focused map:

```
## Module Map: [MODULE NAME]
### Entry point: [URL]

### Elements
| Element              | Type     | Selector / testid     |
|----------------------|----------|-----------------------|
| Search input         | Input    | data-testid="search"  |
| Filter dropdown      | Select   | role="combobox"       |
| Results grid         | List     | data-testid="results" |
| Pagination           | Nav      | nav[aria-label="..."] |
| Sort toggle          | Button   | data-testid="sort"    |

### States identified
- Default state (loaded with data)
- Empty state (no results)
- Loading state (during fetch)
- Error state (if discoverable)
- Filtered state (with active filters)

### Adjacent modules (integration points)
| Direction | Module      | Connection                          |
|-----------|-------------|-------------------------------------|
| Inbound   | Navigation  | Link from main nav to this page     |
| Inbound   | Product page| "Back to results" link              |
| Outbound  | Product page| Click on result → product detail    |
| Outbound  | Cart        | "Add to cart" button on results     |
| Data      | API         | GET /api/products?q=...             |
```

---

## Phase 2: Deep Functional Testing

### A. Core Functionality (all happy paths)

Test every action the module supports:
- Perform each primary action with valid input
- Verify the result is correct and visible
- Verify state updates properly (counters, totals, selections)
- Verify persistence — refresh the page, is state preserved?

### B. Input Testing (every input field in the module)

For each input:
| Test | Action | Check |
|------|--------|-------|
| Empty | Submit with nothing | Error message? Handled gracefully? |
| Whitespace | `"   "` (spaces only) | Treated as empty or accepted? |
| Minimum valid | Shortest valid input | Works correctly? |
| Maximum | 500+ characters | Accepted, truncated, or rejected? |
| Special chars | `<script>`, `& < > "` | Escaped or rendered as HTML? |
| Type mismatch | Text in number field, etc. | Prevented or caught? |
| Boundary | 0, -1, MAX for numbers | Correct handling? |
| Copy-paste | Paste formatted text | Stripped or preserved? |

### C. State Transitions

Map and test every state change the module supports:

```
[Default] → action → [New State] → verify → [correct?]

Example for a search module:
  Empty page → type query → Loading → Results shown
  Results → click filter → Loading → Filtered results
  Results → clear query → Empty state
  Results → click result → Product page → back → Results (preserved?)
  Error → retry → Loading → Results
```

Test each transition. Verify:
- UI reflects the new state immediately
- No flash of incorrect state during transition
- Intermediate states (loading) are shown

### D. Edge Cases (specific to what changed)

If user told you what changed, focus edge cases there:
- If filter logic changed → test every filter combination
- If form was modified → test every field individually + together
- If display changed → test at all viewports
- If data flow changed → test with 0 items, 1 item, many items

If changes are unknown, test common edge cases:
- Zero data state (empty list, no results)
- Single item state
- Maximum data (long list, many items)
- Rapid repeated actions (double-click, rapid toggle)
- Browser back/forward through the module
- Deep link to a specific state (copy URL, open in new tab)
- Refresh during each state

### E. Negative Paths

- What happens on network error during a module action?
- What happens if you access the module without required data?
  (e.g., checkout without cart items)
- What happens if you manipulate URL parameters?
  (`?page=-1`, `?sort=invalid`, `?id=nonexistent`)
- What if another tab modifies the same data?

---

## Phase 3: Integration Boundary Smoke Test

For each adjacent module identified in Phase 1:

### Inbound connections (things that lead TO this module)
- Navigate from each entry point → module loads correctly?
- Data passed from the previous module is correct?
- Back button returns to the source correctly?

### Outbound connections (things this module leads TO)
- Click through to each destination → loads correctly?
- Data passed to the next module is correct?
  (cart item count, selected product, form data)
- Return from destination → module state preserved?

### Data dependencies
- If the module displays data from an API:
  - Is the data current? (not stale/cached when it shouldn't be)
  - Does it handle API errors gracefully?
- If the module modifies shared state (cart, user profile):
  - Is the change reflected in other modules? (header cart count, etc.)
  - Is the change persisted? (refresh, navigate away and back)

```
## Integration Smoke Results
| Connection                          | Status | Notes              |
|-------------------------------------|--------|--------------------|
| Nav → Module                        | PASS   |                    |
| Module → Product detail             | PASS   |                    |
| Module → Cart (add to cart)         | FAIL   | Count not updated  |
| Back from Product → Module          | PASS   | Scroll position OK |
| API error handling                  | WARN   | Generic error msg  |
| Shared state: cart count in header  | FAIL   | Stale after add    |
```

---

## Phase 4: Report

Save to `qa/reports/{project}/module/run-[date]-[NAME].md`:

```
## Module Test Report: [MODULE NAME]
### URL: [url] | Date: [date]
### Changes tested: [what user told you / "unknown — full check"]

### Summary
| Area                     | Pass | Fail | Warn | Total |
|--------------------------|------|------|------|-------|
| Core functionality       | X    | X    | X    | X     |
| Input validation         | X    | X    | X    | X     |
| State transitions        | X    | X    | X    | X     |
| Edge cases               | X    | X    | X    | X     |
| Negative paths           | X    | X    | X    | X     |
| Integration (inbound)    | X    | X    | X    | X     |
| Integration (outbound)   | X    | X    | X    | X     |
| **Total**                | X    | X    | X    | X     |

### Module Health: [Solid / Minor Issues / Needs Attention / Broken]
### Integration Health: [Solid / Minor Issues / Broken Connections]

### Bugs Found

#### BUG-001: Cart count in header not updated after add-to-cart
- **Severity**: S2 Major
- **Area**: Integration (outbound → Cart)
- **Steps**: Add item from search results → check header
- **Expected**: Cart badge shows "1"
- **Actual**: Cart badge still shows "0" until page refresh
- **Evidence**: qa/screenshots/BUG-001.png
- **Impact**: User thinks add-to-cart didn't work

(continue for each bug...)

### Regression Risk Assessment
- **Within module**: [Low / Medium / High] — [reasoning]
- **Impact on other modules**: [Low / Medium / High] — [reasoning]
- **Safe to deploy?**: [Yes / Yes with caveats / No]

### What Wasn't Tested
- [List anything you couldn't test and why]
- [E.g., "Payment processing — mocked in dev"]

### Recommendations
1. ...
```

### After the Report

- "Want me to **create Jira tickets** for the bugs found?" → `jira-create-bug`
- "Want me to **test adjacent modules** that showed issues?" → `module-testing` (for each affected module)
- "Want me to **create test cases** for this module?" → `test-case-designer`
- "Want me to **run a full exploratory** pass to check the rest?" → `exploratory-testing`
- "Want to **update existing regression cases** for this module?"

---

## Rules
- **Depth over breadth** — test this one module thoroughly
- **Always check integration boundaries** — isolated module testing misses
  the most common real-world bugs (data not flowing between modules)
- **Ask what changed** — it focuses edge case testing on the right area
- **Report regression risk** — tell the user whether it's safe to deploy
- **Track what you DIDN'T test** — explicit gaps prevent false confidence
- Screenshot every bug
- If you find a critical issue in an adjacent module, report it but don't
  go deep — suggest a focused test of that module separately
- **Always close the browser** (`browser_close`) after the session is complete
