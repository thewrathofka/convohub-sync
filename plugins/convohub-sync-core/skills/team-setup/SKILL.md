---
name: team-setup
description: First-run onboarding for a Superside team into the sales plugins. Resolves the running BDR's team, and if that team's Notion databases don't exist yet, bootstraps them — asking the team name and whether the team already has a Notion page, linking the team page under the Sales page, creating the "Databases" page with the four databases (Scoreboard, All Accounts, Conversation Hub, All Meetings) from the exact schema spec, seeding the rep roster + AEs, and optionally importing the rep's Salesforce priority accounts. Runs automatically the first time any sync can't find a team's databases; can also be invoked directly, e.g. "set up my team" or "/team-setup".
---

# team-setup (first-run team bootstrap)

Read these shared files fully before doing anything (via `${CLAUDE_PLUGIN_ROOT}/shared/…`), then follow
them exactly — they hold all IDs, the exact schema, and the mappings:
- `shared/team-resolution.md` — resolve the running BDR → pod/team and find (or fail to find) the team's
  databases. **Run this first.** If the team is already fully set up, say so and stop — there's nothing to
  bootstrap.
- `shared/roster.md` — the BDR→pod roster used to seed the team's Squad + Scoreboard rows.
- `shared/notion-setup.md` — the exact, verbatim schema of the four databases + the All Leads view + the
  relation graph + build order. Recreate the setup from this, faithfully.
- `shared/salesforce-import.md` — the optional priority-account import (locked priority mapping, `SDR__c`
  ownership, descriptive-only fields).

## When this runs
Automatically when `team-resolution.md` finds no team page / no Databases page / missing databases for the
resolved rep's pod (Step 5 there hands off here). Or on explicit request.

## Bootstrap sequence (the one place a run is allowed to interrupt and ask)
1. **Team name + existing page.** Ask the rep: what is the team's name, and does the team already have a
   Notion page? Use the existing page if so; otherwise create the team page.
2. **Link it under the Sales page.** Write the team page's link into the Sales page's "BDR Teams:" section
   (`notion-update-page` on the Sales page `2e650ebb-bec0-8091-8436-c06c04ccecca`), so future resolution
   finds it with no asking. Record the team name back into the pod's row in `roster.md` (so the package
   ships knowing it next republish).
3. **Databases page + four DBs.** Create the "Databases" child page, then build Scoreboard, All Accounts,
   Conversation Hub, All Meetings exactly per `notion-setup.md`: empty DBs → properties → relations →
   **set each default view's column order/visibility via `notion-update-view` SHOW per §10** → create the
   two All Meetings monthly views (current month + one back) per §11 → All Leads view. Idempotent — never
   duplicate an existing DB/property/option. Then report the **manual finish-up
   steps** the API can't do (§7–§10): paste the Scoreboard/All Accounts formulas (§9), set the custom All
   Meetings `Status` options/groups, and position the formula columns per §10 — hand the rep this short
   checklist rather than implying the copy is 100% complete.
4. **Seed reps + AEs.** From `roster.md`, add the pod's BDRs to the team page's `# The Squad` and create a
   Scoreboard row per BDR. Ask for each rep's AE (person) and set it. AE may be left blank → **lazy fill**:
   any later plugin run by a rep whose `AE` is blank asks their AE once and writes it.
5. **Offer the Salesforce import.** Per `salesforce-import.md` — ask first; if declined, continue (All
   Accounts backfills organically as the syncs run).
6. **Report.** Summarize what was created/linked, which reps + AEs were seeded, import counts (or that it
   was skipped), and that the analytics formula columns were left for manual re-entry.

Never guess team identity or write into another team's space — if resolution is ambiguous, ask.
