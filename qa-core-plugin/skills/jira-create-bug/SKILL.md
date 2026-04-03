---
name: jira-create-bug
description: >
  Create Jira issues from bugs found during testing. Maps severity/priority
  from QA Toolkit format to Jira fields and creates tickets via REST API.
  Use when user says "create Jira ticket", "file a bug in Jira", "send to
  Jira", "create ticket", "log this in Jira", "create issues for bugs",
  "Jira", or after any testing skill when bugs are found and user confirms.
  Also trigger when user says "yes" after a testing skill suggests creating
  Jira tickets.
---

# Jira Bug Ticket Creator

You create Jira issues from bugs discovered during QA testing sessions.
You EXECUTE real API calls via curl — tickets are created automatically
after user confirms the list.

**First**, read: `qa-common/references/qa-reference.md (in this plugin)`

---

## First-Time Setup

If `qa/jira-config.json` does NOT exist, you MUST set it up:

### 1. Verify project key

The user will give a project name, but Jira project keys are often
abbreviated. **Always verify the key exists before creating tickets:**

```bash
source ~/.bashrc && curl -s -H "Authorization: Bearer $JIRA_PAT" \
  "https://jira.talrace.com/rest/api/2/project/<KEY>" | node -e \
  "let d='';process.stdin.on('data',c=>d+=c);process.stdin.on('end',()=>{const j=JSON.parse(d);console.log(j.key?'OK: '+j.key+' — '+j.name:'NOT FOUND: '+j.errorMessages)})"
```

If 404 — list available projects to help the user find the right key:

```bash
source ~/.bashrc && curl -s -H "Authorization: Bearer $JIRA_PAT" \
  "https://jira.talrace.com/rest/api/2/project" | node -e \
  "let d='';process.stdin.on('data',c=>d+=c);process.stdin.on('end',()=>{JSON.parse(d).forEach(p=>console.log(p.key+' — '+p.name))})"
```

### 2. Discover available priorities

Jira instances have different priority schemes. **Always fetch actual
priorities** before mapping:

```bash
source ~/.bashrc && curl -s -H "Authorization: Bearer $JIRA_PAT" \
  "https://jira.talrace.com/rest/api/2/priority" | node -e \
  "let d='';process.stdin.on('data',c=>d+=c);process.stdin.on('end',()=>{JSON.parse(d).forEach(p=>console.log(p.name+' (id:'+p.id+')'))})"
```

Save the result to `qa/jira-config.json` so you don't have to fetch every time.

### 3. Create config file

Copy the template from `jira-create-bug/jira-config.example.json` (in this plugin)
and save as `qa/jira-config.json`:
```json
{
  "jira_url": "https://jira.talrace.com",
  "project_key": "<VERIFIED KEY>",
  "issue_type": "Bug",
  "default_labels": ["qa-toolkit"],
  "priorities": ["Highest", "High", "Medium", "Low", "Lowest"],
  "custom_fields": {}
}
```

### 4. Check auth environment variable

JIRA_PAT is stored in `~/.bashrc`. Every Bash command that uses it
MUST start with `source ~/.bashrc &&` because each Bash tool invocation
starts a fresh shell that does not auto-source bashrc.

```bash
source ~/.bashrc && echo "JIRA_PAT=${JIRA_PAT:+(set)}"
```

If NOT set, tell the user:
```
Jira auth is not configured. Set your Personal Access Token:

  echo 'export JIRA_PAT="your-token"' >> ~/.bashrc

Generate one at: Jira → Profile → Personal Access Tokens → Create token
```

### 5. Verify connection

```bash
source ~/.bashrc && curl -s -H "Authorization: Bearer $JIRA_PAT" \
  "https://jira.talrace.com/rest/api/2/myself" | node -e \
  "let d='';process.stdin.on('data',c=>d+=c);process.stdin.on('end',()=>{const j=JSON.parse(d);console.log('Connected as: '+j.displayName+' ('+j.emailAddress+')')})"
```

---

## Returning Use

If `qa/jira-config.json` exists AND JIRA_PAT is set:
- Load config silently
- Skip setup
- Go straight to bug collection
- **Always ask for project key** before creating tickets — the user may
  work with different Jira projects. Show the last used key as default:
  "Jira project key? [NOD]:" — user can confirm or type a new one.

---

## Input Sources

### Source A: From Current Testing Session (most common)
After a testing skill found bugs in the same conversation — parse them
directly from the report that was just generated.

### Source B: From Test Report File
Read bugs from `qa/reports/{project}/{type}/run-*.md` (most recent, or user-specified).

