# QA Toolkit — Claude Code Plugin for AI-Powered Web Testing

AI-powered testing framework for web applications. **13 skills** across **2 plugins** — from exploratory testing to Jira ticket creation, all driven by AI through a live browser.

## How It Works

```
You say:  "Run exploratory testing on https://myapp.com"
AI does:  Opens browser → maps all pages → tests every feature → finds bugs → writes report
You say:  "Create Jira tickets"
AI does:  Maps severity → creates tickets in your Jira → attaches screenshots
```

No test scripts needed. No documentation required. The AI explores your app like a senior QA engineer would.

---

## Installation

### Quick Start

```bash
# 1. Register Playwright MCP (one-time)
npx playwright install chromium
claude mcp add playwright -s user -- npx @playwright/mcp@latest

# 2. Install plugins
/plugin marketplace add andrei.shapialevich/qa-toolkit
/plugin install qa-core@qa-toolkit
/plugin install qa-audits@qa-toolkit
```

All 13 skills are loaded. Start testing:
```
Run exploratory testing on https://myapp.com
```

### Install Only What You Need

```
/plugin install qa-core@qa-toolkit      # 8 skills: testing workflow + codegen + Jira
/plugin install qa-audits@qa-toolkit    # 5 skills: UX, security, a11y, perf, content
```

### Optional: TestDino Playwright Skill

Adds 70+ best-practice guides that improve Playwright code generation quality:
```bash
npx skills add testdino-hq/playwright-skill/core
```

---

## Plugins & Skills

### qa-core (8 skills)

| Skill | What It Does | Trigger Phrases |
|-------|-------------|-----------------|
| **exploratory-testing** | Discovers all features, tests against heuristic oracles (FEW HICCUPPS), produces bug report | "test the app", "find bugs", "QA this app", "explore and test", "test everything" |
| **spec-based-testing** | Verifies implementation against provided documentation (PRD, user stories, acceptance criteria) | "test against this spec", "verify requirements", "check if this matches the design doc" |
| **test-case-designer** | Creates formal, reusable YAML test cases — with or without documentation | "create test cases", "design tests", "build test suite", "prepare for regression" |
| **regression-testing** | Executes YAML test cases, compares with previous runs, detects regressions | "run regression tests", "run smoke tests", "did anything break", "verify release" |
| **module-testing** | Deep-tests a single module (80% depth + 20% integration boundaries) | "test the cart", "test just the checkout", "we changed the search — test it" |
| **playwright-codegen** | Generates production .spec.ts files from YAML test cases or exploratory findings | "generate Playwright tests", "automate these test cases", "write test code" |
| **jira-create-bug** | Creates Jira tickets from found bugs, maps severity to priority, attaches screenshots | "create Jira tickets", "send to Jira", "file bugs in Jira" |
| **qa-common** | Shared reference: severity levels, priority matrix, heuristic oracles, formats | *(not invoked directly — used by other skills)* |

### qa-audits (5 skills)

| Skill | What It Does | Trigger Phrases |
|-------|-------------|-----------------|
| **ux-heuristics** | Evaluates usability against Jakob Nielsen's 10 heuristics, scores 0-4 per principle | "UX audit", "usability review", "is this user-friendly", "Nielsen heuristics" |
| **security-baseline** | Frontend security audit: XSS, CSRF, cookies, headers, open redirects (OWASP-informed) | "security check", "security audit", "check for XSS", "is this secure" |
| **accessibility-audit** | WCAG 2.1 Level AA compliance: keyboard nav, ARIA, focus management, contrast | "accessibility audit", "a11y check", "WCAG", "can a blind user use this" |
| **performance-audit** | Perceived performance: loading states, layout stability, interaction responsiveness | "performance check", "is this fast", "feels slow", "loading speed" |
| **content-review** | Text quality: placeholders, typos, terminology consistency, broken links, tone | "review the copy", "check for typos", "content audit", "proofread" |

---

## Workflows

### Full Testing Workflow (no documentation needed)

```
1. "Run exploratory testing on https://myapp.com — deep dive"
   → AI maps app, tests every feature, finds bugs, writes report

2. "Create Jira tickets for the bugs found"
   → AI creates tickets in Jira with severity, steps, screenshots

3. "Create comprehensive test cases for https://myapp.com"
   → AI generates YAML test cases covering all discovered features

4. "Generate Playwright tests from the test cases"
   → AI produces .spec.ts files with real selectors from the app

5. "Run regression tests"
   → AI executes test cases, compares with previous runs
   → Run this on every release to catch regressions
```

