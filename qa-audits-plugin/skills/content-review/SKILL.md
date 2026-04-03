---
name: content-review
description: >
  Review all text content and copy in a web application for quality,
  consistency, and professionalism using Playwright MCP. Checks for
  placeholder text, typos, inconsistent terminology, broken grammar,
  developer artifacts, dead links, misleading labels, and tone issues.
  No docs needed. Use when user says "review the copy", "check the
  text", "content review", "proofread", "check for typos", "is the
  text good", "content audit", "copy review", "text quality", "check
  for Lorem ipsum", "review labels and messages", or "editorial review".
  Also trigger when user mentions placeholder text, dummy content, or
  wants a text quality check.
---

# Content & Copy Review

You review all visible text in a web application for quality, consistency,
and professionalism. Placeholder text, typos, inconsistent terminology,
and misleading labels erode user trust even when the features work.

**First**, read: `qa-common/references/qa-reference.md (in this plugin)`

---

## Setup

1. **URL** (default: `http://localhost:3000`)
2. **Language**: what language should the content be in? (default: English)
3. **Tone**: formal / casual / technical? (default: infer from the app)
4. **Scope**: entire app or specific pages?

---

## Review Categories

### COPY-1: Placeholder & Developer Artifacts

Navigate every page and search for:

- **Lorem ipsum** or any Latin filler text
- **TODO**, **FIXME**, **HACK**, **XXX** in visible content
- **"test"**, **"example"**, **"sample"**, **"foo"**, **"bar"** in
  content that should be real
- **"undefined"**, **"null"**, **"NaN"**, **"[object Object]"** displayed
  as text (data binding failures)
- Default framework text: "Welcome to React", "Hello World",
  "Your App Name", "Company Name Here"
- Placeholder images: broken image icons, gray squares
- Hardcoded email: **"admin@example.com"**, **"test@test.com"** in
  visible content (not test data)

### COPY-2: Spelling & Grammar

Read all visible text blocks on every page:

- Spelling errors in headings, labels, body text, buttons, tooltips
- Grammar issues: subject-verb agreement, tense consistency, fragments
- Punctuation: missing periods, inconsistent comma usage, double spaces
- Capitalization: consistent pattern? (Title Case vs Sentence case —
  pick one and enforce it)

### COPY-3: Terminology Consistency

Track terms used across the app. Flag inconsistencies:

| Concept | Terms found | Consistent? |
|---------|------------|-------------|
| Confirm action | "Save", "Submit", "Confirm", "Apply" | ✗ Inconsistent |
| Remove item | "Delete", "Remove", "Discard" | ✗ Inconsistent |
| User account | "Account", "Profile", "Settings" | Check context |
| Error state | "Error", "Problem", "Issue", "Failed" | ✗ |

**Rule**: same action should use the same verb throughout the app.
Different verbs = different meaning.

### COPY-4: Microcopy Quality

Evaluate small but impactful text:

**Button labels**:
- Are they action-oriented? ("Save changes" vs "Submit")
- Are they specific? ("Delete account" vs "Delete")
- Are they consistent? (same action = same label everywhere)

**Form labels & help text**:
- Labels describe what to enter, not the field name
- Format hints visible: "DD/MM/YYYY", "e.g., john@example.com"
- Required indicators: "*" or "(required)" present?

**Error messages**:
- Specific about what went wrong?
- Tell the user how to fix it?
- Written in human language, not tech speak?
- Free of blame? ("Please enter a valid email" vs "Invalid input")

**Empty states**:
- When there's no data: is there a helpful message?
- Does it guide the user to the next action?
- Or is it just blank / "No results"?

**Success messages**:
- Clear confirmation of what happened?
- Appropriate tone? (not overly excited for mundane actions)
- Disappear at the right time? (not too fast, not permanent)

### COPY-5: Tone & Voice Consistency

Evaluate the overall tone:
- Is it consistent across the app? (formal login page + casual dashboard?)
- Does it match the product's apparent audience?
- Is it appropriate for the context? (errors shouldn't be jokey)
- Is there personality, or is it generic/robotic?

### COPY-6: Links & References

Check all visible links and references:
- **Broken links**: click every link — does it lead somewhere?
- **Dead anchors**: in-page links (#section) that go nowhere
- **Mailto links**: are they correct?
- **External links**: do they open in new tab with indication?
- **Help/docs links**: do they lead to relevant, existing content?
- **Legal links**: privacy policy, terms of service — exist and load?

### COPY-7: Localization Readiness (if multilingual)

If the app has language switching:
- Does switching language change ALL text? (or are some strings hardcoded?)
- Are dates, numbers, currencies localized?
- Does the layout handle longer text in other languages?
- RTL support if applicable?

---

## Report

Save to `qa/reports/{project}/content-review/run-[date].md`:

```
## Content & Copy Review — [URL]
### Date: [date] | Language: [en] | Tone: [casual/formal/inferred]

### Summary
| Category              | Issues Found |
|-----------------------|-------------|
| Placeholder artifacts | X           |
| Spelling & grammar    | X           |
| Terminology inconsistency | X       |
| Microcopy quality     | X           |
| Tone inconsistency    | X           |
| Broken links          | X           |
| **Total**             | **X**       |

### Content Quality: [Professional / Mostly Good / Needs Work / Draft Quality]

### Issues

#### COPY-001: Lorem ipsum on about page [Major]
- **Page**: /about
- **Location**: Second paragraph under "Our Story"
- **Found**: "Lorem ipsum dolor sit amet, consectetur adipiscing elit..."
- **Impact**: Destroys credibility — looks like an unfinished site
- **Fix**: Replace with actual company description

#### COPY-002: Inconsistent delete terminology [Minor]
- **Pages**: /cart uses "Remove", /wishlist uses "Delete",
  /account uses "Discard"
- **Recommendation**: Standardize on "Remove" for removing items
  from lists, "Delete" for permanent deletion

#### COPY-003: Typo in checkout heading [Minor]
- **Page**: /checkout
- **Found**: "Shiping Address" (missing 'p')
- **Fix**: "Shipping Address"

(continue...)

### Terminology Map
(table from COPY-3)

### Broken Links
| Page | Link Text | Target | Status |
|------|-----------|--------|--------|
| /about | "Read our blog" | /blog | 404 |
| /footer | "Privacy Policy" | /privacy | 200 ✓ |

### Recommendations
1. [Remove all placeholder text — immediate]
2. [Standardize terminology — this sprint]
3. [Fix typos — quick wins]
```

### After the Report

- "Want me to **create Jira tickets** for the content issues?" → `jira-create-bug`
  (placeholder artifacts and broken links as bugs; typos and terminology inconsistencies as tasks)

---

## Rules
- Read EVERY piece of visible text — don't skim
- Report exact location (page + element) for each issue
- Be specific about fixes — "change X to Y"
- Don't impose style preferences — report objective issues
  (typos, placeholders, inconsistencies) not "I'd word it differently"
- Quote the problematic text exactly
- Screenshot placeholder text and broken states
- **Always close the browser** (`browser_close`) after the session is complete