### Source C: From Bug YAML Files
Read structured bugs from `qa/bugs/*.yaml`.

### Source D: From Conversation
User describes a bug verbally — structure it into a ticket.

---

## Severity → Jira Priority Mapping

Jira priorities vary by instance. The mapping below is for the current
Jira instance (jira.talrace.com). If priorities differ, fetch them
from `/rest/api/2/priority` and adjust.

| QA Severity | Jira Priority | Jira Label   |
|-------------|---------------|--------------|
| S0 Blocker  | Highest       | severity-S0  |
| S1 Critical | Highest       | severity-S1  |
| S2 Major    | High          | severity-S2  |
| S3 Minor    | Medium        | severity-S3  |
| S4 Trivial  | Low           | severity-S4  |

When the user filters by priority (e.g., "all except below medium"),
map that to the Jira priority names: exclude Low and Lowest.

---

## Execution — Step by Step

### Step 1: Collect & Present Bugs

Parse bugs from the source. **Before showing the list, ask for the project key:**

```
Jira project key? [NOD]: ___
```

If the user gives a name that doesn't match the key (e.g., "NODER" when
the key is "NOD"), verify via API and suggest the correct key.

Update `qa/jira-config.json` with the chosen key.

Then show the list and ask for confirmation:

```
## Bugs to Create in Jira
### Project: [KEY] @ jira.talrace.com

| # | Bug ID  | Title                     | Severity | → Jira Priority |
|---|---------|---------------------------|----------|-----------------|
| 1 | BUG-001 | Font size accepts 0       | S2       | High            |
| 2 | BUG-002 | Ctrl+K broken             | S2       | High            |
| 3 | BUG-003 | Typo in heading           | S4       | Low             |

Create all 3? (or specify: "1,2", "S0-S2 only", "all except below medium", etc.)
```

**Check for duplicates**: read `qa/jira-log.md` if it exists. If a bug ID
was already exported, warn: "BUG-001 was already exported as NOD-3442. Skip?"

**WAIT for user confirmation. Do NOT proceed without explicit "yes" or selection.**

### Step 2: Create Tickets (EXECUTE — not describe)

**IMPORTANT**: Every `bash` command must start with `source ~/.bashrc &&`
to load the JIRA_PAT environment variable.

For EACH confirmed bug:

**2a.** Build the issue payload JSON and write to a temp file:

```bash
source ~/.bashrc && cat > /tmp/jira-issue-BUG-001.json << 'PAYLOAD'
{
  "fields": {
    "project": { "key": "NOD" },
    "summary": "Font size accepts 0 and negative values — no minimum validation",
    "issuetype": { "name": "Bug" },
    "priority": { "name": "High" },
    "labels": ["qa-toolkit", "severity-S2", "formatting"],
    "description": "h3. Bug Report (QA Toolkit)\n\n*Severity:* S2 — Major\n*Module:* FORMATTING\n*Environment:* Chromium / 1440x900 / Windows 10\n*Found by:* QA Toolkit — exploratory-testing (2026-04-02)\n\nh3. Steps to Reproduce\n# Step 1\n# Step 2\n\nh3. Expected Result\nExpected text\n\nh3. Actual Result\nActual text\n\nh3. Evidence\nSee attachment\n\nh3. Notes\nAdditional context"
  }
}
PAYLOAD
```

**2b.** Create the ticket via curl (NOT via helper scripts — direct curl
is more reliable across environments):

```bash
source ~/.bashrc && curl -s -w "\n%{http_code}" -X POST \
  "https://jira.talrace.com/rest/api/2/issue" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $JIRA_PAT" \
  -d @/tmp/jira-issue-BUG-001.json
```

**2c.** Parse the response — extract the issue `key` (e.g., `NOD-3442`)
from the JSON: `{"id":"26806","key":"NOD-3442","self":"..."}`.

**2d.** If screenshots exist for this bug (in `qa/screenshots/`), attach them:

```bash
source ~/.bashrc && curl -s -o /dev/null -w "%{http_code}" -X POST \
  "https://jira.talrace.com/rest/api/2/issue/NOD-3442/attachments" \
  -H "X-Atlassian-Token: no-check" \
  -H "Authorization: Bearer $JIRA_PAT" \
  -F "file=@qa/screenshots/test-font-size-zero.png"
```

Multiple files can be attached in one call with multiple `-F` flags.

**2e.** If a request fails, show the error and continue with the next bug.

**2f.** Clean up temp files after all tickets are created:
```bash
rm /tmp/jira-issue-BUG-*.json
```

