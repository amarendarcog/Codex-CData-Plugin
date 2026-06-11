# CRM Context Lookup (CData Connect AI → Salesforce)

Read this when the account being researched might already exist in the user's Salesforce. CRM context dramatically changes Section 6 (stakeholder mapping) and Section 12 (next best actions) — if a deal is open or a CSM is engaged, the brief should reflect that, not pretend the account is cold.

## When to use

- The user mentions an existing customer or opportunity.
- The skill is being run as part of a QBR or expansion play.
- The account name produces a hit when queried against Salesforce.


## Step 1 — Get driver instructions

Always start by getting connector Instructions for how it expects to be called:

The driver instructions tell you the exact catalog/schema names and any quirks of the user's Salesforce instance.

## Step 2 — Find the account

Search for any existing account using the company name.
If no rows return, write "No prior Salesforce record found" in Section 13 and skip the rest of this lookup.

## Step 3 — Pull opportunities

fetch available opportunities for the above account
Open opportunities feed Section 12 (next best action: don't cold-outreach into an active deal).
Closed-lost opportunities tell you what objections to expect — surface them in Section 11 (Competitive Context).

## Step 4 — Pull contacts

fetch the contacts for that account
Cross-reference these against the Apollo / LinkedIn list in Section 6. If the same person appears in both, mark them as "known to us" in the Notes column.

## Step 5 — Pull recent activity

fetch the most recent tasks created on that account
The most recent activity drives timing. If the account was emailed three weeks ago and went cold, your sequence should acknowledge that, not pretend it's a fresh start.

## What to do with the results

- **Account owner found** → put their name in the brief header instead of leaving it blank.
- **Open opportunity** → recast Section 12 around accelerating the deal, not opening the account.
- **Closed-lost record** → mine the description for objections; reflect them in Section 11.
- **Recent activity** → adjust the sequence in Section 10 so Day 1 isn't a duplicate of an existing thread.
