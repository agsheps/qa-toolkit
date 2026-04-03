---
name: spec-based-testing
description: >
  Test a web application against provided documentation. Supports full
  specs (PRD, user stories, acceptance criteria) and partial specs (brief
  descriptions, bullet points). Use when the user provides ANY documentation
  about expected behavior — "test against this spec", "verify requirements",
  "check if this matches the design doc", "validate acceptance criteria",
  "test these user stories", "here's how it should work — test it",
  "does it match the PRD". Also trigger when user pastes or attaches
  requirements and asks to test, or says "test with documentation".
---

# Specification-Based Testing

You verify that implementation matches documented requirements. Every claim
in the spec becomes a testable assertion.

**First**, read: `qa-common/references/qa-reference.md (in this plugin)`

---

## Step 1: Receive & Classify Spec

Accept: pasted text, file (PDF/DOCX/MD), URL, or verbal description.

**Full specification** (PRD, user stories with AC, OpenAPI):
→ **Strict Mode** — every requirement is PASS/FAIL.

**Partial specification** (bullet points, brief description, incomplete notes):
→ **Guided Mode** — documented items strict, undocumented areas via heuristic oracles.

```
## Spec Analysis
- Source: [pasted / file / URL / verbal]
- Depth: [Full / Partial]
- Mode: [Strict / Guided]
- Estimated coverage: [X% of app features]
```

---

## Step 2: Extract Requirements

Parse into atomic, testable items:

```
| Req ID  | Requirement                           | Source Section | Testable |
|---------|---------------------------------------|----------------|----------|
| REQ-001 | User can register with email/password | 2.1 Auth       | Yes      |
| REQ-002 | Password minimum 8 characters         | 2.1.1 Rules    | Yes      |
| REQ-003 | System should be fast                 | 3.1 Perf       | No*      |
```

*Flag untestable requirements and ask for clarification.

Present for user confirmation before executing.

---

## Step 3: Execute Tests via Playwright MCP

For each requirement:
```
REQ-[ID]: [text]
├── Precondition: [setup state]
├── Action: [browser action]
├── Expected (from spec): [spec says]
├── Actual: [what happened]
├── Verdict: PASS / FAIL / PARTIAL / BLOCKED
└── Evidence: [screenshot if FAIL]
```

**Guided Mode extra:** after all documented requirements, do one pass of
heuristic testing on undocumented areas using FEW HICCUPPS oracles.
Label these findings clearly as "beyond spec."

---

## Step 4: Gap Analysis

```
## Spec Coverage
### Covered: [list areas with req count]
### NOT covered: [error handling, mobile, a11y, edge cases...]
### Ambiguous: [requirements needing clarification]
```

---

## Step 5: Report

Save to `qa/reports/{project}/spec-based/run-[date].md`:

```
## Spec-Based Test Report
### App: [URL] | Spec: [source] | Mode: [Strict/Guided] | Date: [date]

### Summary
| Metric       | Count |
|--------------|-------|
| Requirements | X     |
| Passed       | X     |
| Failed       | X     |
| Pass Rate    | X%    |

### Results by Module
| Req ID  | Requirement             | Verdict | Notes            |
|---------|-------------------------|---------|------------------|
| REQ-001 | Register with email/pwd | PASS    |                  |
| REQ-002 | Password min 8 chars    | FAIL    | Accepts 3 chars  |

### Failed Requirements (Detail)
#### REQ-002: Password min 8 characters
- **Spec says**: reject < 8 chars
- **Actual**: "abc" accepted, registration succeeds
- **Severity**: S1
- **Evidence**: qa/screenshots/REQ-002-fail.png

### Beyond-Spec Findings (Guided Mode only)
(heuristic oracle findings on undocumented areas)

### Gap Analysis
(from Step 4)
```

### After the Report

- "Want me to **create Jira tickets** for failed requirements?" → `jira-create-bug`
- "Want me to **create test cases** from verified requirements?" → `test-case-designer`
- "Want me to **generate Playwright tests** for passing requirements?" → `playwright-codegen`

---

## Rules
- Spec is the oracle — if spec says X and app does Y, it's FAIL
- Quote the spec text when reporting failures
- Don't assume — unclear requirements are NOT automatic PASS
- Screenshot all FAIL and PARTIAL results
- **Always close the browser** (`browser_close`) after the session is complete
