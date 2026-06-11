---
name: spiced-intelligence
description: "Automatically maps a Fathom meeting transcript to the SPICED sales framework and populates the corresponding Salesforce opportunity fields via CData Connect. Use this skill whenever the user wants to: fill in SPICED fields from a call, update Salesforce with SPICED data after a meeting, auto-populate opportunity scoring from a transcript, map call notes to SPICED, or says anything like 'fill SPICED from my call with [company]', 'update SPICED fields from my meeting', 'populate Salesforce SPICED from my [meeting title]', 'auto-fill opportunity fields from my call', 'SPICED update from my discovery call', or 'log SPICED to Salesforce from my meeting'. Always trigger when the user references a meeting or call AND wants SPICED fields, opportunity data, or qualification framework data updated in Salesforce — even if phrased casually."
---

# SPICED Auto-Populate from Meeting Transcript

Fetches a Fathom transcript by meeting title, maps it to the SPICED sales qualification framework, previews the extracted data for user confirmation, then writes it back to the matching Salesforce Opportunity via CData Connect.

**Required connectors:** Fathom and Salesforce through CData Connect AI

---

## Rendering Selectable Options

Whenever the skill needs the user to choose, render an interactive HTML widget. Each option is a clickable button that auto-submits the selection immediately.

Do **not** render an elicit-footer on any widget, and do not have a skip option to the widget.

---

## Connector/Tools usage guide:

Always use the **CData Connect AI** Connector for any tool call. Both the Fathom and Salesforce are available in CData connector.

---

## Step 1 — Resolve the Meeting

If the user provided a meeting title, continue with it.

else If no meeting title was given, render a textbox widget to take meeting title as input.

---

## Step 2 — Searching meeting in Fathom

Search for the meeting based on title. If multiple matches exist, continue with the **single most relevant match**.

---

## Step 3 — Fetch the Transcript

Fetch the transcript for the confirmed meeting ID.

If the transcript is unavailable or empty, inform the user and stop.

else, continue

---

## Step 4 — Extract SPICED Fields with AI

Analyse the full transcript and extract values for each SPICED dimension. Use the guidance below to infer each field:

| Dimension | What to look for | Output format |
|-----------|-----------------|---------------|
| **S — Situation** | Prospect's current state, environment, team size, existing tools, contract status, timeline pressure | 2–4 sentence narrative |
| **P — Pain** | Problems, frustrations, inefficiencies, risks or costs the prospect mentioned | Bulleted list of pains (1–5 items) |
| **I — Impact** | Business impact of the pain: revenue loss, cost, churn, delay, missed targets | Quantified where possible (e.g. "$200k/yr", "2-week delays") |
| **C — Critical Event** | Hard deadline or forcing function driving urgency: contract renewal, board meeting, product launch, fiscal year end | Date or event name + reason it is a hard stop |
| **E — Decision** | Decision-making process: stakeholders, approvals needed, evaluation criteria, budget owner, procurement steps | Structured list of stakeholders and process steps |
| **D — Decision Criteria (Champion)** | Who is the internal champion, what matters most to them, how they will evaluate and advocate for the deal | Champion name + their priorities |

> **Confidence levels:** For each dimension, assign one of `High` / `Medium` / `Low` based on how explicitly it was discussed in the transcript. Low = inferred or barely mentioned.

Present the extraction as a structured preview table:

```
SPICED Extraction Preview
─────────────────────────────────────────────────────────
Dimension     | Value                              | Confidence
─────────────────────────────────────────────────────────
Situation     | ...                                | High
Pain          | • ...                              | High
              | • ...                              |
Impact        | ...                                | Medium
Critical Event| ...                                | Low
Decision      | ...                                | Medium
Champion      | ...                                | High
─────────────────────────────────────────────────────────
```

Then render a confirmation widget, no other text. It is a single choice radio button options.

