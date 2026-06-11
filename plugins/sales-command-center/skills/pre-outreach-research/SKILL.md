---
name: pre-outreach-research
description: "Creates comprehensive account intelligence profiles for B2B sales preparation. Use when researching a company before outreach, discovery calls, or QBRs. Trigger for: 'research this account', 'prep me for a call with [company]', 'build an account profile', 'what do I need to know about [company]', 'account intel', 'company research', or any time a user wants background on a prospect. Also trigger for deal preparation and account planning. Produces a 9-section intelligence report — account snapshot, market & tech intelligence, insights & buying signals, stakeholders, qualification & value fit, outreach strategy, competitive context, next best actions, and sources — delivered as a styled .docx. After delivery, asks the user for revisions, applies them, then drafts a professional email with the brief attached."
---

# Account Research

This skill turns "what do I know about this company?" into a 9-section, sales-ready intelligence brief delivered as a polished Word document.

## Division of responsibilities

This skill owns the **content** of the brief — what every section contains, where the data comes from, how to reason about pain and signals, what good looks like.

The **`docx` skill** owns the document — page layout, fonts, headings, tables, lists, validation. When it's time to produce the .docx, hand the filled content over to the `docx` skill and let it design the file. Don't write your own renderer, don't reach for `python-docx`, don't try to format the docx yourself. The `docx` skill is the single point of truth for how Word documents are made in this organization.

## What you produce

A `.docx` file whose content corresponds — section by section, field by field — to the canonical template at `assets/account_brief_template.md`. The template is the **content contract**: every heading, every bullet, every table cell described there must appear in the final document, with real values you researched. Use a dash `—` only when you genuinely could not find the data, and always note it in Section 9 (Sources) so the reader knows it was searched, not skipped.

**Writing style:** Write in concise phrases and short declarative sentences throughout. No paragraph prose anywhere in the document — if a point takes more than one sentence to make, express it as two bullets instead.

## Workflow

0. **Check CData Connector.** check if the CData Connector is connector and go through available tools.
1. **Read the content contract.** Open `assets/account_brief_template.md` and treat it as your filing checklist — these 9 sections, in this order, with these fields, are what the final document must contain.
2. **Sequence your research.** Follow `references/research_process.md` — firmographics → signals → org mapping → synthesis. Doing it out of order produces generic-sounding output because you'll synthesize before you have specifics to anchor on.
3. **Use the right tool for each section.** `references/tool_extraction_guide.md` maps every section to its primary tool. All structured data (Apollo, Exa, Salesforce) is accessed via CData Connect AI using the queryData tool — not direct connectors. Only Apify stays as a direct MCP call. When a tool is unavailable, fall back and lower the section's confidence rating.
4. **Pull CRM context when relevant.** If the account might already be in Salesforce, run `references/crm_lookup.md` before drafting Sections 4, 7, and 8 — an open opportunity or recent activity completely changes what "next best action" means.
5. **Hand the content to the `docx` skill.** Once every section is filled, invoke the `docx` skill and have it create the document. Pass it the content you produced — section titles, field values, tables, bullet lists — and let it own the layout, headings, table formatting, page setup, and validation. The `docx` skill should produce a single polished `.docx` file that contains every section from `assets/account_brief_template.md` populated with your researched values.
6. **Hand the file to the user.** Save it to the outputs folder and present it with `present_files`. Don't paste the full brief into chat — the docx is the deliverable.
7. **Ask for revisions.** After presenting the document, immediately ask the user: *"Would you like any changes to this document before I send it?"* Wait for their response.
   - If they say **yes** or describe changes: apply all requested edits to the `.docx` (re-invoke the `docx` skill if structural changes are needed, otherwise patch inline), save the updated file, re-present it, and ask again if further changes are needed. Repeat until they are satisfied.
   - If they say **no** or are happy: proceed to the next step.
8. **Send the email.** Once the document is approved, use the Office365 tool for sharing email with the account brief attached.
The email should:
   - Reference the company name and purpose of the brief
   - Be concise and ready to send — not a template with placeholders
   - Include a subject line that references the company name
   - Attach the account brief document in the mail
   - Mention the attached account brief naturally in the body

Show this email in a proper visualization widget and ask for any changes in the email.
 - if any changes mentioned, apply those changes and present the email for review again
 - if no changes then, ask the user for the recipient's email(s) for sending the email.

Then send the email to the mentioned recepient using Office365 through the CData connector.


## Quality bar

A brief is "good" when a rep can read it once and walk into a discovery call without doing more research. Specifically:

- Section 3 (Insights & Buying Signals) contains reasoned pain — trigger → problem → implication — not restated firmographics.
- Section 3 (Insights & Buying Signals) lists trigger events with implications, not vague growth signals.
- Section 4 (Stakeholders) names 3+ real people with titles and tenure, not function placeholders.
- Section 6 (Outreach Strategy) has three distinct angles — trigger / pain / social proof — that don't repeat each other.
- Section 9 (Sources) lists every URL touched, so the rep can verify any claim in 5 seconds.
- Every field value is scannable in under 5 seconds. Prefer phrases and fragments over full sentences. Reason fully, write the conclusion only.

The Research Quality Guidelines in `references/tool_extraction_guide.md` show the bad-vs-good pattern for each of these.

## Confidence rating

After the brief is built, rate it High / Medium / Low based on data coverage and put it in the document header. The exact criteria are in `references/tool_extraction_guide.md` under "Confidence Rating".

## Cross-references

- Pair with `prospect-research` when zooming into individual stakeholders.
- Feed the buying signals + value hypothesis into cold outreach drafts.
- Use the pain hypothesis and conversation hooks to build call prep agendas.
- Re-run after discovery calls — new intel updates Sections 3, 4, and 8.
