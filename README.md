# ConvoHub Sync

A single installable Claude Code plugin marketplace bundling the Superside BDR sales toolkit. Add this
one marketplace and every team's reps get the same tools, each pointed at their **own** team's Notion
databases.

## What's in the bundle (`.claude-plugin/marketplace.json`)
- **convohub-sync-core** — team resolution, first-run Notion bootstrap, Salesforce priority-account
  import, and `/weekly-sync` (runs all three syncs together, weekly). Start here.
- **convohub** — cold-call follow-up review from Jiminny (`/convohub`, `/convohub7`, `/convohub30`), plus
  the `meeting-emails`, `followup-email`, and `ae-handover-notes` skills.
- **linkedin-crm-sync** — LinkedIn inbox → Conversation Hub (`/linkedin-sync`).
- **email-crm-sync** — Gmail replies + genuine inbound → Conversation Hub (`/email-sync`, `/email-sync7`,
  `/email-sync30`).

## Multi-team model
The three syncs no longer hardcode one team's databases. They call `convohub-sync-core`'s
`team-resolution.md`, which resolves the running BDR → their team via the bundled roster and the `💸 Sales`
page, then loads that team's four databases (Scoreboard, All Accounts, Conversation Hub, All Meetings)
live from the team's "Databases" page. The first time a team's databases don't exist, `team-setup`
bootstraps them from the exact schema spec. So one install serves every team, with no cross-team writes.

## Install order for a new team
1. Add this marketplace; enable `convohub-sync-core` + the three syncs.
2. Run any sync (or `/team-setup`). If the team isn't set up, the bootstrap runs once — team name, Notion
   page, databases, reps + AEs, optional Salesforce import.
3. Thereafter, `/weekly-sync` weekly (or the per-channel commands) — everything resolves automatically.

## Guarantees
Never auto-sends email or LinkedIn messages (drafts only). Never writes into another team's databases.
Never guesses identity/account matches — flags instead.
