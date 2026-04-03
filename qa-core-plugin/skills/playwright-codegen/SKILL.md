---
name: playwright-codegen
description: >
  Generate production-quality Playwright test files (.spec.ts) from YAML
  test cases, exploratory findings, or direct app exploration via MCP.
  Leverages TestDino Playwright Skill guides for best practices and can
  invoke Playwright Agents (Planner/Generator/Healer) for automation.
  Use when user says "generate Playwright tests", "write test code",
  "create spec files", "automate these test cases", "generate test files",
  "code the tests", "make tests runnable", "convert to Playwright",
  "bootstrap test suite", "I need automated tests", or "add to CI".
  Also trigger when user says "generate tests" after exploratory or
  regression testing.
---

# Playwright Test Code Generator

You generate production-quality Playwright test files grounded in real app
behavior. You NEVER write tests from imagination — always explore first.

**First**, read: `qa-common/references/qa-reference.md (in this plugin)`

**If TestDino Playwright Skill is installed**, read the following guides
before writing ANY test code (check `.claude/skills/` for `testdino-hq`):
- `core/locators.md` — selector strategy and priority
- `core/assertions.md` — assertion patterns
- `core/waits.md` — waiting strategies (never use arbitrary timeouts)
- `core/fixtures.md` — test fixtures and setup/teardown
- `pom/page-object-model.md` — POM pattern (if generating a large suite)

---

## Three Input Modes

### Mode A: From YAML Test Cases
Test cases exist in `qa/test-cases/*.yaml` (created by `test-case-designer`).
Convert each YAML test case into a Playwright test, preserving the TC-ID
as a comment for traceability.

### Mode B: From Exploratory Findings
An exploratory or spec-based report exists in `qa/test-runs/`.
Generate tests for the flows tested and bugs found.

### Mode C: Direct Exploration
No prior artifacts. Explore the app via Playwright MCP first,
then generate tests based on what you discover.

---

## Step 1: Explore via Playwright MCP (always)

Even in Mode A, navigate the app to verify:
1. Elements from test cases actually exist
2. Selectors are current and correct
3. Expected behaviors match reality

Record: element selectors, text content, state changes, URLs.

---

## Step 2: Plan Test Structure

```
tests/
├── auth.spec.ts           # Login, signup, logout
├── [feature].spec.ts      # One file per module
├── navigation.spec.ts     # Routes, links, 404
└── playwright.config.ts   # Config (generate if missing)
```

For large suites (20+ tests), also generate:
```
tests/
├── pages/                 # Page Object Model
│   ├── login.page.ts
│   ├── cart.page.ts
│   └── base.page.ts
├── fixtures/
│   └── auth.fixture.ts   # Shared auth setup
└── ...specs...
```

---

## Step 3: Write Tests

### Selector Priority (from TestDino guides)
1. `data-testid`: `page.getByTestId("submit-btn")`
2. Role + name: `page.getByRole("button", { name: "Submit" })`
3. Label: `page.getByLabel("Email")`
4. Placeholder: `page.getByPlaceholder("Enter email")`
5. Text: `page.getByText("Welcome")`
6. CSS (last resort): `page.locator(".submit-btn")`

**NEVER**: `div > span:nth-child(3)`, auto-generated classes, XPath.

### Test Template

```typescript
import { test, expect } from "@playwright/test";

test.describe("AUTH — Login", () => {
  test.beforeEach(async ({ page }) => {
    await page.goto("/login");
    await page.waitForLoadState("networkidle");
  });

  // TC-AUTH-001: Successful login with valid credentials
  test("should redirect to dashboard on valid login", async ({ page }) => {
    await page.getByLabel("Email").fill("test@example.com");
    await page.getByLabel("Password").fill("TestPass123");
    await page.getByRole("button", { name: "Log in" }).click();

    await expect(page).toHaveURL(/dashboard/);
    await expect(page.getByText("Welcome")).toBeVisible();
  });

  // TC-AUTH-002: Login rejected with wrong password
  test("should show error on invalid password", async ({ page }) => {
    await page.getByLabel("Email").fill("test@example.com");
    await page.getByLabel("Password").fill("wrongpass");
    await page.getByRole("button", { name: "Log in" }).click();

    await expect(page.getByText("Invalid email or password")).toBeVisible();
    await expect(page).toHaveURL(/login/);
  });
});
```

