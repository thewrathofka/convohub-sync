# Internal history (Salesforce-led)

The **internal** half of `/account-intel`: everything we already know about this account. Read-only from
Salesforce (+ Jiminny, Gmail, and the team's Notion Conversation Hub / All Meetings). Always recomputed
each run (unlike external research, it changes daily). Resolve tools by name — Salesforce `soqlQuery`,
`find`, `getUserInfo`; Jiminny `search_calls`, `get_call`; Gmail `search_threads`, `get_thread`; Notion
`notion-query-data-sources`, `notion-fetch`. Team DB IDs come from `team-resolution.md`.

## 0. Resolve the account
Get the Salesforce `Account.Id` + `Website`/domain (`find` by name, or `soqlQuery` on Account). Derive
the email domain from `Website`. Keep the `AccountId` and domain — every query below keys on them. If no
confident SF account match, say so and fall back to Notion All Accounts + domain only.

## 1. Activity timeline — 18 months (the raw material for "Conversations Dropped")
`Task.AccountId` and `Event.AccountId` are directly populated, so no polymorphic joins:
```sql
SELECT ActivityDate, Type, TaskSubtype, Direction__c, Subject,
       CallDisposition, CallDurationInSeconds, CallType,
       SalesLoft1__Call_Sentiment__c, Owner.Name, Who.Name, WhoId, WhatId
FROM Task
WHERE AccountId = :accountId AND ActivityDate >= LAST_N_MONTHS:18
ORDER BY ActivityDate DESC
```
```sql
SELECT StartDateTime, Type, Subject, Meeting_Type_Jiminny__c, No_Show__c, Owner.Name, Who.Name
FROM Event
WHERE AccountId = :accountId AND StartDateTime >= <18 months ago>
ORDER BY StartDateTime DESC
```
Channel identification: `Type LIKE 'LinkedIn%'` (or `TaskSubtype='LinkedIn'`) = LinkedIn / Sales Navigator;
`Type='Email'`/`TaskSubtype='Email'` = email; `Type='Call'` = cold call. `Direction__c` (`In`/`Out`) and
the `[Apollo] [Email] >>`/`<<` subject convention mark inbound vs outbound. Also read
`Account.LastActivityDate` for cheap last-touch recency.

## 2. "Conversations Dropped" — the core detection
A **dropped conversation** = a person/thread that showed **positive interest** but has **no follow-up, or
a stale one**, and no upcoming booked meeting / no open opp. Sweep the 18-month window across three
channels:
- **Cold calls** — `Type='Call'` with `CallDurationInSeconds > 60` **and** a positive/connected outcome
  (`CallDisposition`/`SalesLoft1__Call_Sentiment__c` indicating connected-positive; cross-check Jiminny
  `get_call` sentiment/summary if a call is ambiguous). Sub-60-second calls are ignored.
- **Emails** — an **inbound** reply (`Direction__c='In'` / `<<`) showing interest, cross-checked in Gmail
  (`search_threads`) for the actual back-and-forth.
- **LinkedIn / Sales Nav** — `Type LIKE 'LinkedIn%'` with a reply/positive response.
"No/stale follow-up" = the last touch on that person is an **unanswered outbound**, or there's been **no
outbound at all since the interest**, and there is **no upcoming meeting** (Event/All Meetings) and **no
open Opportunity** for them. Skip anyone whose Conversation Hub page already has `Had meeting?` checked or
an active follow-up cadence in progress (that's ConvoHub Sync's job — don't double-surface).

For **each** dropped item capture: person name + title, company, channel, the **date** of the last
positive signal, a **1–2 sentence summary** of what was discussed (from the SF subject/notes + Jiminny
summary + Gmail snippet), **age in months**, and a **temperature** (see `scoring.md`: HOT ≤30d · WARM
1–3mo · COOL 3–6mo · DORMANT 6–18mo). Order newest-first; the ≤30-day (HOT) ones lead and each gets a
drafted follow-up (the revival candidates).

## 3. Leads (2 years) — footprint + MQLs
Distinct people who engaged in the last 2 years, and the MQL/campaign signals:
- **Contacts with recent activity:**
  `SELECT Id, Name, Title, LastActivityDate FROM Contact WHERE AccountId = :accountId AND LastActivityDate >= LAST_N_MONTHS:24`
- **Distinct people actually touched:** `SELECT COUNT_DISTINCT(WhoId) FROM Task WHERE AccountId = :accountId AND ActivityDate >= LAST_N_MONTHS:24`
- **Leads + MQLs by domain** (this is the Apollo/marketing MQL feed — see below):
  ```sql
  SELECT Id, Name, Title, Company, Email, LeadSource, Status, IsConverted, CreatedDate
  FROM Lead
  WHERE Email_Domain__c = :domain AND CreatedDate >= LAST_N_MONTHS:24
  ORDER BY CreatedDate DESC
  ```
Headline **`Leads (2y)`** = distinct people (contacts-with-activity ∪ leads on domain), deduped
client-side (SOQL has no UNION; run the counts separately and reconcile). List them newest-first in the
page `### Leads` section with source + date + status.

### Apollo MQLs (website visits / call requests) — read from Salesforce, not Apollo
Apollo's MCP can't return per-account intent, but the MQLs land in SF as Leads. Flag those whose
`LeadSource IN ('Website Visitors','Book-A-Call Form','Content Download/View','Content Subscription','Apollo')`
— `Website Visitors` = website visits, `Book-A-Call Form` = call/demo requests. Also read the account-level
synced intent scores off the SF Account when present (`isMQA`, 6sense intent score, Aggregated
Engagement) to color the section and feed the score.

## 4. "Close Lose" — closed-lost, last 2 years, with stall analysis
```sql
SELECT Id, Name, StageName, CloseDate, Amount, Loss_Reason__c, Loss_Details__c,
       Win_Loss_Detail__c, Retarget_lost_notes_from_AE__c, Prospect_intention_after_Closed_Lost__c,
       Owner.Name
FROM Opportunity
WHERE AccountId = :accountId AND IsClosed = true AND IsWon = false
  AND CloseDate >= LAST_N_MONTHS:24
ORDER BY CloseDate DESC
```
For each: report `Loss_Reason__c` + `Loss_Details__c` + the AE's `Retarget_lost_notes_from_AE__c`, a
**summary of the conversations**, and **where/why it stalled** — enriched from **Jiminny**: `search_calls`
for calls tied to that opp/account around the deal window, `get_call` for the summary/transcript, and read
out the point the deal went quiet (last real engagement → what was the open question / unmet need). The
closed-lost reason also feeds the Readiness modifier in `scoring.md` (re-engageable vs neutral vs
disqualifying).

## 5. Notion cross-check
Pull the account's existing **Conversation Hub** pages (people already tracked) and **All Meetings** rows
for this team, so the Intel doesn't re-surface people already in an active cadence and so revival
dedups against existing Hub pages (Name + Account) exactly like ConvoHub Sync.

## Degradation (non-BDRs / missing connectors)
Salesforce + Notion reads work for anyone. **Jiminny and Gmail are BDR-only** — if absent, the call/email
enrichment and the Close-Lose stall analysis degrade gracefully (use the SF activity + Loss fields alone)
and say so in the output. Non-BDRs never get the revival step (that's `SKILL.md`'s ask-gated BDR path).
