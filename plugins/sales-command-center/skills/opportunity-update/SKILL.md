---
name: opportunity-update
description: "Automatically suggests and applies updates to Salesforce Opportunity fields (excluding SPICED fields) based on a meeting transcript. Use this skill whenever the user wants to: update an opportunity after a call, sync meeting insights to Salesforce fields, refresh opportunity data from a transcript, update deal details from a meeting, or says anything like 'update the opportunity from my call with [company]', 'sync my meeting notes to Salesforce', 'update the deal fields from my [meeting title]', 'refresh opportunity after my call', 'post-meeting Salesforce update', or 'update opportunity from transcript'. Always trigger when the user references a meeting or call AND wants any Opportunity fields (stage, close date, amount, next step, description, competitors, etc.) updated in Salesforce — even if phrased casually."
---

# Opportunity Auto-Update from Meeting Transcript

Fetches a Fathom transcript by meeting title, uses AI to extract and propose updates to standard Salesforce Opportunity fields (excluding SPICED fields), previews proposed changes for user confirmation, then writes accepted values back to the matching Opportunity via CData Connect.

**Required connectors:** Fathom and Salesforce through CData Connect AI

---

## Rendering Selectable Options

Whenever the skill needs the user to choose, render an interactive HTML widget. Each option is a clickable button that auto-submits the selection immediately.

Do **not** render an elicit-footer on any widget, and do not include a skip option.

---

## Connector / Tools Usage Guide

Always use the **CData Connect AI** connector for any tool call. Both Fathom and Salesforce are accessed through it.

---

## Step 1 — Resolve the Meeting

If the user provided a meeting title, use it directly.

If no meeting title was given, render a textbox widget to collect it.

---

## Step 2 — Search for the Meeting in Fathom

Search for the meeting by title using CData. If multiple matches exist, continue with the **single most relevant match**.

---

## Step 3 — Fetch the Transcript

Fetch the full transcript for the matched meeting.

If the transcript is unavailable or empty, inform the user and stop.

---

## Step 4 — Extract Opportunity Field Updates with AI

Use `getColumns` on the `Opportunity` table to retrieve all available fields at runtime. From those, consider only fields where `Readonly = false` and the API name does **not** start with `SPICED_` as candidates for update.

For each candidate field, use its name and description to infer whether the transcript contains relevant signal. Propose an update only where there is clear evidence.

> **Rules:**
> - Only suggest a field if there is **clear evidence** in the transcript.
> - If a field currently has a value, only suggest a change if the transcript gives a reason to update it.
> - Do **not** infer, fabricate, or guess. If it's not in the transcript, skip that field.
> - Assign each suggestion a confidence level: `High` / `Medium` / `Low`.

Present the proposed updates as a structured preview table:

```
Opportunity Update Preview — [Meeting Title]
─────────────────────────────────────────────────────────────
Field              | Current Value     | Proposed Value      | Confidence | Reason
─────────────────────────────────────────────────────────────
StageName          | Qualification     | Proposal/Price Quote| High       | Prospect asked for pricing
CloseDate          | 2025-06-30        | 2025-09-30          | Medium     | Mentioned end of Q3 deadline
NextStep           | Send deck         | Schedule POC        | High       | Agreed to POC in call
MainCompetitors__c | (empty)           | Salesforce, HubSpot | High       | Competitor named directly
...
─────────────────────────────────────────────────────────────
```

If no fields can be confidently suggested, tell the user and stop — do not proceed with empty updates.

Then render a confirmation widget (single-choice radio):

- **Question:** `"Do these proposed updates look correct?"`
- Option: `✅ Yes, update Salesforce` → submit
- Option + textbox: `✏️ Edit before saving` → Specify changes → submit (disable submit if textbox empty)
- Option: `❌ Cancel` → submit

---

## Step 5 — Find the Salesforce Opportunity

Extract the company/prospect name from the transcript, then query Salesforce:

1. Search `Opportunity` by `Name LIKE '%[company]%'` OR via `AccountId` where `Account.Name LIKE '%[company]%'`
2. Return `Id`, `Name`, `StageName`, `CloseDate`, `AccountId`
3. If **multiple** matches → render a selection widget (Name · Stage · Close Date) for the user to pick
4. If **no match** → offer to search by a different name or accept a manual Opportunity name

---

## Step 6 — Review Field Mapping

Query the Opportunity object columns to confirm each proposed field's API name exists and is writable (not `Readonly = true`).

Show the user a concise field-mapping confirmation table:

```
Field Mapping
───────────────────────────────────────
Label              | API Name              | Value to Write
───────────────────────────────────────
Stage              | StageName             | Proposal/Price Quote
Close Date         | CloseDate             | 2025-09-30
Next Step          | NextStep              | Schedule POC
Main Competitors   | MainCompetitors__c    | Salesforce, HubSpot
───────────────────────────────────────
```

Then render a proceed widget:

- `✅ Proceed with update` → submit
- `❌ Cancel` → submit

---

## Step 7 — Write Fields to Salesforce

For each field in the confirmed mapping:

- Use a **parameterized UPDATE** statement (never inline user values into SQL strings)
- Update **one field at a time** to isolate failures
- Format text fields as clean plain text (no markdown bullets — use semicolons or newlines for lists)
- Skip read-only fields silently

---

## Step 8 — Present Results

After all writes, display a results summary:

```
Opportunity Update Results — [Opportunity Name]
──────────────────────────────────────────────────────
Field               | Value Written           | Status
──────────────────────────────────────────────────────
StageName           | Proposal/Price Quote    | ✅ Updated
CloseDate           | 2025-09-30              | ✅ Updated
NextStep            | Schedule POC            | ✅ Updated
MainCompetitors__c  | Salesforce, HubSpot     | ✅ Updated
Amount              | (no signal in call)     | ⏭️ Skipped
──────────────────────────────────────────────────────
Opportunity ID: 006...
```

For any failures, explain the error and offer to retry or let the user fix it manually.

---

## Fields Intentionally Excluded

The following fields are **never updated** by this skill:

- All `SPICED_*` fields (handled by the **spiced-intelligence** skill)
- All `Readonly = true` fields (e.g. `Id`, `IsClosed`, `IsWon`, `ExpectedRevenue`, `ForecastCategory`, `CreatedDate`, `LastActivityDate`, etc.)

---

## Error Handling

| Scenario | Response |
|---|---|
| Transcript unavailable | Inform user; suggest checking Fathom recording status |
| No Opportunity match | Offer name search or manual ID entry |
| No fields to update | Inform user that the transcript didn't surface clear update signals |
| Field not writable | Skip silently; list as skipped in results |
| CData connection error | Call `getInstructions` again; surface the error detail |
| Partial update failure | Complete the rest; report failed fields individually |

---

## Valid Values Reference

Before proposing or writing any value to a picklist field, query the `PickListValues` table in Salesforce filtered by `TableName = Opportunity` to retrieve the active valid values — never assume or hardcode them as they vary per org.

---

## Example Triggers

- "Update the opportunity from my call with Acme Corp"
- "Sync insights from my [Discovery Call - TechCo] to Salesforce"
- "Update deal fields from today's meeting with GlobalInc"
- "Refresh Salesforce after my demo with Sarah at Globex"
- "Post-meeting opportunity update from my [QBR - MegaCorp] call"
- "What should I update on the opportunity based on my last call?"