# convohub-sync-core

The shared backbone for the Superside sales plugins. It makes the three channel syncs
(`email-crm-sync`, `convohub`, `linkedin-crm-sync`) **multi-team** and adds a weekly run-all command.

## What it provides
- **Team resolution** (`shared/team-resolution.md`) — the one routine every plugin runs first to figure
  out *which team's* Notion databases to use. Only one ID is hardcoded (the `💸 Sales` page); each team's
  databases are discovered live: Sales page → team page → "Databases" page → the four DBs by name.
- **First-run bootstrap** (`skills/team-setup` + `shared/notion-setup.md`) — when a team's databases don't
  exist yet, it asks the team name, links the team page under the Sales page, and creates the four
  databases (Scoreboard, All Accounts, Conversation Hub, All Meetings) from an exact, verbatim schema
  spec, seeds the reps + AEs, and offers the Salesforce import.
- **Salesforce priority-account import** (`shared/salesforce-import.md`) — pulls a rep's A/B/C
  (+ BU-curated, for eligible reps) accounts into All Accounts with the locked priority mapping and
  descriptive-only fields; also defines the organic backfill every sync uses.
- **`/weekly-sync`** (`skills/weekly-sync`) — resolves rep + team once, then runs the email, cold-call,
  and LinkedIn syncs together over a 7-day window and reports once. Weekly only.
- **Roster** (`shared/roster.md`) — the bundled BDR → pod → segment → team map.

## Requirements
Each rep needs their own Jiminny, Notion (Money Moguls workspace / their team's space), Gmail, Calendar,
and Salesforce connected. Tools are referenced by name, so the same package works for any rep on any team.

## Notes
- Draft-never-send is preserved everywhere; nothing is auto-sent.
- The Scoreboard's analytics formula columns can't be created via the API — the bootstrap builds
  everything else and documents those for manual re-entry.
