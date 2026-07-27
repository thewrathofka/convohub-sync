---
name: convohub
description: Daily cold-call follow-up check for the rep invoking it. Pulls today's connected cold calls from Jiminny, flags new Meeting-booked/Positive/Neutral/CP-not-now-or-authority contacts, resolves their account in All Accounts, runs the standing All Meetings cleanup sweep, advances the Conversation Hub follow-up cadence and nurture hand-off, and drafts (never sends) Gmail follow-ups. Use when a rep asks for their daily cold-call/conversation-hub check, e.g. "run my convohub" or "/convohub".
---

# convohub (daily)

Run the workflow in `${CLAUDE_PLUGIN_ROOT}/shared/core-logic.md` with:

- `CALL_WINDOW_DAYS` = 1 — window per Step 2: from the rep's `Last Convohub Run` timestamp through now, so nothing since the last check (by any of the three commands) is missed; falls back to yesterday 00:00 (or the preceding Friday 00:00 on a Monday) only if no prior run is on record.
- `RUN_NO_SHOW_RESOLUTION` = false — this command does NOT run Step 4 (the deal-activity no-show resolution check tied to newly-pulled calls). Everything else in core-logic.md, including Step 5's All Meetings cleanup sweep and Step 7's follow-up cadence, runs exactly as written.

Read the full core-logic.md file before doing anything else in this run — it contains all fixed IDs, schema details, and step-by-step instructions. Do not skip reading it even if you recall it from a prior run; schema options and property names can change.
