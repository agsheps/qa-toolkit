---
name: regression-testing
description: >
  Execute existing test cases from qa/test-cases/ against a live web app
  via Playwright MCP. Compares with previous runs to detect regressions.
  Requires YAML test cases created by test-case-designer. Use when user
  says "run regression tests", "run test suite", "regression test",
  "execute test cases", "re-run tests", "check for regressions", "run
  smoke tests", "run P0 tests", "did anything break", "test after deploy",
  "verify release", "re-test", "run the suite", or "sanity check".
---

# Regression Testing

Execute pre-defined test cases via Playwright MCP and produce a PASS/FAIL
report. Compare results against prior runs to surface regressions.

**First**, read: `qa-common/references/qa-reference.md (in this plugin)`

---

## Step 1: Load Test Suite

Look for `qa/test-cases/*.yaml` in the project, or a user-specified path.

**If no test cases found** → tell the user:
"No test cases found. Say 'create test cases for [URL]' to generate them."

Display the loaded suite:
```
## Test Suite Loaded
| File            | Module | Total | Smoke | P0 | P1 |
|-----------------|--------|-------|-------|----|----|
| AUTH.yaml       | AUTH   | 9     | 3     | 2  | 4  |
| CART.yaml       | CART   | 12    | 4     | 3  | 5  |
| Total           |        | 21    | 7     | 5  | 9  |
```

## Step 2: Configure Run

Ask the user:
1. **URL** (default from test case files)
2. **Filter**:
   - `all` (default), `smoke`, `P0`, `P0,P1`
   - `module:AUTH`, `tag:security`
   - Specific IDs: `TC-AUTH-001, TC-CART-003`
3. **Stop on first failure?** (default: no)
4. **Credentials** if needed

---

## Step 3: Execute

For each test case in the filtered set:

1. Read preconditions, steps, expected results
2. Set up preconditions via Playwright MCP
3. Execute each step — perform action, verify expected result
4. Record verdict: **PASS / FAIL / BLOCKED / SKIPPED**
5. On FAIL: screenshot + actual result + console errors

Progress after each module:
```
[AUTH 7/9] ✓✓✓✓✓✗✓ — 6 pass, 1 fail (TC-AUTH-005)
[CART 12/12] ✓✓✓✓✓✓✓✓✓✓✓✓ — 12 pass
```

On failure, capture:
```yaml
test_case: TC-AUTH-005
verdict: FAIL
failed_at_step: 3
expected: "Redirected to /dashboard"
actual: "Stayed on /login, no error shown"
screenshot: qa/screenshots/TC-AUTH-005-fail.png
console_errors: ["TypeError: Cannot read 'user'"]
```

---

## Step 4: Compare with Previous Runs

Find most recent `run-*.md` in `qa/reports/{project}/regression/`.

- 🔴 **Regression**: was PASS, now FAIL
- 🟢 **Fix**: was FAIL, now PASS
- 🟡 **Persistent failure**: FAIL in both runs
- ⚪ **Stable**: PASS in both runs

If no previous run exists: "First run — no baseline for comparison."

---

## Step 5: Report

Save to `qa/reports/{project}/regression/run-[date]-[N].md`:

```
## Regression Test Report
### Run: [ID] | App: [URL] | Filter: [all/smoke/P0] | Date: [date]

### Summary
| Metric          | Count |
|-----------------|-------|
| Executed        | X     |
| Passed          | X     |
| Failed          | X     |
| Blocked         | X     |
| **Pass rate**   | X%    |
| 🔴 Regressions  | X     |
| 🟢 Fixes        | X     |

### Results by Module
| Module | Total | Pass | Fail | Blocked | Rate |
|--------|-------|------|------|---------|------|
| AUTH   | 9     | 8    | 1    | 0       | 89%  |
| CART   | 12    | 12   | 0    | 0       | 100% |

### Detailed Results
| TC ID       | Title                  | Verdict | Regression? |
|-------------|------------------------|---------|-------------|
| TC-AUTH-001 | Successful login       | PASS    |             |
| TC-AUTH-005 | Session expiry         | FAIL    | 🔴 New       |
| TC-CART-001 | Add item               | PASS    |             |

### Regression Detail
#### 🔴 TC-AUTH-005: Session expiry
- Previously: PASS ([previous run ID])
- Now: FAIL at step 3
- Expected: "Session active for 30 min"
- Actual: "Session expired immediately"
- Evidence: qa/screenshots/TC-AUTH-005-fail.png

### Release Assessment
**Pass rate**: X% | **Regressions**: N
**Verdict**: [Release / Release with known issues / Do NOT release]
```

### After the Report

Based on results:
- 🔴 Regressions: "Want me to **create Jira tickets** for regressions?"
  → triggers `jira-create-bug` for all new FAIL results
- 🔴 Regressions: "Want me to **investigate [TC-ID] in depth**?"
  → triggers exploratory deep dive on that module
- ✓ All pass: "All green! Want me to **expand the test suite**?" → `test-case-designer`
- 🟡 Persistent fails: "These have failed for N runs: [list].
  Should I **verify if test cases are still valid**?" → `test-case-designer` (re-baseline)
- "Want me to **auto-fix broken tests** with Playwright Healer?"
  → `npx playwright test --agents healer`

---

## Rules
- Execute exactly what test case says — no improvising
- Verdicts are objective — close enough ≠ PASS
- Screenshot every failure
- Never modify test cases during a run
- Complete the full run unless configured otherwise
- Always save the report to `qa/test-runs/`
- Never overwrite previous run reports
- **Always close the browser** (`browser_close`) after the session is complete