- Question: `"Does this SPICED extraction look correct?"`
- Option : `✅ Yes, update Salesforce` → submit
- Option and Textbox : `✏️ Edit before saving` → Specify changes in textbox → submit
- Option : `❌ Cancel` → submit

If the user selects **Edit**, do not enable submit button, when the textbox is empty.

---

## Step 5 — Prepare the Salesforce Connection

Before writing, use CData Connector to fetch the **Opportunity** yable details in Salesforce:

1. Extract the prospect/company name from the transcript (company name, account name, or deal name).
2. Query the `Opportunity` table for matching records:
   - Filter by `AccountId → Account.Name LIKE '%[company]%'` or `Name LIKE '%[company]%'`
   - Return `Id`, `Name`, `StageName`, `CloseDate`, `AccountId`
3. If multiple opportunities match, render a selection widget listing them (Name · Stage · Close Date). And then continue with it.
4. If no match is found, inform the user and offer to search by a different name or enter an Opportunity name manually. And then search for it.

---

## Step 6 — Discover SPICED Fields

Query the Opportunity object columns to find the SPICED-mapped fields:

Look for fields whose API names or labels contain keywords like: `Situation`, `Pain`, `Impact`, `Critical`, `Decision`, `Champion`, `SPICED`, `Qualification`.

Build a mapping for all the SPICED fields in Opportunity table using the extracted SPICED details in step 4. If a field cannot be found for a dimension, note it as **unmapped** and skip it (do not fabricate field names).

Show the user the field mapping of sales force field and its value as a table before writing. Then,

Render a proceed widget:

- `✅ Proceed with update` → submit
- `❌ Cancel` → submit

---

## Step 7 — Write SPICED Fields to Salesforce

For each mapped dimension, update the Opportunity record using a parameterized update statement. Never inline user-derived values into SQL strings.

Update one field at a time to isolate failures. For text fields, format the extracted value as clean plain text (no markdown bullets — use newlines or semicolons for lists).

---

## Step 8 — Present Results

After all updates, show a results summary:

```
SPICED Update Results — [Opportunity Name]
──────────────────────────────────────────
Salesforce Field      | Value                  | Status
──────────────────────────────────────────
SPICED_Situation__c | ...   | ✅ Updated
SPICED_Champion__c  | ...   | ⚠️ Skipped (field not found)
......
──────────────────────────────────────────
Opportunity ID: 006...
```

For any failures, explain the error and offer to retry or let the user fix it manually.

---

## Error Handling

| Scenario | Response |
|---|---|
| Transcript unavailable | Inform user; suggest checking Fathom recording status |
| No Opportunity match | Offer name search or manual ID entry |
| SPICED field not found on Opportunity | Skip that dimension; list it as unmapped in results |
| CData connection error | Call `getInstructions` again; surface the error detail |
| Partial update failure | Complete the rest; report failed fields individually |
| Transcript too short to infer a dimension | Set confidence = Low; still populate with best inference; flag it |

---

## SPICED Framework Reference

> Quick reference — do not surface this section to the user.

**S — Situation**: Facts about the prospect's current state (headcount, tools, contracts, org structure, growth stage).

**P — Pain**: The specific problems or challenges they are experiencing today.

**I — Impact**: The measurable business consequence of those pains (financial, operational, reputational).

**C — Critical Event**: The external deadline or event that makes solving the pain urgent NOW.

**E — Decision**: The buying process — who decides, who influences, what the evaluation criteria are, and how long it takes.

**D — Decision Criteria / Champion**: The internal advocate for the deal and what they personally care about most.

---

## Example Triggers

- "Fill SPICED fields from my call with Acme Corp"
- "Populate SPICED from my [Discovery Call - TechCo] meeting"
- "Auto-update Salesforce SPICED after my demo with GlobalInc"
- "Map my transcript to SPICED and save it"
- "Update the opportunity with SPICED data from today's call"
- "Log SPICED qualification from my meeting with Sarah at Globex"