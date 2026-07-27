---
name: weekly-sync
description: Runs all three Superside channel syncs together for the invoking BDR on a weekly cadence — email replies/inbound (Gmail), cold-call follow-ups (Jiminny), and LinkedIn inbox — over a 7-day window, into the team's Notion Conversation Hub / All Accounts / All Meetings, and emits one combined report. Resolves the rep + team once and reuses it across all three. Weekly only (no daily). Use when a BDR wants their full weekly catch-up, e.g. "run my weekly sync" or "/weekly-sync".
---

# weekly-sync (run all three channel syncs, weekly)

One weekly pass over every channel. Resolve the rep and team **once**, make sure the team is set up, then
run the three 7-day syncs in sequence and report once. Draft-never-send is preserved throughout (each
sync only drafts Gmail follow-ups / LinkedIn messages; nothing is auto-sent).

## Procedure
1. **Resolve rep + team once.** Follow `${CLAUDE_PLUGIN_ROOT}/shared/team-resolution.md` to identify the
   running BDR and load the four team database IDs. If the team isn't set up yet, that file hands off to
   `team-setup` (bootstrap) — let that finish, then continue. Pass the resolved team database IDs to each
   sync below so none of them re-resolve or hardcode IDs.
2. **Email — 7 days.** Run the `email-crm-sync` core logic with `EMAIL_WINDOW_DAYS = 7`,
   `DEEP_RESOLUTION = true` (same as `/email-sync7`).
3. **Cold calls — 7 days.** Run the `convohub` core logic with `CALL_WINDOW_DAYS = 7`,
   `RUN_NO_SHOW_RESOLUTION = true` (same as `/convohub7`). This also runs the pre-meeting `meeting-emails`
   workflow (convohub Step 1e), so that's covered — don't run it separately.
4. **LinkedIn — 7 days.** Run the `linkedin-crm-sync` core logic over the last 7 days (its Step 3 window).
5. **One combined report.** Merge the three syncs' summaries into a single report grouped by channel:
   per person name + company, what was registered/updated, priority, and any drafted follow-up; plus each
   sync's skip/flag counts. Note anything that needs the rep's attention (ambiguities each sync flagged,
   AE asked, accounts backfilled).

## Rules
- Run the three syncs **sequentially**, not interleaved, so their Notion writes don't race; each dedups
  against the Hub as usual, so a person touched on two channels is updated, never duplicated.
- Weekly only — there is no daily variant here; the per-channel daily commands still exist for that.
- If any one sync errors, finish the others and report the failure rather than aborting the whole run.
- Do not send anything. Every follow-up remains a draft for the rep to review.

Read each sync's `core-logic.md` before running it (schemas/options drift); resolve all MCP tools by name.
