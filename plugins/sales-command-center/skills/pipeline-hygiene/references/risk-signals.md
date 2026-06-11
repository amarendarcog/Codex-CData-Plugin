# Risk Signal Definitions

This file defines the nine risk signals evaluated for every open opportunity.
A signal fires only when there is direct evidence from retrieved data. Keep field names, raw values, source names, and connector details internal; the user-facing report should state business conclusions only.

---

## Signal Catalogue

| # | Signal | Definition | Evidence Required |
|---|--------|-----------|-------------------|
| 1 | **Deal inactivity** | No activity logged in 14+ days | Last recorded activity is more than 14 days old and there are no future tasks |
| 2 | **Missing next steps** | No open task and no next step captured | No open follow-up task and no clear next step in the opportunity record |
| 3 | **No exec engagement** | No VP-level or above contact in activities in the last 30 days | Contact titles in recent activities — none with VP, SVP, EVP, C-level, or Director |
| 4 | **Weak champion** | Champion contact not marked or not recently engaged | No contact flagged as champion, OR champion not appearing in calls/emails in the last 30 days |
| 5 | **Competitive mention** | Competitor named in meetings, emails, or team discussions | Recent engagement text contains a known competitor name |
| 6 | **Negative sentiment** | Meeting or email flagged with negative or at-risk language | Recent engagement includes language such as "concerned", "pausing", "cancelled", "not moving forward", or "budget freeze" |
| 7 | **Forecast inconsistency** | Close date within 30 days but stage probability below 50% | Deal is scheduled to close within 30 days while the current stage indicates less than 50% probability |
| 8 | **Stage aging anomaly** | Opportunity stuck in the same stage beyond the median duration for that stage | Time in current stage exceeds median calculated across all same-stage opportunities |
| 9 | **Deal hygiene issues** | Missing required deal details | Missing amount, close date, owner, or next step for stages above Qualification |

---

## Risk Score Thresholds

| Score | Signals Present | Override Condition |
|-------|-----------------|--------------------|
| 🔴 High Risk | 3 or more | OR: signals 1 + 3 + 6 together (inactivity + no exec + negative sentiment) |
| 🟡 Medium Risk | 1–2 | — |
| 🟢 Low Risk | 0 | — |

---

## Stage-to-Probability Mapping

Use this mapping when the CRM does not expose a probability field directly:

| Stage Name (Common Variants) | Default Probability |
|-------------------------------|---------------------|
| Prospecting, Lead, Inquiry | 5–10% |
| Discovery, Qualification | 10–20% |
| Demo, Evaluation, Proof of Concept | 30–40% |
| Proposal, Quote Sent | 40–50% |
| Negotiation, Contract Review | 60–80% |
| Verbal Commit, Pending Approval | 90% |
| Closed Won | 100% |
| Closed Lost | 0% |

If the stage name does not match any of the above, use 25% as a conservative default and note the ambiguity in the output.

---

## How to cite a signal

Every fired signal must be backed internally by:
1. Signal name
2. The business fact that triggered it
3. The supporting date, amount, contact, or activity detail when relevant

**Good internal evidence:**
> Signal: Deal inactivity | Last meaningful activity was 17 days ago | No future task found

**Bad evidence:**
> This deal looks stale.
