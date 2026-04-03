---
name: test-case-designer
description: >
  Design formal, reusable test cases for any web app. Two modes: WITH docs
  (derive from requirements) or WITHOUT docs (explore via Playwright MCP
  and baseline current behavior). Outputs structured YAML consumed by
  regression-testing and playwright-codegen skills. Use when user says
  "create test cases", "write test cases", "design tests", "build test
  suite", "generate test plan", "prepare for regression", "document tests",
  "what should we test", "define acceptance tests", or "prepare test plan".
---

# Test Case Designer

You create structured, reusable test cases precise enough to be re-executed
by another tester (human or AI) with identical results.

**First**, read: `qa-common/references/qa-reference.md (in this plugin)`

---

## Step 1: Determine Mode

**Mode A — With Documentation**: user provides spec/stories/PRD.
Each requirement → one or more test cases.

**Mode B — Without Documentation**: no spec available.
Explore app via Playwright MCP, baseline current behavior.
Flag suspicious behavior with `baseline_suspect: true`.

---

## Step 2: Gather Inputs

1. **URL** (default: `http://localhost:3000`)
2. **Documentation** (Mode A): pasted text, file, URL, or verbal
3. **Coverage depth**:
   - **Smoke** (~10-20 cases): critical paths only
   - **Standard** (~30-60 cases): all features, happy + negative
   - **Comprehensive** (~60-100+ cases): edge cases, a11y, responsive
4. **Credentials** if needed

---

## Step 3: Plan the Test Suite

### Mode A: Derive from Docs
For each requirement:
- Positive test (works as specified)
- Negative test (handles invalid input)
- Boundary test (edge cases)

### Mode B: Discover from App
1. Navigate entire app via Playwright MCP
2. Build feature map (same as exploratory-testing Phase 1)
3. For each feature: document current behavior as "expected"
4. Create positive, negative, boundary tests

### Present plan for confirmation:
```
## Test Plan
### App: [URL] | Mode: [Docs/Exploration] | Coverage: [Smoke/Std/Comprehensive]

| Module   | Description          | Positive | Negative | Boundary | Total |
|----------|----------------------|----------|----------|----------|-------|
| AUTH     | Login, register      | 4        | 3        | 2        | 9     |
| CART     | Add, remove, qty     | 4        | 3        | 2        | 9     |
| Total    |                      | 8        | 6        | 4        | 18    |

Proceed?
```

---

## Step 4: Create Test Cases

Save to `qa/test-cases/[MODULE].yaml` — one file per module:

```yaml
module: AUTH
description: "Authentication — login, registration, password management"
app_url: "http://localhost:3000"
created: "2026-04-01"
source: "specification" | "exploration"

test_cases:

  - id: TC-AUTH-001
    title: "Successful login with valid credentials"
    type: positive
    priority: P0
    tags: [smoke, happy-path]
    preconditions:
      - "Test account exists: test@example.com / TestPass123"
      - "User is logged out"
    steps:
      - step: 1
        action: "Navigate to /login"
        expected: "Login page loads with email and password fields"
      - step: 2
        action: "Enter 'test@example.com' in email field"
        expected: "Email field accepts input"
      - step: 3
        action: "Enter 'TestPass123' in password field"
        expected: "Password field shows masked input"
      - step: 4
        action: "Click 'Log in' button"
        expected: "Redirect to /dashboard. Welcome message visible."
    cleanup: null

  - id: TC-AUTH-002
    title: "Login rejected with wrong password"
    type: negative
    priority: P1
    tags: [security]
    preconditions:
      - "Test account exists: test@example.com / TestPass123"
    steps:
      - step: 1
        action: "Navigate to /login"
        expected: "Login page loads"
      - step: 2
        action: "Enter 'test@example.com' in email, 'wrongpass' in password"
        expected: "Fields accept input"
      - step: 3
        action: "Click 'Log in'"
        expected: "Error: 'Invalid email or password'. Stay on /login."
    cleanup: null
```

### Writing Rules

- `action` — exact user action with specific values
- `expected` — observable result verifiable by reading the page
- BAD: "Fill in the form" → GOOD: "Enter 'John' in 'First name' field"
- BAD: "Page updates correctly" → GOOD: "Counter shows '3 items'"
- One scenario per test case, deterministic steps

### Tags for filtering
`smoke` `happy-path` `validation` `security` `responsive` `a11y`
`destructive` `integration` `boundary`

---

## Step 5: Dry Run

Execute every test case via Playwright MCP to verify:
1. Steps are unambiguous
2. Elements actually exist
3. Expected results match current behavior

**Mode A discrepancy** = potential bug → flag it.
**Mode B discrepancy** = adjust expected result to match reality.

---

## Step 6: Deliver

```
## Test Suite Created
### App: [URL] | Mode: [Docs/Exploration] | Date: [date]

| File                    | Module | Cases | Smoke | P0 | P1 |
|-------------------------|--------|-------|-------|----|----|
| qa/test-cases/AUTH.yaml | AUTH   | 9     | 3     | 2  | 4  |
| qa/test-cases/CART.yaml | CART   | 12    | 4     | 3  | 5  |
| Total                   |        | 21    | 7     | 5  | 9  |

### Dry Run: [all passed / N adjusted]
```

### After Delivery — suggest next steps:

- "Run **regression tests** on these cases" → `regression-testing`
- "**Generate Playwright test code** from these cases" → `playwright-codegen`
- "Run the **Playwright Planner agent** to expand coverage" →
  `npx playwright test --agents planner --url [URL]`

---

## Rules
- One test = one scenario, deterministic steps
- Always dry-run before delivery
- Never change test case IDs after creation
- Ask before assuming ambiguous requirements
- **Always close the browser** (`browser_close`) after the session is complete
