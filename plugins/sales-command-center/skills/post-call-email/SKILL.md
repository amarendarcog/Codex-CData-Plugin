---
name: post-call-email
description: "Drafts a professional AI-powered follow-up email from a meeting transcript. Use this skill whenever the user wants to write a follow-up email after a meeting, summarize a call and send next steps, create post-meeting action items as an email, or draft a recap email from a meeting transcript. Trigger for phrases like \"draft a follow-up for my meeting with X\", \"write a follow-up email from my call\", \"create a meeting recap email\", \"follow up on my [meeting title]\", \"send next steps from our meeting\", or \"post-meeting email\". Also trigger for casual requests like \"can you follow up on that call we had\" — this skill handles the full flow from meeting lookup to polished email."
---

# AI Follow-Up Email Drafter

Generates a professional follow-up email from a meeting transcript using Fathom. Handles meeting lookup, title matching, confirmation, and email composition.

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

Render a widget with following sections — no other text:

- Question: "Please Confirm/Edit the meeting title for follow-up email."
- An editable textbox with the meeting title autofilled.
- A continue button. 

Then say to user to either mention the meeting title above or in the chat input.

---

## Step 3 — Fetch Transcript

Once a meeting is confirmed, fetch its full transcript using fathom. If the transcript is unavailable or empty, inform the user and stop.

---

## Step 4 — Present the Follow-Up Email

Analyze the transcript thoroughly and produce a complete, professional follow-up email with these sections:

### Email Structure

**Subject:** Follow-Up: [Meeting Title] – [Date]

---

**Hi Team,**

**Opening line** — A warm, brief sentence referencing the meeting.

---

**Attendees:** [List all attendees identified from the transcript, comma-separated. If roles or companies are known, include them in parentheses, e.g. "Jane Smith (Acme Corp), John Doe (Host)". If names are unavailable, use "See participant list".]

---

**Meeting Summary**
A 2–4 sentence overview of what the meeting was about and the main outcome or decision reached.

---

**Key Discussion Points**
A concise bulleted list (3–6 items) of the most important topics discussed.

---

**Agreed Next Steps**
A numbered list of concrete actions agreed upon. Include owner (if known) and deadline (if mentioned).

---

**Mutual Action Plan**

| Owner | Action | Due Date |
|-------|--------|----------|
| [Name] | [Task] | [Date or TBD] |

---

**Closing line** — Forward-looking and positive.

**[Your name]**


- render the above email to the user in a proper widget without any buttons

---

Step 5 - Ask for changes to email

1. Render an interactive widget using show_widget with:
   - A textarea for "Requested Changes" (placeholder: "Describe any changes, additions, or corrections…")
   - A "Apply Changes" button (indigo #4F46E5) that calls sendPrompt() with:
     CHANGES::{"changes": "<user input text>"}
   - A secondary "Continue to Send Email →" button (green #059669) that calls sendPrompt() with:
     ACTION::{"action": "send email to recipients"}
   - Widget title: "Review Follow-up Email"
   - Subtitle: "Would you like to make any changes, or continue to send the follow-up email?"

2. Widget Behavior:
   - "Apply Changes" button disabled if textarea is empty
   - Both buttons mutually exclusive — clicking one dismisses the other
   - Card layout, rounded corners, subtle drop shadow, single column

3. After creating the widget, always say to user to fill the above widget or directly give changes in the chat input.

4. If CHANGES:: payload received:
   → Parse requested changes from JSON
   → Apply all changes to the email
   → Re-render the Step 5 widget again so the user can request further changes or proceed
   → After rendering the widget, always say to user to fill the above widget or directly give answer in the chat input

5. If ACTION::{"action": "draft_email"} received:
   → Proceed directly to Step 6

---

## Step 6 — Send the Email

Send the email to the participents in the meeting using Office365 through the CData connector.
- Confirm tools via getProcedures() first
- Call queryData or executeProcedure as required by the CData Office365 send-mail interface.
- Use Office365 `@Attachments` parameter for passing document base64 string, while attaching a document in mail.
- `@ContentType` parameter should have value in lowercase.

---

## Edge Cases

- **No transcript available**: inform the user and End.
- **Very short transcript**: Draft with whatever is available; note that the email may be brief.
- **Multiple attendees without names**: Use "the team" or "all attendees" as placeholders.
- **No action items found**: Note this and have placeholder text as no action items
- **Technical meeting / heavy jargon**: Preserve technical accuracy but keep the email accessible.