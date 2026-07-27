---
name: email-sync
description: Daily email-reply check for the rep invoking it. Reads the rep's Gmail for external people who replied to outreach (even outreach sent from another domain via Apollo) or sent a genuine inbound since the last run, registers only those with no meeting yet or a re-engageable Closed-Lost opp, logs them into the Notion Conversation Hub with channel Email, a reply-intent priority, a conversation summary, a suggested follow-up date, and a drafted (never sent) Gmail reply. Starts by reconciling existing email threads and email-booked meetings in the funnel. Use when a rep asks for their daily email check, e.g. "run my email-sync" or "/email-sync".
---

# email-sync (daily)

Run the workflow in `${CLAUDE_PLUGIN_ROOT}/shared/core-logic.md` with:

- `EMAIL_WINDOW_DAYS` = 1 — window per Step 0: from the rep's `Last email check` timestamp through now, so nothing since the last check (by any of the three commands) is missed; falls back to yesterday 00:00 (or the preceding Friday 00:00 on a Monday) only if no prior run is on record.
- `DEEP_RESOLUTION` = false — this command runs Step 7's follow-up cadence in its light form (advancing only rows touched this run or with an obvious state change). Everything else in core-logic.md, including Step 1's Conversation Hub + All Meetings reconciliation sweep, runs exactly as written.

Read the full core-logic.md file before doing anything else in this run — it contains all fixed IDs, schema details, and step-by-step instructions. Do not skip reading it even if you recall it from a prior run; schema options and property names can change.
