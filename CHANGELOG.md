# Changelog

## [1.0.0] - 2026-04-03

### Added

**qa-core plugin (8 skills):**
- **exploratory-testing** - Autonomous discovery and heuristic testing (FEW HICCUPPS oracles)
- **spec-based-testing** - Verification against PRD, user stories, acceptance criteria
- **test-case-designer** - Formal YAML test case creation with/without documentation
- **regression-testing** - Execute YAML test cases, compare runs, detect regressions
- **module-testing** - Deep-test a single module (80% depth + 20% integration boundaries)
- **playwright-codegen** - Generate production .spec.ts files from test cases or findings
- **jira-create-bug** - Create Jira tickets via REST API with severity mapping and screenshots
- **qa-common** - Shared reference: severity levels, priority matrix, heuristic oracles, formats

**qa-audits plugin (5 skills):**
- **ux-heuristics** - Jakob Nielsen's 10 heuristic evaluation with 0-4 scoring
- **security-baseline** - Frontend security audit (OWASP-informed): XSS, CSRF, headers, etc.
- **accessibility-audit** - WCAG 2.1 Level AA compliance: keyboard nav, ARIA, focus, contrast
- **performance-audit** - Perceived performance evaluation with A-F rating scale
- **content-review** - Text quality: placeholders, typos, terminology, broken links, tone