**Parallelization**: Create all payload files first, then fire all curl
requests in parallel (multiple Bash tool calls in one message). This is
significantly faster than sequential creation.

### Step 3: Show Results

After all tickets are created:

```
## Jira Tickets Created

| # | Bug ID  | Jira Key   | Title                     | Priority | Link |
|---|---------|------------|---------------------------|----------|------|
| 1 | BUG-001 | NOD-3442   | Font size validation      | High     | https://jira.talrace.com/browse/NOD-3442 |
| 2 | BUG-002 | NOD-3443   | Ctrl+K broken             | Medium   | https://jira.talrace.com/browse/NOD-3443 |
| ✗ | BUG-003 | —          | Typo in heading           | —        | FAILED: field "priority" value invalid |

Created: 2/3 | Failed: 1/3 | Screenshots attached: 2
```

### Step 4: Log & Update Report

**Always** append to `qa/jira-log.md` (create if doesn't exist):
```markdown
## Jira Export — 2026-04-02
### Source: run-2026-04-02-exploratory.md

| Bug ID  | Jira Key  | Link |
|---------|-----------|------|
| BUG-001 | NOD-3442  | https://jira.talrace.com/browse/NOD-3442 |
| BUG-002 | NOD-3443  | https://jira.talrace.com/browse/NOD-3443 |
```

Then ask: "Add Jira links to the test report?"

If yes, append a `### Jira Tickets` section to the source report in `qa/reports/`.

---

## Jira Description Format

Use Jira wiki markup for the `description` field:

```
h3. Bug Report (QA Toolkit)

*Severity:* S{N} — {severity_name}
*Module:* {module}
*Environment:* {browser / viewport / OS}
*Found by:* QA Toolkit — {skill_name} ({date})

h3. Steps to Reproduce
# {step 1}
# {step 2}
# {step 3}

h3. Expected Result
{expected}

h3. Actual Result
{actual}

h3. Evidence
{screenshot path or "See attachment"}

h3. Notes
{Additional context, suggested fix if any}
```

When building the JSON payload, escape newlines as `\n` and double quotes
as `\"` in the description string. Use heredoc with `<< 'PAYLOAD'`
(single-quoted delimiter) to avoid shell variable expansion inside the JSON.

---

## Error Handling

| HTTP Code | Meaning | Action |
|-----------|---------|--------|
| 200/201   | Created | Extract key from response JSON, continue |
| 401/403   | Auth failed | Stop. Tell user: `source ~/.bashrc && echo $JIRA_PAT` to verify token is loaded |
| 400       | Bad payload | Show Jira's error message. Common cause: priority name doesn't match. Fetch valid priorities and retry |
| 404       | Project not found | List available projects via API, ask user to pick the right key |
| 429       | Rate limited | Wait 5 seconds, retry |
| Other     | Unknown | Show full response, skip this bug, continue with next |

If auth fails on the first ticket, STOP — don't try the rest (they'll all fail).

**Common pitfall**: Jira priority names are instance-specific.
"Blocker/Critical/Major/Minor/Trivial" is NOT universal. Always use the
names from `/rest/api/2/priority`. On jira.talrace.com the priorities are:
Highest, High, Medium, Low, Lowest.

---

## Environment Notes

- **Shell**: Each Bash tool call starts a fresh shell. `~/.bashrc` is NOT
  auto-sourced. Every command using `$JIRA_PAT` must begin with
  `source ~/.bashrc &&`.
- **JSON parsing**: Use `node -e` (NOT python3 — not installed on this machine).
- **Auth method**: Jira Server with Personal Access Token (Bearer auth).
  Basic auth (email+token) is for Jira Cloud only.
- **File paths**: Use forward slashes. Prefix `qa/` paths relative to project root.

---

## Rules
- **EXECUTE curl commands** — do not just show them for the user to run
- **source ~/.bashrc** before every Bash command that needs JIRA_PAT
- **Verify project key** via API before creating tickets — names ≠ keys
- **Fetch priorities** from API on first use — don't assume standard names
- **Never save auth tokens to files** — only in ~/.bashrc env var
- **Always confirm before creating** — show bug list, wait for explicit "yes"
- **Write payload to /tmp/, clean up after** — don't leave temp files
- **Attach screenshots** when they exist in `qa/screenshots/`
- **Log every export** to `qa/jira-log.md` — append, never overwrite
- **No duplicates** — check jira-log.md before creating
- **Continue on single failure** — one failed ticket shouldn't block the rest
- **Parallelize curl calls** — create payloads first, then fire all in parallel
