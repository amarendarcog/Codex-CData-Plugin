# Connector Guide

What to fetch from each data source during Phase 2 — Research. **Use all connectors listed below — no exceptions.** Read this before making tool calls.

---

## Office365

Access Office365 through CData Connect AI.

Search the calendar for the meeting title keyword to retrieve the event details — attendees, start time, organiser, and description. Search recent email threads with the company or attendees to surface prior context, commitments made, and the overall tone of the relationship.

To send the team email in Phase 5, use the CData Office365 connector. First check what send procedures are available, then use the appropriate one to send the email with the brief attached.

---

## Apollo.io

Apollo.io is a **sales intelligence and enrichment platform** — NOT a CRM. Do not use it for deal history or rep notes.

Enrich the company by domain to get firmographics, estimated revenue, employee count, tech stack (160+ signals), subsidiary map, and departmental headcounts.

For each known attendee, enrich by name and company to get their title, bio, email, and LinkedIn URL.

Search your team's existing contacts at the company to determine if this is a net-new account or one your team has previously engaged. Zero results means net-new — it does NOT mean the company doesn't exist in Apollo.

Always enrich the company first, then enrich each attendee, then check team contacts.

---

## CData / Salesforce CRM

Pull the account record to get industry, revenue, and last activity date.

Pull all contacts linked to the account — names, titles, and last activity dates.

Pull all open opportunities — deal name, stage, amount, close date, probability, and the Description field. The Description field often contains rep notes, pursuit leads, and assignee info written by the AE — always include it.

Pull recent tasks and activities to understand the last touchpoints and any outstanding commitments.

Always query Account, Contact, and Opportunity independently. Zero contacts does not mean zero CRM history — there may still be open opportunities.

---

## Fathom

Fathom stores recorded meeting transcripts. Use it to surface past conversation history with the company before generating the brief.

Search for previous meetings using the company name and/or attendee names as keywords. Fetch a **maximum of 2 meetings** — prioritise the most recent ones.

For each matched meeting:
- Retrieve the full transcript using CData Connect AI to access the Fathom connector
- Note the meeting title, date, and participants
- Extract: pain points raised, commitments made, objections voiced, next steps agreed, product areas discussed, sentiment signals, and any open questions left unresolved

Incorporate this transcript intelligence throughout the brief:
- **Section 4 (Stakeholder Intelligence)** — surface what each person said, their concerns, and communication style
- **Section 3 (Meeting Context)** — use as the primary source of prior interaction context if CData has no activities
- **Section 7 (Agenda)** — tailor discovery questions based on prior objections and stated priorities
- **Section 8 (Risks & Next Steps)** — pre-load with objections raised in past calls
- **Section 9 (AI Brief Layer)** — reference specific quotes or themes from transcripts to raise confidence score

Write conclusions only — extract the implication for the rep, not raw transcript summaries.

If no Fathom meetings are found for this company, note `[No prior Fathom meeting transcripts found]` and proceed with other sources.

---

## Exa

Search for recent news and press coverage about the company. Look for funding announcements, investor activity, and growth signals. Research strategic priorities such as expansion, AI adoption, product launches, and hiring trends. Look into their competitive landscape — who they compete with and how they're positioned. For each known attendee, search for their background, recent activity, and any published content.

Prefer results from the last 90 days. If a specific article looks highly relevant, fetch its full content for deeper context.

Write conclusions only — surface the implication for the rep, not the article summary.

---

## Apify

Access Apify through CData Connect AI to scrape LinkedIn profiles for each known attendee and extract their current title, career history, tenure, and recent public activity.

Use CData Connect AI to scrape the company's LinkedIn page for headcount, employee growth signals, company description, and recent posts.

If a LinkedIn URL is unknown, search the web to find it first, then scrape via CData.

---

## Data Quality Rules

1. **Always cite sources** — every claim should trace to a data source in Section 10 (Sources)
2. **Flag uncertainty** — use `*[inferred]` or `*[unconfirmed]` for hypotheses
3. **Confidence scoring** — the AI Brief Layer confidence score reflects actual data coverage
4. **Sparse fields** — if data is unavailable, write: `[Not available — consider asking during the call]`
5. **Prioritize recency** — prefer data from the last 90 days for news and interactions
6. **Writing style** — write conclusions only; no paragraph summaries in the document

---

## Known Gotchas

### Apollo.io is NOT a CRM
Apollo.io is the source of truth for firmographics, tech stack, and person enrichment — not for deal history, opportunity stages, or rep notes. Use CData → Salesforce for all CRM context.

### Searching team contacts ≠ searching Apollo's full database
The team contacts search only returns people your team has explicitly added to Apollo. Zero results means net-new — not that the company doesn't exist. Always enrich the company separately.

### CRM opportunity Description field often contains key rep context
The Salesforce Opportunity Description field frequently contains pursuit lead, solution lead, and assignee info. Always pull it.

### 0 Contacts in CRM ≠ net-new account
An account may have open opportunities but no linked contacts. Always query Account, Contact, and Opportunity independently.