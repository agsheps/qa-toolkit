# QA Common Reference

Shared definitions for all QA Toolkit skills.
Other skills read this file when they need severity levels, oracles, or formats.

---

## Tool Stack

This toolkit uses a layered architecture. Each layer has a clear role:

### Strategy layer (our skills — WHAT and WHEN to test)
- `exploratory-testing` — autonomous discovery + heuristic oracles
- `spec-based-testing` — verification against specifications
- `test-case-designer` — formal test case creation
- `regression-testing` — test case execution + regression tracking

### Tactics layer (external skills — HOW to write good test code)
- **TestDino Playwright Skill** (`testdino-hq/playwright-skill`)
  70+ guides on locators, assertions, POM, mocking, CI/CD, accessibility.
  Location: `.claude/skills/` (installed via `npx skills add`).
  When generating Playwright test code, Claude should consult these guides
  for correct patterns — especially `core/locators.md`, `core/assertions.md`,
  and `core/waits.md`.

- **Anthropic webapp-testing** (`anthropics/skills/webapp-testing`)
  Python Playwright scripts for server management, element discovery,
  console capture. Location: installed via `/plugin install`.
  Optional — only needed if the app requires a local server to be started
  before testing (`scripts/with_server.py`).

### Automation layer (Playwright Agents — self-healing pipeline)
- **Planner** — explores the app, produces a Markdown test plan
- **Generator** — converts the plan into executable Playwright tests
- **Healer** — monitors failures, auto-fixes broken selectors/waits
  Initialized via `npx playwright init-agents`.

### Interaction layer (Playwright MCP — live browser control)
- `@playwright/mcp` — 25+ browser control tools via MCP protocol
  Used by our strategy skills for exploration and test execution.
  Always prefer MCP over bash for interactive testing.

---

## Severity Definitions

| Level    | Code | Meaning                                                   | SLA              |
|----------|------|-----------------------------------------------------------|------------------|
| Blocker  | S0   | Crash, data loss, security breach, core flow broken       | Fix before release|
| Critical | S1   | Major feature broken, no workaround                       | Fix within 24h   |
| Major    | S2   | Feature partially broken, workaround exists               | Fix within sprint |
| Minor    | S3   | Cosmetic, edge case, inconsistent but functional          | Backlog          |
| Trivial  | S4   | Typo, alignment off, suggestion                           | Nice-to-have     |

## Priority Matrix

|                       | All users   | Some users | Edge case |
|-----------------------|-------------|------------|-----------|
| Blocker / Critical    | P0          | P1         | P1        |
| Major                 | P1          | P2         | P3        |
| Minor                 | P2          | P3         | P4        |
| Trivial               | P3          | P4         | P4        |

---

## Heuristic Oracles (FEW HICCUPPS)

When there is NO specification, use these to decide if behavior is a bug:

| Oracle | Key question |
|--------|-------------|
| **F**amiliarity | Does it work like similar features in well-known apps? |
| **E**xplainability | Can the behavior be explained by a good design reason? |
| **W**orld | Does it match real-world logic and common sense? |
| **H**istory | Is it consistent with its own past behavior? |
| **I**mage | Does it match the product's brand and quality bar? |
| **C**omparable products | How do competitors handle this? |
| **C**laims | Does it match what the UI text/labels/tooltips claim? |
| **U**ser expectations | Would a reasonable user be surprised or confused? |
| **P**urpose | Does the feature fulfill its apparent purpose? |
| **P**roduct consistency | Is it consistent across the whole product? |
| **S**tandards | Does it follow web platform conventions? |

---

## Test Case YAML Format

```yaml
id: TC-[MODULE]-[NNN]
title: "[Short descriptive title]"
module: "[Feature area]"
type: positive | negative | boundary | destructive | integration
priority: P0-P4
tags: [smoke, happy-path, validation, security, responsive, a11y]
preconditions:
  - "User is logged in as test@example.com"
steps:
  - step: 1
    action: "Navigate to /products"
    expected: "Product list page loads with grid of products"
  - step: 2
    action: "Click 'Add to Cart' on product 'Widget A'"
    expected: "Cart counter in header shows '1'"
cleanup: null | "Delete created test data"
```

## Bug Report YAML Format

```yaml
id: BUG-[NNN]
title: "[What is wrong — concise]"
severity: S0-S4
priority: P0-P4
module: "[Feature area]"
environment: "[Browser / viewport / OS]"
preconditions:
  - "[Required state]"
steps_to_reproduce:
  - "[Step 1]"
expected_result: "[What should happen]"
actual_result: "[What actually happens]"
evidence: "qa/screenshots/BUG-NNN.png"
oracle: "[Which heuristic detected this — or spec reference]"
notes: "[Context, suggested fix]"
```

