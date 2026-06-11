---
name: pre-call-brief
description: Generate a comprehensive, AI-powered 10-section pre-call intelligence brief for any sales meeting. Use this skill whenever the user says "prep me for my call with [company]", "get me ready for my meeting with [company]", "pre-call brief [company]", "I'm meeting with [company]", "prepare for my [type] call", or any variation. This skill uses CData Connect AI to access all data sources — Apify for LinkedIn/web scraping, Exa for news and research, Apollo.io for prospect and company enrichment, Fathom for meeting transcripts, Salesforce for CRM context, and produces a full intelligence brief covering meeting overview, account snapshot, meeting context & objectives, stakeholder intelligence, market intelligence & pain, solution fit, agenda, risks & next steps, AI-generated brief layer, and sources. Always trigger this for meeting or call prep tasks — even casual requests like "help me prep for my call tomorrow" deserve the full brief.
---

# Pre-Call Intelligence Brief

A comprehensive 10-section pre-call intelligence brief. Pulls from web research (Apify + Exa), prospect/company data (Apollo.io), and CRM context (CData) to give you everything you need before walking into any meeting.

## Reference Files

| File | When to read |
|------|-------------|
| `references/output-format.md` | Phase 3 — before populating the 10 sections |
| `references/connector-guide.md` | Phase 2 — before making any tool calls |

---

## How It Works

```
┌──────────────────────────────────────────────────────────────────────┐
│                        PRE-CALL INTELLIGENCE BRIEF                   │
├──────────────────────────────────────────────────────────────────────┤
│  ALWAYS (standalone)                                                 │
│  ✓ Take meeting title input                                          │
│  ✓ Meeting Title lookup → Office365 Calendar via CData auto-fill     │
│  ✓ Company + meeting type confirmed via intake form                  │
│  ✓ Exa web search: news, funding, leadership, strategy               │
│  ✓ Apify (via CData): attendee profiles, company page                │
│  ✓ AI synthesis: hypotheses, talking points, objection handling      │
├──────────────────────────────────────────────────────────────────────┤
│  SUPERCHARGED (with connectors)                                      │
│  + Apollo.io: contact enrichment, company firmographics              │
│  + CData: CRM history, past activities, open opportunities           │
│  + Calendar (Office365 via CData): auto-pull meeting details         │
│  + Email (Office365 via CData): recent thread context                │
│  + Fathom: past meeting transcripts (max 2), prior objections/notes  │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Required Inputs

- **Meeting Title** *(required — used to look up the event in Office365 Calendar via CData Connect AI)*
- Company and contact name *(required — derived from calendar data or user input)*
- Meeting type (Discovery / Demo / Follow-up / QBR / Negotiation)
- Attendee names and titles *(auto-filled from calendar if found)*
- Meeting date/time *(auto-filled from calendar if found)*
- Any context, notes, or emails to include *(auto-filled from calendar description if present)*

---

## Meeting Title Widget — Meeting Title Lookup (Render First)

**This is the very first thing to render — before the intake form.** Show a minimal widget asking only for the meeting title, then use Microsoft 365 to look up that calendar event and pre-fill the intake form.

### Widget Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| Meeting Title | Text input | ✅ Yes | Validate non-empty before submit |
| Submit Button | Button | — | Label: "Find Meeting →" |

### Widget Behavior

- **Meeting Title** must be non-empty to enable Submit; show inline error if blank on submit.
- On submit, call `sendPrompt()` with payload:
  ```
  MEETING_TITLE::{"title":"<user input>"}
  ```
- If user's original message already contained a meeting title or company name, pre-fill the field.

### Widget Styling

- Minimal card layout, rounded corners, subtle drop shadow
- Submit button: `#4F46E5` (indigo), white text
- Required field label: red asterisk `*`
- Font: Inter or system-ui, single column, compact
- Subtitle: "Enter the meeting title to look it up in your calendar and auto-fill your brief."

### After Submission

Once `MEETING_TITLE::` arrives:
1. **Search Office365 Calendar via CData Connect AI** using the provided title keyword (partial match — use a SQL LIKE filter, e.g. WHERE Subject LIKE '%<keyword>%').
2. Retrieve the best-matching upcoming event and extract:
   - `calendarTitle` — full event title
   - `calendarDateTime` — start date/time
   - `calendarAttendees` — all attendees (name + email)
   - `calendarDescription` — event body/description (may contain company info, agenda, links)
   - `calendarOrganizer` — organiser name/email
