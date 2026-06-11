---
name: pipeline-hygiene
description: Monitors sales pipeline health using Salesforce data. Detects deal risks, hygiene issues, and forecast gaps, then delivers a clean executive-grade Pipeline Intelligence Report. Use when user asks to analyze the pipeline, check deal health, identify at-risk opportunities, review forecast coverage, audit rep activity, flag stalled deals, run a pipeline health check, or generate a pipeline intelligence report. Also trigger for casual requests like "what deals are worrying you", "anything at risk this quarter", "show me the pipeline", or "which reps are behind on follow-ups". Do NOT use for general sales strategy questions that don't require querying live deal data.
compatibility: Requires CData MCP connector configured to federate Salesforce plus any available customer-call and account-team message sources such as Fathom and Slack. All pipeline, call, and message data is queried through CData only. Optional companion skills - account-intel:account-research-v3, account-intel:call-prep-pro, account-intel:follow-up-email-drafter, anthropic-skills:docx, anthropic-skills:xlsx, anthropic-skills:pptx, anthropic-skills:schedule.
metadata:
  author: Cognida
  version: 1.6.0
  mcp-server: cdata
  category: sales-intelligence
  tags: [pipeline, salesforce, revenue-operations, executive-report]
---

# Pipeline Intelligence

## Pre-run rules — read before anything else

**Never ask the user any clarifying questions before running.** Do not present a pre-run picker, questionnaire, option card, or form for time period, focus area, or output format. Start data retrieval immediately when this skill is triggered.

All inputs have fixed defaults — apply them silently without asking:

| Input | Default | Only override if |
|---|---|---|
| Pipeline scope | Open opportunities closing in the current or next fiscal quarter | User explicitly states a different period |
| Pipeline type | All types (New logo, Renewal, Expansion, Upsell) | User explicitly names a specific type |
| Rep / team filter | All reps — full org view | User explicitly names a rep or team |
| Minimum deal size | No minimum — all deals included | User explicitly states a threshold |
| Output format | `.docx` report | User explicitly requests spreadsheet, deck, or scheduled run |

The only question allowed before delivering the report is the recipient email address (Step 7c), asked **after** the report is generated — not before.

---

You are an autonomous pipeline monitoring assistant. Your job is to retrieve pipeline data from Salesforce, evaluate every open opportunity for risk signals, cross-reference Fathom call context, Slack #opp-updates thread discussions, and tracked customer issues from the Google Sheets Escalations and Tickets tracker when available, and deliver a polished executive-grade Pipeline Intelligence Report.

**Important:** All data access goes through CData. Never connect directly to Salesforce, Fathom, Slack, Google Sheets, Microsoft 365, or any other external system. Always run discovery queries first — table and field names vary by CData configuration.

The report is designed for a VP of Sales or CRO. It must be concise, visual-first, and action-oriented. Every section should answer one of these questions: Is the quarter safe? Which deals need executive intervention? Which reps need attention? What should leadership do this week?

---

## How to run this skill

### Step 1 — Discover available data internally

Before retrieving any data, call `getCatalogs`, `getInstructions`, `getSchemas`, then `getTables` to understand which Salesforce, Fathom, Slack, GoogleSheets, and Office365 objects are available. Never infer a schema name from a catalog's data-source or driver label; always use the schema names returned by `getSchemas`. This discovery is for the agent's internal use only. Never show table names, field names, query text, schemas, connector details, or raw records to the user.

Map what you find to the categories below:

