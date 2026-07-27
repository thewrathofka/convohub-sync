# Salesforce priority-account import (shared)

Two entry points share this logic:
- **Bulk import** at first-run bootstrap (`skills/team-setup`, Part D) — offer to pull a rep's whole
  priority book into All Accounts.
- **Organic backfill** — whenever any sync (convohub / linkedin / email) encounters an account not yet
  in All Accounts, create it with the right priority + descriptive fields, on the fly.

Resolve Salesforce tools by name: `getUserInfo`, `find`, `soqlQuery`. Read-only except the Notion writes
into All Accounts. Never run DML against Salesforce.

## Ownership + priority mapping (locked)
- **"A rep's accounts"** = ownership on **`SDR__c`** (the BDR's assigned book — default). (Alternative
  `OwnerId` exists; `SDR__c` is authoritative for BDR prospecting. For Yehia: `SDR__c`≈906 vs
  `OwnerId`≈828.)
- **Priority mapping** (SF `Priority_Account__c` → Notion All Accounts `Priority Type`):

  | SF value | Notion `Priority Type` | Applies to |
  |---|---|---|
  | `A` | `A` | all reps |
  | `B` | `B` | all reps |
  | `C` | `BDM Selected` | all reps |
  | `BU Expansion Priority` | `BU Expension` | only the three BU-eligible reps¹ |
  | `BU Focus 5` | `BU Expension` | only the three BU-eligible reps¹ |
  | anything else / null | skip | — |

  ¹ BU-eligible reps (from `roster.md`): **Katarina Misovic, Alvaro Andres Villavicencio Espinoza,
  Gabriela Moncada Madriñán**. For everyone else, `BU Expansion Priority` and `BU Focus 5` are not
  imported.

## Bulk import (bootstrap)
1. **Ask the rep first.** If they decline → skip; All Accounts fills in via organic backfill. Offer a
   paste-a-list fallback if they'd rather not use Salesforce.
2. Resolve the rep's SF user id: `getUserInfo` (self) or `find`/`soqlQuery` on `User` by the rep's
   Superside email.
3. Query (parameterized by `:repId`; add the BU tiers only for the three eligible reps):
   ```sql
   SELECT Id, Name, Website, Domain__c, Industry, NumberOfEmployees, LastActivityDate,
          Priority_Account__c, Prospecting_Status__c, Parent.Name
   FROM Account
   WHERE SDR__c = :repId
     AND Priority_Account__c IN ('A','B','C')          -- + 'BU Expansion Priority','BU Focus 5' iff eligible
   ORDER BY Priority_Account__c, Name
   ```
   **Paginate and retry** — the org is ~188K accounts and the connector times out intermittently. Page
   with `LIMIT`/`OFFSET` or Id-cursor; retry a timed-out page before giving up; report how many pages/rows
   were pulled and whether anything was dropped (never silently truncate).
4. For each account, **upsert** into the team's All Accounts (match by `Account Name` / domain first;
   never duplicate) with **descriptive fields only** (see below) and `Priority Type` per the map. Set the
   `Rep` relation to the running rep's Scoreboard row.
5. Report counts by tier and any accounts skipped/uncertain.

## Descriptive fields to set (never buying-signal / lead fields)
Set only: `Account Name` (Name), `Account URL` (Website), `Parent` (Parent.Name if any), `Size`
(bucket `NumberOfEmployees` → `<1000` / `1000-2000` / `2000-5000` / `5000+`), `Last Prospected` (from
`LastActivityDate` when it reflects prospecting), `Priority Type` (mapped), `Rep`.
**Never** set: `Prospecting Status`, `Temperature`, `Buying`-anything, `Research Status/Confidence`,
`Needs Attention`, `Pain Point Tags`, `Number of Connects`/`Total Number of Leads`, `Notes`, `Cluster`,
`Strategy`, `Structure Notes` — those are BDR- or research-agent-owned. (Note: the current All Accounts
schema has already dropped most raw buying-signal columns; this rule still governs the ones that remain.)

## Organic backfill (used by every sync's account-resolution step)
When a sync resolves an account that has **no** All Accounts row for this team:
1. Create the row with `Account Name` + `Rep`.
2. Look up that account in Salesforce for this rep (`find`/`soqlQuery` by name/domain, confirm it's the
   rep's via `SDR__c`), read `Priority_Account__c`, and set `Priority Type` per the map (respecting BU
   eligibility). If no confident SF match, leave `Priority Type` unset and flag — never guess a tier.
3. Fill the same descriptive-only fields as above where confidently known. Never buying-signal fields.
4. Continue the sync. This is how a team that skipped the bulk import still ends up with correctly-tiered
   accounts over time.
