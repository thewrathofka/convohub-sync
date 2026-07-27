---
name: convohub30
description: 30-day cold-call follow-up audit for the rep invoking it. Sweeps the last 30 days of connected cold calls from Jiminny, resolves no-shows on booked meetings, flags Positive/Neutral/CP-not-now-or-authority contacts with zero follow-up, resolves accounts in All Accounts, runs the All Meetings cleanup sweep, advances the Conversation Hub follow-up cadence and nurture hand-off, and drafts (never sends) Gmail follow-ups. Use when a rep wants a deep monthly catch-up sweep to find anyone who fell through the cracks, e.g. "run my convohub30" or "/convohub30", "check the last month".
---

# convohub30 (30-day audit)

Run the workflow in `${CLAUDE_PLUGIN_ROOT}/shared/core-logic.md` with:

- `CALL_WINDOW_DAYS` = 30
- `RUN_NO_SHOW_RESOLUTION` = true — this command DOES run Step 4 (the deal-activity no-show resolution check on Meeting-booked calls pulled in this window), in addition to everything else in core-logic.md.

Read the full core-logic.md file before doing anything else in this run — it contains all fixed IDs, schema details, and step-by-step instructions. Do not skip reading it even if you recall it from a prior run; schema options and property names can change.