### Audit Workflow

```
"Run a UX audit on https://myapp.com"
"Security check on https://myapp.com"
"Accessibility audit on https://myapp.com"
"Check performance of https://myapp.com"
"Review the copy on https://myapp.com"
"Create Jira tickets for all findings"
```

### Module Testing (after changes)

```
"We changed the search — test it on https://myapp.com/search"
"Test just the checkout flow on https://myapp.com/checkout"
```

### With Documentation

```
"Here's the PRD — test against this spec on https://myapp.com"
(paste or attach PRD / user stories / acceptance criteria)
```

---

## Jira Integration

The `jira-create-bug` skill creates tickets automatically in Jira after any testing session.

### Setup (one-time)

1. **Generate a Personal Access Token** in Jira:
   Profile → Personal Access Tokens → Create token

2. **Save the token** as an environment variable:
   ```bash
   echo 'export JIRA_PAT="your-token-here"' >> ~/.bashrc
   ```

3. **First use** — the skill will ask for your project key and create `qa/jira-config.json`.
   A template is included at `qa-core-plugin/skills/jira-create-bug/jira-config.example.json`.

### How It Works

After any testing skill finds bugs:
```
AI:  Found 5 bugs. Want me to create Jira tickets?
You: Yes
AI:  In which Jira project? [NOD]:
You: NOD
AI:  [creates 5 tickets, attaches screenshots, shows links]
```

Severity mapping: S0/S1 → Highest, S2 → High, S3 → Medium, S4 → Low.

---

## Report Structure

All reports are organized by **project**, **testing type**, and **date**:

```
qa/reports/
├── myapp/
│   ├── exploratory/
│   │   ├── run-2026-04-01.md
│   │   └── run-2026-04-02.md
│   ├── regression/
│   │   └── run-2026-04-02-1.md
│   ├── ux-heuristics/
│   ├── security/
│   ├── accessibility/
│   ├── performance/
│   └── content-review/
├── other-project/
│   └── ...
```

Other output:
```
qa/
├── test-cases/        ← YAML test cases (by module)
├── screenshots/       ← Bug evidence
├── jira-config.json   ← Jira connection settings
└── jira-log.md        ← All exported bug → Jira ticket mappings
```

---

## Architecture

```
┌─────────────────────────────────────────────────┐
│  Strategy Layer (this plugin — WHAT to test)     │
│  exploratory · spec-based · regression · module  │
│  ux · security · a11y · performance · content    │
├─────────────────────────────────────────────────┤
│  Integration Layer (Jira)                        │
│  jira-create-bug → REST API → tickets            │
├─────────────────────────────────────────────────┤
│  Code Generation Layer                           │
│  test-case-designer → YAML → playwright-codegen  │
├─────────────────────────────────────────────────┤
│  Interaction Layer (Playwright MCP)              │
│  Live browser control — 25+ tools via MCP        │
└─────────────────────────────────────────────────┘
```

---

## Repository Structure

```
qa-toolkit/
├── qa-core-plugin/
│   ├── .claude-plugin/
│   │   └── plugin.json
│   └── skills/
│       ├── qa-common/
│       │   ├── SKILL.md
│       │   └── references/
│       │       └── qa-reference.md
│       ├── exploratory-testing/
│       │   └── SKILL.md
│       ├── spec-based-testing/
│       │   └── SKILL.md
│       ├── test-case-designer/
│       │   └── SKILL.md
│       ├── regression-testing/
│       │   └── SKILL.md
│       ├── module-testing/
│       │   └── SKILL.md
│       ├── playwright-codegen/
│       │   └── SKILL.md
│       └── jira-create-bug/
│           ├── SKILL.md
│           └── jira-config.example.json
├── qa-audits-plugin/
│   ├── .claude-plugin/
│   │   └── plugin.json
│   └── skills/
│       ├── ux-heuristics/
│       │   └── SKILL.md
│       ├── security-baseline/
│       │   └── SKILL.md
│       ├── accessibility-audit/
│       │   └── SKILL.md
│       ├── performance-audit/
│       │   └── SKILL.md
│       └── content-review/
│           └── SKILL.md
├── .gitignore
└── README.md
```

## License

TALRACE
# qa-toolkit