3. Derive `company` from the meeting title or attendee domains if not explicit.
4. Proceed to render the **Intake Widget** (Phase 1), pre-filled with all calendar data.

---

## Intake Widget — Render After Calendar Lookup

**Render this widget after the Office365 calendar data has been fetched via CData Connect AI.** Pre-fill every field using the calendar data retrieved in Phase 0. The user can review, correct, and submit to confirm.

### Widget Fields

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| Company Name | Text input | ✅ Yes | Pre-fill from calendar data; validate non-empty |
| Meeting Type | Dropdown | No | Options: Discovery, Demo, Follow-up, QBR, Negotiation, Other; infer from title/description if possible |
| Attendee Names & Titles | Textarea | No | Pre-fill from calendar attendees; format: "Jane Doe – VP Sales" |
| Meeting Date & Time | DateTime-local input | No | Pre-fill from calendar event start time |
| Opportunity Name | Textarea | No | Pre-fill from calendar data |
| Additional Context / Notes | Textarea | No | Pre-fill with calendar event description/body if present |
| Submit Button | Button | — | Label: "Generate Pre-Call Intelligence Brief →" |

### Widget Behavior

- **All fields pre-filled** from calendar data where available; user may edit any field before submitting.
- Show a subtle banner at the top: `✅ Meeting found: "[calendarTitle]" — details pre-filled from your calendar.`
- If no calendar event was found, show: `⚠️ No matching meeting found in your calendar. Please fill in the details below.` and render the form empty.
- **Company Name** must be non-empty to enable Submit; show inline error if blank on submit.
- On submit, call `sendPrompt()` with a structured payload:
  ```
  INPUTS::{"company":"Acme Corp","meetingType":"Discovery","attendees":"Jane Doe – VP Sales","dateTime":"2025-05-10T14:00","context":"Prior email intro attached","calendarEventId":"<id if available>"}
  ```
- Optional fields left blank are omitted from the payload.

### Widget Styling

- Card layout, rounded corners, subtle drop shadow
- Submit button: `#4F46E5` (indigo), white text
- Required field label: red asterisk `*`
- Font: Inter or system-ui, single column, compact

### After Submission

Once `INPUTS::` arrives, parse the JSON and proceed directly to Phase 2. Do not ask for further input unless a field is ambiguous.

---

## Execution Flow

### Phase 0 — Meeting Title Lookup

```
1. Render the Phase 0 widget (see "Meeting Title Widget — Meeting Title Lookup" above).
   → Pre-fill Meeting Title if already mentioned by the user.
   → Wait for MEETING_TITLE:: payload before proceeding.

2. Once MEETING_TITLE:: received:
   → Use CData Connect AI (queryData) with the Office365 connection
   → Query calender events using the keywords from meeting title
   → Query recent emails with keywords from meeting title
   → Extract: title, start datetime, attendees (names + emails), description, organiser, and any additional context
   → Store as calendarData for pre-filling the intake form
```

### Phase 1 — Intake

```
1. Render the intake widget pre-filled with calendar data from Phase 0.
   → Show "✅ Meeting found" or "⚠️ No match" banner as appropriate.
   → Wait for the user to confirm/edit and submit the form.

2. Once INPUTS:: payload received, parse JSON and extract:
   - company (required)
   - meetingType, attendees, dateTime, opportunityName, context, calendarEventId (all optional)

3. If CData is available:
   → getCatalogs() to discover connection names
   → Query Account, Contact, and Opportunity tables
   → Note: 0 CRM contacts ≠ net-new account; always query Opportunity independently
   → Use Opportunity name from calender data if available
```

### Phase 2 — Research

Read `references/connector-guide.md` for what to fetch from each source. Run all research in parallel where possible.
Do not skip steps mentioned in `references/connector-guide.md`.

## Non-obvious filters: 
- Apollo OrganizationEnrichment requires WHERE [domain] = 'company.com' (not name).
- Apollo OrganizationJobPostings requires WHERE [OrganizationId]. Get the ID from OrganizationEnrichment first.
- Exa search requires WHERE [query] = 'your search string'.
- Apollo People title filters must be exact match strings; LIKE on organization name causes pagination errors. 


### Phase 3 — Synthesis

**Read `references/output-format.md` for the full 10-section template.**
**Read `/mnt/skills/public/docx/SKILL.md` before generating the output file.**