### Quality Rules

- **One behavior per test** — don't combine scenarios
- **Descriptive names**: `"should show error when email empty"` not `"test 1"`
- **Independent tests** — no shared state between tests
- **No arbitrary waits**: use `waitForLoadState`, `waitForResponse`,
  `expect().toBeVisible()` — never `waitForTimeout` (per TestDino guides)
- **TC-ID as comment** above each test for traceability to YAML cases
- **Assertion messages**: `expect(count).toBe(3)` — keep it clean
- **Clean up** created data in `afterEach` when needed

### Auth Setup (if needed)

```typescript
// tests/fixtures/auth.fixture.ts
import { test as setup, expect } from "@playwright/test";

setup("authenticate", async ({ page }) => {
  await page.goto("/login");
  await page.getByLabel("Email").fill("test@example.com");
  await page.getByLabel("Password").fill("TestPass123");
  await page.getByRole("button", { name: "Log in" }).click();
  await page.waitForURL("/dashboard");
  await page.context().storageState({ path: "tests/.auth/user.json" });
});
```

### Page Object Model (large suites only)

```typescript
// tests/pages/login.page.ts
import { type Page, type Locator } from "@playwright/test";

export class LoginPage {
  readonly page: Page;
  readonly emailInput: Locator;
  readonly passwordInput: Locator;
  readonly submitButton: Locator;
  readonly errorMessage: Locator;

  constructor(page: Page) {
    this.page = page;
    this.emailInput = page.getByLabel("Email");
    this.passwordInput = page.getByLabel("Password");
    this.submitButton = page.getByRole("button", { name: "Log in" });
    this.errorMessage = page.getByRole("alert");
  }

  async goto() {
    await this.page.goto("/login");
    await this.page.waitForLoadState("networkidle");
  }

  async login(email: string, password: string) {
    await this.emailInput.fill(email);
    await this.passwordInput.fill(password);
    await this.submitButton.click();
  }
}
```

---

## Step 4: Generate Config (if missing)

```typescript
// playwright.config.ts
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  testDir: "./tests",
  fullyParallel: true,
  retries: process.env.CI ? 2 : 1,
  reporter: process.env.CI ? "github" : "html",
  use: {
    baseURL: "http://localhost:3000",
    trace: "on-first-retry",
    screenshot: "only-on-failure",
  },
  projects: [
    { name: "desktop", use: { ...devices["Desktop Chrome"] } },
    { name: "mobile", use: { ...devices["Pixel 7"] } },
  ],
  webServer: {
    command: "npm start",
    url: "http://localhost:3000",
    reuseExistingServer: !process.env.CI,
  },
});
```

---

## Step 5: Verify & Deliver

1. Run `npx playwright test --list` to verify tests parse
2. Offer to run: `npx playwright test`
3. Fix any failures
4. Report:

```
## Generated Test Suite
| File                     | Tests | Coverage                     |
|--------------------------|-------|------------------------------|
| tests/auth.spec.ts       | 5     | Login, logout, bad password  |
| tests/cart.spec.ts       | 8     | Add, remove, qty, totals     |
| Total                    | 13    |                              |

### Run with:
npx playwright test              # all tests
npx playwright test --ui         # interactive UI
npx playwright test --project=mobile  # mobile only

### CI (GitHub Actions):
(suggest adding .github/workflows/playwright.yml)
```

### After Delivery

- "Want to **auto-heal** broken tests?" → `npx playwright test --agents healer`
- "Want the **Planner agent** to find more test scenarios?" →
  `npx playwright test --agents planner --url [URL]`
- "Want to **run regression**?" → `regression-testing` skill

---

## Rules
- **Explore first, write second** — never generate from imagination
- **Use real selectors** from the accessibility tree
- **Consult TestDino guides** for patterns when available
- One file per module, TC-ID comments for traceability
- Include negative tests, not just happy paths
- Generate `playwright.config.ts` if missing
- **Always close the browser** (`browser_close`) after the session is complete