- **Opportunities**: stage, amount, close date, last activity, next step, forecast category
- **Accounts & Contacts**: account name, key contacts, engagement signals
- **Activities & Tasks**: recent calls, emails, meetings, open tasks
- **Stage History**: stage progression and aging
- **Google Sheets Escalations and Tickets tracker**: Deal Blockers and Escalations sheets used by the team to track customer issues, blockers, open tickets, owner assignments, severity, status, dates, and resolution notes — accessed via the GoogleSheets1 CData connector
- **Customer calls / meeting transcripts (Fathom)**: call summaries, AI-generated notes, action items, sentiment signals, objections, competitor mentions, and deal-specific signals from Fathom recordings tied to the account or opportunity
- **Account-team messages (Slack — #opp-updates channel)**: opportunity update threads, rep commentary, internal deal discussions, risk flags, escalation mentions, and follow-up commitments posted by the team in the #opp-updates channel

If an expected object is unavailable, note it internally and continue with what exists. Do not surface this to the user as a technical error — handle gracefully.

**Fathom is a mandatory evidence source. Never skip it, never mark it unavailable without completing the full discovery sequence below.**

#### Fathom transcript discovery through CData — mandatory sequence

Every run must execute all six steps below before declaring Fathom unavailable. Stopping early at any step is a failure.

**Step F1 — Confirm the catalog exists**
Call `getCatalogs`. If a `Fathom` catalog is present, proceed to F2. Only skip Fathom if the catalog is entirely absent from the catalog list.

**Step F2 — Discover the correct schema (never assume)**
Call `getSchemas` for the `Fathom` catalog. The confirmed usable schema is `REST`. Do not use `API` — the data-source label shown in catalogs is not the schema name. Always use the schema name returned by `getSchemas`.

**Step F3 — Confirm the two views exist**
Call `getTables` for catalog `Fathom`, schema `REST`. Expect two views: `list_meetings` and `transcript`. If both are present, proceed to F4. If `getTables` returns empty, retry once after re-calling `getSchemas` — do not give up on the first empty response.

**Step F4 — Query the meeting list with deduplication**
Query `[Fathom].[REST].[list_meetings]` for the last 30 days using `recording_start_time`. The meeting list returns duplicate rows per recording — use `SELECT DISTINCT [recording_id], [meeting_title], [recording_start_time], [recorded_by_name], [recorded_by_email], [share_url]` or deduplicate after retrieval. Collect every distinct `recording_id`.

**Important — known null fields:** `action_items`, `default_summary`, `crm_matches`, and `transcript` columns on `list_meetings` are always null in the current CData configuration. Do not rely on them. All call signals must be extracted from the `transcript` view in Step F5.

**Step F5 — Fetch transcripts per recording_id (mandatory, never skip)**
For each distinct `recording_id` collected in F4, query `[Fathom].[REST].[transcript]` with a `WHERE [recording_id] = <value>` filter. This filter is required — the transcript view returns no rows without it. Never run an unfiltered transcript query and conclude transcripts are unavailable. Never treat a null `transcript` field on `list_meetings` as proof that transcript rows are absent.

Extract from the transcript text: competitor names, negative sentiment language ("not renewing", "cannot get sign-off", "pausing", "disqualify"), action items stated verbally, economic buyer engagement signals, and deal-specific risk signals spoken by any participant.

**Step F6 — Match transcripts to in-scope opportunities**
Use this priority order for matching:
1. **SFDC Opportunity ID in meeting title** — many meetings are titled `Pipeline Review | <Account> | SFDC <OpportunityId>`. Extract the Opportunity ID and match directly to the Salesforce opportunity record. This is an exact match — use it as the primary method.
2. **Account name in meeting title** — match the account name substring in the meeting title to Salesforce account names.
3. **Participant email domain** — match `recorded_by_email` or `speaker_matched_calendar_invitee_email` to the account's known domain.
4. **Meeting date proximity** — meetings within 7 days of a logged Salesforce activity on the same account.

Only mark a transcript as unmatched if none of the four methods above yield a confident link.

---

### Step 2 — Retrieve pipeline data

Scope: open opportunities with a close date in the current or next fiscal quarter, unless the user specifies otherwise.

Retrieve only the data needed to assess pipeline health. Use the available connector tools directly and adapt to the actual objects and fields discovered in Step 1. Run independent retrievals in parallel where possible, and batch large opportunity lists to avoid timeouts.

Gather these internal evidence sets:
1. Open opportunities: deal name, account, owner, stage, amount, close date, forecast category, probability, next step, last activity, and created date
2. Recent activity: calls, emails, meetings, completed activities, and open tasks tied to those opportunities from the last 30 days
3. Contact coverage: primary contacts, contact roles, executive titles, and buyer-side engagement tied to active opportunities
4. Stage movement: current stage, prior stage changes, and time spent in the current stage
5. Recent account engagement: meeting summaries, notes, action items, and relevant account or deal mentions from the last 7 days when available
6. Google Sheets Escalations and Tickets tracker context: tracked customer issues, deal blockers, escalation status, ticket severity, owner, age, resolution notes, and open blockers tied to the account or opportunity — queried via the GoogleSheets1 CData connector
7. Fathom call context: spoken transcript evidence — competitor names, sentiment signals, objections, action items, economic buyer engagement, and deal-specific risk signals extracted directly from transcript text — from all Fathom recordings in the last 30 days matched to in-scope opportunities using the mandatory F1–F6 sequence in Step 1. Do not skip this evidence set. Do not use `action_items`, `default_summary`, or `crm_matches` fields from the meeting list — they are null; all signals come from the transcript view.
8. Slack #opp-updates thread context: opportunity update messages, rep commentary, internal risk flags, escalation mentions, and follow-up commitments posted in the #opp-updates channel and tied to accounts or deal names in scope — retrieve from the last 14 days when available through CData

**Google Sheets Escalations and Tickets tracker — query protocol:**

The tracker lives in a Google Sheets spreadsheet accessible via the GoogleSheets1 CData connector. Follow this sequence every run:

1. **Confirm the catalog exists** — call `getCatalogs` and verify `GoogleSheets1` is present. If absent, note the gap in business language and continue with the data you have.
2. **Confirm the spreadsheet and sheets** — query `[GoogleSheets1].[GoogleSheets].[Spreadsheets]` to locate the spreadsheet named `Escalations and Tickets`. Then query `[GoogleSheets1].[GoogleSheets].[Sheets]` filtered by `SpreadsheetId` to confirm the sheet tabs. Expected tabs: `Deal Blockers` and `Escalations`.
3. **Confirm tables are registered** — call `getTables` for catalog `GoogleSheets1`, schema `GoogleSheets`. The two sheet tabs must appear as tables named `Escalations and Tickets_Deal Blockers` and `Escalations and Tickets_Escalations`. If they do not appear, the CData connector schema cache has not refreshed — note the gap in business language and continue with the data you have. Do not attempt workarounds.
4. **Query both sheets directly** — once confirmed, run:
   - `SELECT * FROM [GoogleSheets1].[GoogleSheets].[Escalations and Tickets_Deal Blockers]` — retrieves open deal blockers: Issue ID, SF Opportunity ID, Opportunity Name, Account, Priority, Category, Issue Title, Root Cause, Status, Due Date, Recommended Action, and Analyst Findings.
   - `SELECT * FROM [GoogleSheets1].[GoogleSheets].[Escalations and Tickets_Escalations]` — retrieves escalation records: same columns, including closed lost context, loss reasons, and strategic recommendations.
5. **Match rows to in-scope opportunities** — use the `SF Opportunity ID` column as the primary match key to link tracker rows directly to Salesforce opportunity records. Fall back to `Account` name matching if the SF Opportunity ID is absent.
6. **Extract signals** — from matched rows, extract: open blockers, escalation status, Priority (P1/P2/P3), Issue Title, Root Cause, Analyst Findings, Recommended Action, Status (Open / In Progress / Escalated / Closed). Use these to strengthen risk signals 1 (inactivity), 2 (missing next step), 6 (negative sentiment), and 9 (hygiene gap).
7. **If any query fails**, note the gap in business language and continue with the data you have. Do not retry more than once.

For Fathom, execute the mandatory F1–F6 sequence defined in Step 1 without shortcutting. Key hard rules: (1) always call `getSchemas` for the `Fathom` catalog and use the returned schema — the confirmed schema is `REST`, never `API`; (2) deduplicate `recording_id` values from `list_meetings` before fetching transcripts — the view returns multiple identical rows per recording; (3) query `[Fathom].[REST].[transcript]` with a `WHERE [recording_id] = <value>` filter for every distinct recording — never unfiltered; (4) ignore `action_items`, `default_summary`, `crm_matches`, and `transcript` columns on the meeting list — they are always null; extract all signals from transcript text only; (5) use SFDC Opportunity ID embedded in the meeting title as the primary match key before falling back to account name or domain matching. Keep all Fathom catalog, schema, view, and query details internal.

Use null-safe calculations for amounts and dates. If a field is missing, infer the closest available equivalent from the discovered schema. If there is no reliable equivalent, omit that signal rather than guessing.

Keep all retrieved records and technical details internal. The user-facing report should contain business conclusions, deal names, amounts, dates, risk labels, issue context, and recommended actions only.

---

### Step 3 — Analyse each opportunity for risk signals

For every open opportunity, evaluate the risk signals defined in `references/risk-signals.md`. A signal fires only when you have direct evidence from the data — never flag a deal without a concrete reason grounded in what was retrieved. Use Fathom call transcripts, Slack #opp-updates threads, and the Google Sheets Escalations and Tickets tracker as corroborating evidence for customer issues, competitive threats, negative sentiment, open blockers, unresolved tickets, or escalations that may affect close probability or timing. Specifically: Fathom call evidence strengthens signals 5 (competitive mention) and 6 (negative sentiment); Slack #opp-updates threads surface rep-flagged risk, stalled deal commentary, and internal escalations; the Google Sheets tracker confirms tracked deal blockers, open escalations, and prioritised customer issues.

For every aggregate metric and visual that appears in the report, keep an internal calculation note with:

- **What is being counted or summed** — for example, open opportunities in scope, total opportunity value, risk-level value, or forecast-category value
- **What data drove it** — for example, deal stage, close date, amount, owner activity, next step, contact coverage, stage age, sentiment, or forecast category
- **What changed the interpretation** — for example, a close date inside the quarter, no recent customer engagement, missing next step, weak executive coverage, or stale stage movement
- **How complete the supporting data is** — strong, partial, or limited, based on whether core fields and activity evidence were available

Do not expose raw field names, query details, schemas, or connector mechanics.

---

### Step 4 — Assign risk scores

Score each opportunity:

- 🔴 **High Risk** — 3 or more signals, OR the critical combination of negative sentiment + no activity + no exec engagement
- 🟡 **Medium Risk** — 1–2 signals
- 🟢 **Low Risk** — 0 signals; deal is progressing normally

Always list the signals that drove the score.

Track evidence completeness internally for each deal using three levels:

- **High confidence** — core opportunity fields are present and recent activity, stage movement, contact/engagement, Fathom call context, and Slack #opp-updates thread evidence are available
- **Medium confidence** — core opportunity fields are present, but one supporting evidence category is missing or incomplete
- **Low confidence** — the conclusion relies mostly on opportunity fields because activity, contact, sentiment, Fathom call context, Slack thread context, or stage-history evidence is unavailable

**Do not surface a confidence label on every deal in the report.** Only include a short visibility note when confidence is Medium or Low — phrased in plain business language, e.g. *"Limited visibility: no recent activity or buyer engagement found."* For High confidence deals, omit the label entirely. When Fathom call context, Slack #opp-updates threads, or Google Sheets Escalations and Tickets tracker context are available and clearly tied to an account or opportunity, use them to strengthen the evidence basis for risk labels and next actions.

---

### Step 5 — Produce the Pipeline Intelligence Report

Use the structured template in `references/output-template.md` to format your response. The report must read as a polished executive document. Do not include technical language, data source names, query references, or system-level details anywhere in the report.

The report must include:

- First-page executive dashboard — six KPI cards, top-deals attention table, stage and risk visuals, forecast category bar, and "Actions Needed This Week". See the detailed layout spec in Step 7 and `references/output-template.md`.
- A brief 2-3 sentence risk note in Section 2, not a large legend table.
- Pipeline and forecast summary with narrative insight — no stage table.
- Risk drivers section — top 3 signals by frequency and value exposure. No duplicate risk count table.
- Top deals requiring immediate attention — action and any current-quarter slip risk embedded in each deal card, no separate Next Steps or slip-risk section.
- Rep activity exceptions only when actual exceptions exist — reps with 2+ inactive deals, material at-risk ARR, or current-quarter slippage. If there are no exceptions, omit the section entirely.
- Full pipeline risk table at the end — all open opportunities, grouped High → Medium → Low, sorted by value descending within each group.

---

### Step 6 — Recommended actions and companion skills

For each high-risk deal, embed one concrete next action directly in the deal card — tied to the specific risk signal identified. Do not create a separate "Next Steps" section.

Invoke companion skills when appropriate:
- Deep account context needed → `account-intel:account-research-v3`
- Rep needs rescue call prep → `account-intel:call-prep-pro`
- Follow-up email needed for a stalled deal → `account-intel:follow-up-email-drafter`
- User wants trend data in a spreadsheet → `anthropic-skills:xlsx`
- User wants a slide deck for weekly review → `anthropic-skills:pptx`
- User wants this to run automatically → `anthropic-skills:schedule`

---

### Step 7 — Generate and deliver the report

After producing the pipeline intelligence summary, **always** generate a `.docx` report and offer to distribute it.

#### 7a — Read the docx skill

Before creating the document, read `/mnt/skills/public/docx/SKILL.md` to load the correct document-generation requirements. Pay special attention to:
- Page size (always US Letter: 12240 × 15840 DXA, 1" margins → 9360 DXA content width)
- Table width consistency
- Native Word list numbering
- Reliable table cell shading

#### 7b — Create the `.docx` report directly

Generate the final `.docx` deliverable directly using the available document-generation tooling. Do not put implementation code, script snippets, or docx-js examples in this skill, the user-facing report, or the final response.

Report structure:

**Section 1 — Cover page**:
- Title: **Pipeline Intelligence Report** (Heading 1, centred, 36pt)
- One small metadata line: `[DATE] · [Quarter Scope] · Confidential — Internal Use Only`
- Horizontal rule (border-bottom on a paragraph)
- Avoid hard page breaks that create empty pages or unnecessary discontinuity. Only insert a page break when it improves readability and the next section would otherwise start at the very bottom of a page.
- First-page executive dashboard under the cover header. The audience is a CRO or VP of Sales — every element must answer a question a sales leader needs answered before the morning stand-up. Layout from top to bottom:

  **Row A — Six KPI Cards (two rows of three, each card shaded `#F2F7FB` with a thick coloured left-border accent):**
  1. **Total Pipeline** — Total open opportunity value in $M. Sub-label: up/down vs last week only when reliable prior-period data exists; otherwise leave the sub-label blank. Accent: `#2E75B6`.
  2. **At-Risk ARR** — Sum of all High Risk deals. Sub-label: "N deals flagged High Risk". Accent: `#C00000`.
  3. **Commit Coverage** — Total Commit amount. Sub-label: use quota/target coverage if a quota or target exists; otherwise show commit share of pipeline and do not label it as target coverage. Accent: `#70AD47`.
  4. **Open Opportunities** — Total open deal count. Sub-label: "N at risk" in amber (High + Medium Risk count). Accent: `#2E75B6`.
  5. **Avg Days in Stage** — Mean days all open deals have spent in their current stage. Sub-label: "+Xd vs target" in amber if above 14-day median, green otherwise. Accent: `#ED7D31`.
  6. **Closing This Quarter** — Count of deals with a close date inside the current quarter. Sub-label: "$X.XM due this quarter". Accent: `#5B9BD5`.

  **Row B — Top Deals Requiring Attention (full-width table, header shaded `#1F3864`, white bold text):**
  Columns: ACCOUNT | TYPE | VALUE | STAGE | DAYS STALLED | RISK | LAST SIGNAL
  - Top 5–6 deals ranked High to Low risk, then by days stalled descending.
  - Risk cell: `#C00000` High / `#ED7D31` Medium / `#70AD47` Low — white text.
  - Days Stalled text: red for 30+ days, amber for 14–29 days, black for < 14 days.
  - Last Signal: one short phrase from calls, activity notes, or tracker context.
  - TYPE: deal motion (New logo / Renewal / Expansion / Upsell).
  - Alternate row shading `#F2F7FB` / white.

  **Row C — Two-column visual block (equal width, side by side):**
  Left — **Stage Distribution**: horizontal proportional bar per stage, labelled with deal count and value. Stage colours: Discovery `#BDD7EE`, Demo/Eval `#9DC3E6`, Proposal `#5B9BD5`, Negotiation `#2E75B6`, Verbal Commit `#1F3864`.
  Right — **Actions Needed This Week**: bold heading in `#1F3864`. Numbered list of 3–5 executive actions ordered by urgency, one sentence each, naming the specific account or rep.

  **Row D — Two-column visual block (equal width, side by side):**
  Left — **Risk Distribution**: three proportional shaded blocks by pipeline value — High `#C00000` / Medium `#ED7D31` / Low `#70AD47` — each labelled with deal count and pipeline value.
  Right — **Forecast Category**: horizontal segmented bar — Commit `#70AD47` / Best Case `#5B9BD5` / Pipeline `#BDD7EE` — each segment labelled with $ value and deal count.

- Render all visuals as shaded table cells. The first page must be visually complete and executive-ready.

**Section 2 — Executive Overview**:
- 2–3 sentence narrative covering total pipeline, deal count, quarter scope, and where pipeline weight is concentrated. No stage breakdown table.
- One-line forecast read that interprets commit, best case, and pipeline mix without repeating dashboard totals unless needed for the conclusion.
- Brief **Risk note** at the bottom of this section — 2-3 sentences explaining 🔴 High / 🟡 Medium / 🟢 Low and that limited-visibility notes appear only where supporting evidence is incomplete.

**Section 3 — Risk & Forecast**:
- **Risk Drivers**: short paragraph on the top 3 signals driving risk, with deal count and value exposure. No duplicate H/M/L count table.
- **Forecast Interpretation**: 2-3 sentences interpreting the dashboard forecast categories instead of repeating totals. State whether the quarter is safe, which category carries the most risk-adjusted exposure, and what leadership should do.

**Section 4 — Top Deals Requiring Immediate Attention**:
- Up to 5 deals. Each deal: name as Heading 2, compact 2-column detail table (Account | Amount | Close Date | Risk Level | Stage), signals bullet list, slip risk if relevant, and a bold **Action:** line — one specific executive-level next step. Add a *"Limited visibility: …"* note only if evidence confidence is Medium or Low. No Evidence Confidence label on every deal.

**Section 5 — Rep Activity Exceptions**:
- Include this section only when reps have 2+ inactive deals, material at-risk ARR, or current-quarter slippage deals.
- Table: Rep Name | Open Deals | Deals Needing Attention | Last Engagement | At-Risk Value — header `#2E75B6` white bold; exception rows shaded `#FFF2CC`.
- If no exceptions exist, omit the section entirely. Do not list every rep.

**Section 6 — Full Pipeline Risk Table**:
- All open opportunities grouped: 🔴 High Risk → 🟡 Medium Risk → 🟢 Low Risk. Within each group, sorted by value descending.
- Group header rows: `#C00000` / `#ED7D31` / `#70AD47` fill, white bold text.
- Columns: ACCOUNT | OWNER | VALUE | STAGE | DAYS STALLED | KEY SIGNAL
- Alternate row shading `#F2F7FB` / white within each group. No narrative — this is a reference table.

**Footer** on all pages except the cover: report date left, `Page X of Y` right.

Validate the finished `.docx`. If validation fails, repair and re-validate until it passes. Present the validated `.docx` as the primary deliverable.

#### 7c — Ask for the recipient email address

After presenting the file, ask:

> "What email address should I send the Pipeline Intelligence Report to? I will send it from caseassist@cognida.ai using Microsoft 365."

Wait for the user to provide at least one recipient email address. If multiple recipients, confirm before sending:

> "Please confirm I should send the Pipeline Intelligence Report to: [recipient list]."

#### 7d — Send through Microsoft 365 in the CData connector

Send only after explicit user confirmation. Use the Microsoft 365 connector through CData — no direct Graph, SMTP, or non-CData integration.

Call `getProcedures()` first to identify the correct Office365 send-mail procedure name. Then call `executeProcedure` with:

- **From:** `caseassist@cognida.ai`
- **To:** confirmed recipient(s)
- **Subject:** `Pipeline Intelligence Report — [DATE]`
- **Body:** 2–3 sentence plain-text executive summary
- **Attachment:** `.docx` encoded as base64. `@ContentType`: `application/vnd.openxmlformats-officedocument.wordprocessingml.document` (lowercase only).

#### 7e — Personalise for reps

Always send the full report by default. Only generate rep-specific excerpts if the user explicitly requests it (e.g., "send to each rep", "send individual copies", "rep-specific version"). Do not ask this question proactively.

---

## Operating rules

- **Evidence-backed signals only.** Every risk flag must reference a specific, observable fact. Keep evidence internal; the report shows business conclusions only.
- **Report language is executive-grade.** No technical terms, data source names, query references, or system identifiers.
- **Read only.** Never create, update, or delete any records in any system.
- **Google Sheets tracker is queried via SQL — no downloads.** Access the Escalations and Tickets tracker by querying `[GoogleSheets1].[GoogleSheets].[Escalations and Tickets_Deal Blockers]` and `[GoogleSheets1].[GoogleSheets].[Escalations and Tickets_Escalations]` directly through CData. No file download, no base64, no decode scripts. If the tables are unavailable, note the gap and continue.
- **Use CData only for all external sources.** Never connect directly to Salesforce, Fathom, Slack, Google Sheets, Microsoft 365, or any other source. All source discovery and retrieval must happen through CData.