---

## Jira Integration

Bugs found by any skill can be exported to Jira via `jira-create-bug`.

### Jira Instance
- Type: **Jira Server** or **Jira Cloud** (configured per user)
- URL: Stored in `qa/jira-config.json` → `jira_url` field
- Auth: `JIRA_PAT` environment variable
  - Jira Server/Data Center: Bearer token (Personal Access Token)
  - Jira Cloud (`*.atlassian.net`): Basic auth with `email:api-token`
- Config: `qa/jira-config.json`
- Log: `qa/jira-log.md` (append-only, tracks all exported bugs)

### Environment Notes
- If `$JIRA_PAT` is not loaded, source the user's shell profile first
- Use `node -e` for JSON parsing (fallback: `python3 -c`)
- Project keys may differ from project names — always verify via API
- Always verify project key via `/rest/api/2/project/<KEY>` before creating tickets

### Severity → Jira Priority Mapping

**IMPORTANT**: Priority names are instance-specific. The standard Jira names
(Blocker/Critical/Major/Minor/Trivial) may not exist on every instance.
Always fetch actual names from `/rest/api/2/priority` during first-time setup.

Default mapping:

| QA Severity | Jira Priority | Jira Label   |
|-------------|---------------|--------------|
| S0 Blocker  | Highest       | severity-S0  |
| S1 Critical | Highest       | severity-S1  |
| S2 Major    | High          | severity-S2  |
| S3 Minor    | Medium        | severity-S3  |
| S4 Trivial  | Low           | severity-S4  |

### Audit Finding → Jira Issue Type Mapping

Common Jira issue types: Story, Task, Bug, Sub-task, Feature, Epic.
Actual types depend on the project configuration — fetch via API if needed.

| Source Skill         | Severe findings → | Minor findings → |
|----------------------|-------------------|------------------|
| exploratory-testing  | Bug               | Bug              |
| spec-based-testing   | Bug               | Bug              |
| regression-testing   | Bug               | Bug              |
| module-testing       | Bug               | Bug              |
| ux-heuristics        | Bug (sev 3-4)     | Task (sev 1-2)   |
| security-baseline    | Bug               | Bug              |
| accessibility-audit  | Bug (Critical/Major) | Task (Minor)  |
| performance-audit    | Bug (grade D-F)   | Task (grade C)   |
| content-review       | Bug (placeholders) | Task (typos)    |

---

## Output Structure

All skills save to `qa/` in the project root:

```
qa/
├── reports/{project}/{type}/  ← Test reports organized by project and type
│   ├── noder/
│   │   ├── exploratory/
│   │   │   └── run-2026-04-02.md
│   │   ├── regression/
│   │   │   ├── run-2026-04-02-1.md
│   │   │   └── run-2026-04-03-1.md
│   │   ├── spec-based/
│   │   ├── module/
│   │   ├── ux-heuristics/
│   │   ├── security/
│   │   ├── accessibility/
│   │   ├── performance/
│   │   └── content-review/
│   └── other-project/
│       └── ...
├── test-cases/                ← YAML test cases (by module)
├── bugs/                      ← Bug reports (YAML)
├── screenshots/               ← Evidence
├── specs/                     ← User documentation (optional)
├── plans/                     ← Playwright Agent plans (Markdown)
├── jira-config.json           ← Jira project settings (reusable)
└── jira-log.md                ← Export log (bug ID → Jira key)
```

### Report Path Convention

Every testing skill MUST save its report to:
```
qa/reports/{project}/{type}/run-YYYY-MM-DD[-N].md
```

Where:
- `{project}` — short lowercase name derived from the app URL
  (e.g., `noder` from `noder.talrace.com`, `myapp` from `localhost:3000/myapp`).
  **Ask the user** for a project name if it cannot be inferred from the URL.
- `{type}` — the testing type: `exploratory`, `spec-based`, `regression`,
  `module`, `ux-heuristics`, `security`, `accessibility`, `performance`,
  `content-review`
- `YYYY-MM-DD` — run date
- `[-N]` — sequential number if multiple runs of the same type on the same day
  (e.g., `run-2026-04-02-1.md`, `run-2026-04-02-2.md`)

Create the directories automatically (`mkdir -p`) before saving.

## Module Naming

Short uppercase labels derived from the app's own terminology:
AUTH, NAV, SEARCH, CART, FORM, PROFILE, SETTINGS, ADMIN, API, etc.

## ID Sequences

- Test cases: `TC-[MODULE]-[NNN]` (e.g., TC-AUTH-001)
- Bugs: `BUG-[NNN]` (sequential across modules)
- Test runs: `run-[YYYY-MM-DD][-N]`
