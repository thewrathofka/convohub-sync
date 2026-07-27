---
name: email-sync7
description: 7-day email-reply audit for the rep invoking it. Sweeps the last 7 days of Gmail (not just since the last run) for external people who replied to outreach or sent a genuine inbound, registers only those with no meeting yet or a re-engageable Closed-Lost opp, logs them into the Notion Conversation Hub with channel Email, a reply-intent priority, a conversation summary, a suggested follow-up date, and a drafted (never sent) Gmail reply. Runs the full follow-up cadence and nurture hand-off, and reconciles existing email threads and email-booked meetings in the funnel. Use when a rep wants a wider catch-up sweep than the daily check, e.g. "run my email-sync7" or "/email-sync7", "check the last week of email".
---

# email-sync7 (7-day audit)

Run the workflow in `${CLAUDE_PLUGIN_ROOT}/shared/core-logic.md` with:

- `EMAIL_WINDOW_DAYS` = 7 — window per Step 0: today − 7 days at 00:00 local time through now.
- `DEEP_RESOLUTION` = true — this command runs Step 7's follow-up cadence in full across all of this rep's email-touch Conversation Hub rows (first/second follow-up verification and nurture hand-off), not just rows touched this run, in addition to everything else in core-logic.md.

Read the full core-logic.md file before doing anything else in this run — it contains all fixed IDs, schema details, and step-by-step instructions. Do not skip reading it even if you recall it from a prior run; schema options and property names can change.