```
1. Merge all data sources into unified context
2. Fill all 10 sections using the template in references/output-format.md
3. Flag sparse or uncertain sections with *[inferred] or *[unconfirmed]
4. Generate Section 9 (AI Brief Layer) last, after all other sections are complete
5. Generate the output as a .docx file following the docx skill instructions
   → File name: PreCall_Brief_[CompanyName]_[YYYY-MM-DD].docx
   → Save to /home/claude/, validate, copy to /mnt/user-data/outputs/
6. Call present_files so the user gets a download link
```

**Writing style:** Write in concise phrases and short declarative sentences throughout. No paragraph prose anywhere in the document — if a point takes more than one sentence to make, express it as two bullets instead. Reason fully, write the conclusion only.

### Phase 4 — Review & Revisions

```
1. After presenting the .docx file to the user, render an interactive widget using show_widget with:
   - A textarea for "Requested Changes" (placeholder: "Describe any changes, additions, or corrections…")
   - A "Apply Changes" button (indigo #4F46E5) that calls sendPrompt() with:
     CHANGES::{"changes": "<user input text>"}
   - A secondary "Continue to Email Draft →" button (green #059669) that calls sendPrompt() with:
     ACTION::{"action": "draft_email"}
   - Widget title: "Review Your Pre-Call Brief"
   - Subtitle: "Would you like to make any changes, or shall we draft the team email?"

2. Widget Behavior:
   - "Apply Changes" button disabled if textarea is empty
   - Both buttons mutually exclusive — clicking one dismisses the other
   - Card layout, rounded corners, subtle drop shadow, single column

3. After creating the widget, always say to user to fill the above widget or directly give answer in the chat input.

4. If CHANGES:: payload received:
   → Parse requested changes from JSON
   → Apply all changes to the .docx file (re-generate or patch the relevant sections)
   → Save updated file as PreCall_Brief_[CompanyName]_[YYYY-MM-DD].docx
   → Call present_files with the updated document
   → Re-render the Phase 4 widget again so the user can request further changes or proceed
   → After rendering the widget, always say to user to fill the above widget or directly give answer in the chat input

5. If ACTION::{"action": "draft_email"} received:
   → Proceed directly to Phase 5
```

### Phase 5 — Send Email to Team (Office365)

```
1. Use CData Connect AI with the Office365 connection for sending email.
     Confirm tools via getProcedures() first. Call queryData or executeProcedure as required by the CData Office365
     send-mail interface.
     Use Office365 `@Attachments` parameter for passing document base64 string, while attaching a document in mail.
     `@ContentType` parameter should have value in lowercase.

2. Gather the following metadata (from Phase 1 inputs and calendar data):
   - Company Name
   - Meeting Title (from calendar or intake form)
   - Meeting Date & Time
   - Attendees (names and titles)
   - Opportunity name/deal (from CRM if available, else omit)

3. Compose email template:
   - To: all meeting attendees (use email addresses extracted from Office365, fetched in Phase 0 via CData)
   - Subject: "Pre-Call Brief – [Meeting Title] with [Company Name] – [Date]"
   - Body:
     Hi team,

     Please find attached the Pre-Call Intelligence Brief for our upcoming meeting.

     📋 Meeting Details
     • Company: [Company Name]
     • Meeting: [Meeting Title]
     • Date & Time: [Meeting Date & Time]
     • Attendees: [Attendee 1 – Title, Attendee 2 – Title, …]
     [• Opportunity: [Opportunity Name]  ← include only if CRM data available]

     Please review the attached brief ahead of the call. It covers company background,
     stakeholder profiles, talking points, objection handling, and recommended next steps.

     Best,
     [Sender Name from Office365 profile]

   - Attachment: the final PreCall_Brief_[CompanyName]_[YYYY-MM-DD].docx
```

6. Error handling:
   - If attendee emails cannot be resolved, list the names and ask the user to provide email addresses
   - If the CData Office365 send call fails, surface the error message from CData and suggest the user send manually

---

## Meeting Type Adjustments

| Meeting Type | Agenda Emphasis | Key Focus |
|-------------|----------------|-----------|
| **Discovery** | Questions 60%, Talking 40% | Pain, priorities, decision process |
| **Demo** | Show 50%, Discover 30%, Next Steps 20% | Tailored use case, technical fit |
| **Follow-up** | Progress 40%, Objections 40%, Close 20% | Address blockers, advance deal |
| **QBR** | Review 50%, Expansion 30%, Roadmap 20% | Value delivered, upsell signal |
| **Negotiation** | Value 40%, Objections 40%, Close 20% | Justify ROI, handle pushback |