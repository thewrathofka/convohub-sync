---
name: email-sync30
description: 30-day email-reply audit for the rep invoking it. Deep monthly sweep of the last 30 days of Gmail for external people who replied to outreach or sent a genuine inbound, registering only those with no meeting yet or a re-engageable Closed-Lost opp, logging them into the Notion Conversation Hub with channel Email, a reply-intent priority, a conversation summary, a suggested follow-up date, and a drafted (never sent) Gmail reply. Runs the full follow-up cadence and nurture hand-off, and reconciles existing email threads and email-booked meetings in the funnel. Use when a rep wants a deep monthly catch-up to find anyone who fell through the cracks, e.g. "run my email-sync30" or "/email-sync30", "check the last month of email".
---

# email-sync30 (30-day audit)

Run the workflow in `${CLAUDE_PLUGIN_ROOT}/shared/core-logic.md` with:

- `EMAIL_WINDOW_DAYS` = 30 — window per Step 0: today − 30 days at 00:00 local time through now.
- `DEEP_RESOLUTION` = true — same as email-sync7: runs Step 7's follow-up cadence in full across all of this rep's email-touch Conversation Hub rows, in addition to everything else in core-logic.md. Same logic as email-sync7 with a wider window.

Read the full core-logic.md file before doing anything else in this run — it contains all fixed IDs, schema details, and step-by-step instructions. Do not skip reading it even if you recall it from a prior run; schema options and property names can change.
