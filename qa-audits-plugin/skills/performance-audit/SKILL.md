---
name: performance-audit
description: >
  Evaluate perceived performance of a web application using Playwright
  MCP. Focuses on what the USER feels — loading states, feedback timing,
  layout stability, progressive rendering — not synthetic Lighthouse
  scores. Use when user says "performance check", "is this fast",
  "feels slow", "loading speed", "performance audit", "Core Web Vitals",
  "layout shift", "slow page", "page load test", "does it feel snappy",
  "responsiveness check", or "performance perception". Also trigger when
  user mentions jank, lag, or stuttering.
---

# Performance Perception Audit

You evaluate how fast a web application FEELS to a user. Synthetic metrics
matter, but perceived performance is what shapes the user experience.
A page that loads in 2s but shows nothing for 1.8s feels slower than a
page that loads in 3s but shows a skeleton in 200ms.

**First**, read: `qa-common/references/qa-reference.md (in this plugin)`

---

## Setup

1. **URL** (default: `http://localhost:3000`)
2. **Pages to test**: all navigable or specific? (default: all)
3. **Network condition**: fast / average / slow?
   (simulate via Playwright throttling if needed)

---

## Evaluation Areas

### PERF-1: Initial Page Load

For each page, navigate and observe:

- **Time to first content**: how quickly does ANYTHING appear?
  - Instant (<200ms): Excellent
  - Fast (<1s): Good
  - Noticeable (1-3s): Acceptable
  - Slow (>3s): Problem
- **Progressive rendering**: does content appear incrementally?
  Or does the page stay blank until everything loads?
- **Skeleton screens / loading placeholders**: present during data fetch?
- **Above-the-fold content**: is the visible area populated first?
- **Font loading**: does text flash (FOUT) or remain invisible (FOIT)?

### PERF-2: Layout Stability (CLS-related)

During and after page load:
- Do elements jump or shift position as content loads?
- Do images cause reflow? (dimensions set in HTML, or images pop in?)
- Does late-loading content push other content down?
- Does ad/banner insertion shift the layout?
- Do web fonts cause text to re-layout?

**Test method**: Navigate to page, observe the accessibility tree at 0.5s
intervals for 5s. Note any elements that change position.

### PERF-3: Interaction Responsiveness

For every interactive element:

- **Click → response time**:
  - <100ms: feels instant (ideal)
  - 100-300ms: noticeable but acceptable
  - >300ms: feels slow — needs loading indicator
- **After button click**: immediate visual change? (button state, spinner)
- **Form submission**: instant feedback or delay?
- **Search / filter**: results appear as you type or after delay?
- **Scroll**: smooth or janky? (check long lists, infinite scroll)
- **Animations**: smooth or stuttering?

### PERF-4: Loading States & Feedback

For every action that involves a network request:

| Action | Has loading state? | Loading type | Acceptable? |
|--------|--------------------|--------------|-------------|
| Page navigation | ? | Spinner / Skeleton / None | ? |
| Form submit | ? | Button disabled+spinner / None | ? |
| Data fetch | ? | Skeleton / Spinner / None | ? |
| Search | ? | Inline spinner / None | ? |
| File upload | ? | Progress bar / None | ? |
| Delete action | ? | Optimistic / None | ? |

**Missing loading states** = user thinks the app is broken.

### PERF-5: Perceived Speed Patterns

Check for performance UX patterns:
- **Optimistic updates**: does the UI update before the server confirms?
  (adding a like, toggling a switch)
- **Lazy loading**: images below the fold load on scroll?
- **Pagination / infinite scroll**: large lists handled efficiently?
- **Caching**: does going Back re-fetch or use cache?
- **Prefetching**: do links preload on hover?
- **Debounced search**: typing doesn't trigger a request per keystroke?

### PERF-6: Heavy Page Stress

Identify and test potentially heavy pages:
- Pages with large data tables (100+ rows)
- Pages with many images (gallery, product grid)
- Pages with complex forms (20+ fields)
- Pages with real-time updates (dashboards, chat)
- Pages with maps or embedded content

Check: does performance degrade noticeably?

---

## Perception Rating Scale

| Rating | Meaning | Typical load feel |
|--------|---------|-------------------|
| A | Instant | Content appears < 500ms, no layout shift |
| B | Fast | Content within 1s, minimal shift, has loading states |
| C | Acceptable | Content within 2s, some shift, loading states present |
| D | Slow | Content > 2s, noticeable shifts, missing loading states |
| F | Broken | Content > 5s, major shifts, no feedback, feels stuck |

---

## Report

Save to `qa/reports/{project}/performance/run-[date].md`:

```
## Performance Perception Audit — [URL]
### Date: [date]

### Page Ratings
| Page          | Load | Stability | Responsiveness | Loading States | Grade |
|---------------|------|-----------|----------------|----------------|-------|
| /             | A    | B         | A              | A              | A     |
| /products     | C    | D         | B              | C              | C     |
| /checkout     | B    | B         | C              | D              | C     |

### Overall: [A-F]

### Issues

#### PERF-001: No loading state on product list fetch [D]
- **Page**: /products
- **Action**: Navigate to page
- **Problem**: Page shows blank white space for ~2s while products
  load from API. No skeleton, no spinner, no indication anything
  is happening. Users may click away thinking the page is broken.
- **Recommendation**: Add skeleton cards matching the product grid
  layout. Show them within 200ms of navigation.

#### PERF-002: Major layout shift when images load [D]
- **Page**: /products
- **Problem**: Product images have no set dimensions. As each image
  loads, the grid re-layouts, pushing content down. Cumulative
  shift is significant.
- **Recommendation**: Set explicit width/height on `<img>` elements
  or use aspect-ratio CSS. Add placeholder background color.

(continue...)

### Loading State Inventory
(table from PERF-4)

### Recommendations
1. [Add skeleton screens — biggest perceived improvement]
2. [Set image dimensions — eliminates layout shift]
3. ...
```

### After the Report

- "Want me to **create Jira tickets** for the performance issues?" → `jira-create-bug`
  (grade D-F issues as bugs; grade C as improvements)

---

## Rules
- Focus on PERCEPTION, not synthetic metrics
- Test as a user would — don't use DevTools timing unless needed
- Every "feels slow" moment should have a concrete fix
- Screenshot layout shifts and missing loading states
- Rate each page independently — homepage might be A while checkout is D
- **Always close the browser** (`browser_close`) after the session is complete
