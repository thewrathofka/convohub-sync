---
description: Sync your LinkedIn inbox against tracked Notion accounts, cross-check Salesforce and your calendar, and update Conversation Hub / All Meetings.
---

Run the workflow in `${CLAUDE_PLUGIN_ROOT}/shared/core-logic.md`, Steps 0-7 only.

Read the full core-logic.md file before doing anything else in this run — it contains the fixed IDs, schema, and step-by-step instructions. Do not skip reading it even if you recall it from a prior run; schema options and property names can change.

Step 0 resolves which rep is running this from their own LinkedIn login — this plugin is shared across the team, so never assume it's any specific person. If Step 0 can't confidently match the logged-in LinkedIn account to a Monthly Scoreboard row, stop and ask rather than guessing.

Step 8 (sending a LinkedIn message) is a separate, explicit action — never triggered by this command running. If the rep wants to message someone, that's a distinct ask handled per Step 8's rules, requiring their explicit go-ahead in that turn.
