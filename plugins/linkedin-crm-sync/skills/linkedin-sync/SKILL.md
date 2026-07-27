---
name: linkedin-sync
description: Reads the rep's own LinkedIn inbox, matches conversation participants against accounts assigned to them in the Notion All Accounts CRM, summarizes matched conversations into Conversation Hub, cross-checks Salesforce and the calendar, and reports back in chat. Never sends a LinkedIn message as part of this automatic pass. Use when a rep asks to sync their LinkedIn inbox, check who they're talking to from their account list, or "run my linkedin sync" / "/linkedin-sync".
---

# linkedin-sync

Run the workflow in `${CLAUDE_PLUGIN_ROOT}/shared/core-logic.md`, Steps 0-7 only.

Read the full core-logic.md file before doing anything else in this run — it contains the fixed IDs, schema, and step-by-step instructions. Do not skip reading it even if you recall it from a prior run; schema options and property names can change.

Step 0 resolves which rep is running this from their own LinkedIn login — this plugin is shared across the team, so never assume it's any specific person. If Step 0 can't confidently match the logged-in LinkedIn account to a Monthly Scoreboard row, stop and ask rather than guessing.

Step 8 (sending a LinkedIn message) is a separate, explicit action — never triggered by this command running. If the rep wants to message someone, that's a distinct ask handled per Step 8's rules, requiring their explicit go-ahead in that turn.
