---
name: forecast-prep
description: >
  Generates a personalized pre-forecast pipeline briefing for Account Executives
  before weekly forecast calls. Use this skill whenever a user wants to: prepare
  for a forecast call, review pipeline health before a sales meeting, get a deal
  risk summary, identify stale or slipping opportunities, get CRM hygiene
  recommendations, or generate an AE forecast brief. Trigger for any phrases like
  "prep me for my forecast call", "what's my pipeline health", "show me risky
  deals", "CRM hygiene check", "forecast briefing", "pipeline review", "stale
  deals", "which deals are slipping", "get me ready for my forecast", or "forecast
  prep". Always use this skill when the user is an AE or sales manager who wants
  insight into their pipeline before any kind of review or forecast meeting.
---

# Pre-Forecast Pipeline Briefing Agent

A "Forecast Prep Copilot" that pulls live data from Salesforce and Fathom via
CData Connect AI, analyzes pipeline health and call signals, then generates a
concise, prioritized briefing before weekly forecast calls.

**Data sources:** Salesforce (catalog `Salesforce1`) and Fathom (catalog
`Fathom`), both accessed exclusively through CData Connect AI tools.

Always ground every finding in the retrieved data. Never invent deal values,
stages, customer actions, or forecast numbers.

---

## Step 1 — Pull Salesforce Pipeline Data

Use CData Connect AI to query the Salesforce catalog.

Discover and fetch all open (non-closed) opportunities for the AE's pipeline.

Key fields to retrieve per opportunity:
- Opportunity name and account name
- Stage, amount, close date
- Forecast category (Commit / Best Case / Pipeline / Omit)
- Last activity date
- Next step and next step date
- Stage last modified date (to compute days in stage)
- Close date change count or history
- Exec sponsor / key stakeholder fields
- Any MEDDPICC-related custom fields

Use date filters and `LIMIT` to avoid timeouts on large orgs.

---

## Step 2 — Pull Fathom Call Data

Use CData Connect AI to query the Fathom catalog.

Discover and fetch recent meeting and call data.

Key fields to retrieve:
- Meeting title and associated account or opportunity name
- Meeting date
- Transcript summary or key topics discussed
- Action items and next steps captured
- Sentiment signals (if available)
- Attendees

Focus on calls from the **last 30 days**. Match calls to opportunities from
Step 1 by account name or opportunity reference.

---

## Step 3 — Build Deal Intelligence Objects

Merge the Salesforce and Fathom data into one summary per open opportunity.
Use `null` for any field not present in the retrieved data.

```
opportunity_id, account, stage, amount, close_date,
days_in_stage, last_customer_activity_days,
next_step, next_step_date, meeting_count_14d,
close_date_changes, forecast_category,
exec_sponsor_identified, recent_call_summary, sentiment_signal
```

---

## Step 4 — Run Pipeline Health Analysis

Apply these rules to every deal. A deal may carry multiple flags.

### Staleness Flags
| Condition | Flag |
|-----------|------|
| `days_in_stage > 21` for Negotiate / Close stages | "Stage Stagnation" |
| `days_in_stage > 30` for earlier stages | "Stage Stagnation" |
| No customer activity in 14+ days | "Customer Silence" |
| `next_step` is null or empty | "Missing Next Step" |
| `next_step_date` is in the past | "Overdue Next Step" |

### Risk Flags
| Condition | Flag |
|-----------|------|
| Close date changed 2+ times | "Deal Slipping" |
| Forecast = Commit AND no exec sponsor on record | "Weak Commit — No Exec Sponsor" |
| Zero meetings in last 14 days AND stage is Proposal or Negotiate | "Engagement Drop" |
| Negative sentiment in recent Fathom call | "Negative Sentiment Signal" |

### Coverage
- Sum pipeline totals by forecast category (Commit, Best Case, Pipeline)
- If quota is available in Salesforce, flag if Commit < quota × 0.8

---

## Step 5 — Run CRM Hygiene Analysis

Identify missing, stale, or inconsistent fields. For each issue, produce:

```
deal: <opportunity name>
field: <field name>
current_value: <existing value>
suggested_value: <recommended value>
reason: <one sentence grounded in the data>
```

Common checks:
- Missing next step or next step date
- Close date in the past or within 7 days with no recent activity
- Close date slipped 2+ times
- Forecast category = Commit but stage is Discovery or Qualification
- No meetings in last 30 days on deals > $50K
- Empty MEDDPICC fields on Commit deals

---

## Step 6 — Generate the Forecast Briefing Document

Using the deal summaries, risk flags, and hygiene findings, generate the
briefing directly. Use the docx skill for creating the output brief document as per the format in `references/output-template.md`.

Cover:
- Pipeline coverage summary (totals by forecast category)
- Top 3–5 high-risk deals — name each, cite the exact signals
- CRM hygiene gaps to fix before the call
- Prioritized recommended actions for the AE
- Likely questions the manager will raise

Rules:
- Only use data retrieved in Steps 1–2. No invented figures.
- Name the deal and cite the exact data point behind every flag.
- Order by revenue impact and risk severity.

Use the docx skill to create the briefing document.

---

## Step 7 — Present for Human Review

Show the proposed changes in a table listing each proposed change with the following columns: Deal, Field, Current Value, Suggested Value, Reason.

Render a HTML widget of a multi-select checkbox list of proposed changes.
Then ask to select the changes to be updated in salesforce.

**Do NOT auto-update Salesforce.** Only proceed with updates of selected changes.

---

## Step 8 — Apply Approved CRM Updates

Only after explicit per-item AE approval:
- Use CData Connect AI to write back to Salesforce
- Log: timestamp, approved_by, field, previous_value, new_value, reason

---

## Key Constraints

- **No hallucination** — every flag must cite a specific field and value from the data
- **Explainability** — every recommendation includes a reason tied to evidence
- **Human control** — the AE can reject any update; never auto-apply
- **Data gaps** — if a field is missing, state "unable to assess — data not available"
  rather than guessing
