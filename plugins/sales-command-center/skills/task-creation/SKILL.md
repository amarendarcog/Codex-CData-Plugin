---
name: task-creation
description: "Automatically creates follow-up tasks in Salesforce from a Fathom meeting transcript. Use this skill whenever a user wants to: auto-create tasks from a call, generate Salesforce tasks after a meeting, sync call outcomes to CRM, create follow-ups from a transcript, or says anything like \"create tasks from my call with [company]\", \"log follow-ups from my meeting\", \"sync my call notes to Salesforce\", \"what tasks came out of my call with X\", or \"create action items in Salesforce from my [meeting title] meeting\". Always trigger this skill when the user references a meeting or call AND wants tasks, action items, or follow-ups created in Salesforce — even if they phrase it casually."
---

# Auto Task Creation from Call Outcomes

Extracts action items and follow-up tasks from a Fathom meeting transcript and creates them as Tasks in Salesforce via CData Connect.

---

## Rendering Selectable Options

Whenever the skill needs the user to choose, render an interactive HTML widget. Each option is clickable option that auto-submit the selection immediately.

Do not render an elicit-footer for the widget

---

## Connector Guide

- Always use **CData Connect AI** connector for fathom
- Always follow this sequence before writing SQL: getCatalogs → getSchemas → getTables → inspect columns via schemaOnly: true query — then write the actual data query. Never assume schema or column names.
- The transcript column in list_meetings is typically null — always fetch transcript lines from the transcript table

---

## Step 1 — Get Meeting Title from User

if the user provided a meeting title, Use the list meetings tool to fetch recent meetings, then pick the **single most relevant match** for the provided meeting title input. if multiple matches found, continue with only one of meeting from the matches.

else If the user hasn't provided a meeting title, continue with the latest meeting.

---

## Step 2 — Meeting confirmation from User

Render a Yes/No widget — no other text:

- Question: "Is this the meeting you'd like to follow up on?"
- Option 1: `✅ Yes — [Title] · [Date]`
- Option 2: `❌ No, show me other options`

**If the user selects No, or replies with anything indicating no** (e.g. "nope", "wrong one", "not that one"), render a new widget listing the **10 most recent meetings** — no other text:

- Question: "Select the meeting you'd like to follow up on:"
- One Option per meeting, labelled `[Title] · [Date]`, Prompt value is "Proceed with [Title] on [Date]", { submit: true }

---

## Step 3 — Fetch Transcript

Once a meeting is confirmed, fetch its full transcript using fathom. If the transcript is unavailable or empty, inform the user and stop.

---

## Step 4 — Extract Tasks with AI

Using the transcript + summary, identify all **follow-up tasks**. For each task extract:

| Field | How to determine |
|---|---|
| `Subject` | Clear, action-oriented title (e.g. "Send pricing proposal to Acme") |
| `Description` | 1–2 sentence context from transcript. Include speaker name if relevant. |
| `Priority` | High / Normal / Low — infer from urgency language ("ASAP", "critical", "when you get a chance") |
| `Due_Date` | If a specific date/timeframe was mentioned (e.g. "by end of week" → next Friday). Otherwise leave null. |
| `WhoId_hint` | Person's name mentioned (prospect/customer) — used to look up Contact in Salesforce |
| `WhatId_hint` | Company / account name — used to look up Account in Salesforce |
| `Type` | One of: `Call`, `Email`, `Send`, `Follow-up`, `Demo`, `Proposal`, `Other` |
| `Owner` | Speaker who committed to the task (if determinable) |

**Good task signals:**
- Explicit commitments: "I'll send you…", "We'll follow up on…", "Can you share…"
- Named next steps: "Next step is…", "Action item:", "We need to…"
- Promises with deadlines: "by Friday", "this week", "before the next call"

**Do not create tasks for:**
- General discussion points with no clear owner or action
- Things already completed during the call
- Vague statements without a responsible party

Show the extracted tasks to the user in a clear table before creating them. Ask: **"Should I create these [N] tasks in Salesforce?"** Wait for confirmation.

---

## Step 4 — Prepare Salesforce Connection

Before inserting, get driver-specific instructions

Then discover available catalogs/schemas if needed

Get the Task columns to confirm field names

Key Salesforce Task fields:
- `Subject` (required) — task title
- `Description` — detailed notes
- `Priority` — "High", "Normal", "Low"
- `Status` — default to "Not Started"
- `ActivityDate` — due date (YYYY-MM-DD)
- `WhoId` — Contact or Lead ID (optional)
- `WhatId` — Account, Opportunity, or other object ID (optional)
- `Type` — task category

To resolve `WhoId` / `WhatId`, query Salesforce, search the Contact table with the name.

If no match is found, create the task without `WhoId`/`WhatId` and note it for the user.

---

## Step 5 — Create Tasks in Salesforce

For each confirmed task, insert using parameterized queries. Pass parameters separately, never inline user-derived values into SQL strings.

Create tasks one at a time (not in a batch) so each failure is isolated and reported.

---

## Step 6 — Present Results

After all inserts, show a summary table:

| # | Task | Priority | Due | Status | Salesforce ID |
|---|------|----------|-----|--------|---------------|
| 1 | Send pricing proposal | High | 2026-05-09 | ✅ Created | 00T... |
| 2 | Schedule demo with IT team | Normal | — | ✅ Created | 00T... |
| 3 | Share case studies | Low | — | ❌ Failed (no Contact found) | — |

For any failures, explain what went wrong and offer to retry or let the user fix it manually.

---

## Error Handling

| Scenario | Response |
|---|---|
| No tasks extracted | Inform user; offer to show full summary instead |
| Salesforce connection error | Call getInstructions again; show error detail |
| Duplicate task concern | Warn user if subject exactly matches an existing open task |
| Missing required fields | Use sensible defaults (`Status = "Not Started"`, `Priority = "Normal"`) |

---

## Example Triggers

- "Create tasks in Salesforce from my call with Acme Corp"
- "Auto-create follow-ups from my [Discovery Call - TechCo] meeting"
- "Log action items from today's demo to Salesforce"
- "Sync my call outcomes to CRM from the Q2 pipeline review meeting"
- "What follow-up tasks should I create from my call with Sarah at Globex?"