# linkedin-crm-sync

Reads a rep's own LinkedIn inbox, matches conversation participants against accounts assigned to them in the shared "Money Moguls" Notion CRM (`All Accounts` / `Conversation Hub` — the same databases `/convohub` writes to), cross-checks Salesforce and the rep's calendar, summarizes matched conversations into Conversation Hub, and reports back in chat. Sending a LinkedIn message is a separate, always-confirmed action, never automatic.

Runs in **Claude Code, locally** — not Cowork. The LinkedIn tool ([stickerdaniel/linkedin-mcp-server](https://github.com/stickerdaniel/linkedin-mcp-server)) is a Playwright browser-automation server authenticated via a persistent local browser profile at `~/.linkedin-mcp/profile/`; that can't run in a hosted Cowork sandbox.

## Shared across the team

This plugin is built to be installed by every rep, each running it against their own LinkedIn inbox and their own row in Monthly Scoreboard. There's no shared config that says who's running it — `shared/core-logic.md` Step 0 asks LinkedIn who's logged in (`get_my_profile`) and matches that name to a Monthly Scoreboard row every time. Each rep's LinkedIn browser profile lives only on their own machine, so one teammate's login never touches another's.

## Commands

- **`/linkedin-sync`** — pull the inbox, match against tracked accounts, cross-check Salesforce and calendar, summarize, write to Conversation Hub, report back. Never sends a message.

## Scope: inbox + existing connections only

This deliberately does **not** expose LinkedIn people/company/job search. `.claude/settings.json` hard-denies `search_people`, `connect_with_person`, `get_sidebar_profiles`, `search_companies`, `get_company_profile`, `get_company_posts`, `get_company_employees`, `search_jobs`, `get_job_details`, and `get_feed` at the permission layer — Claude cannot call these even if asked to. `get_person_profile` is only ever called on usernames pulled from the inbox itself, never from search.

`send_message` is intentionally left off both the allow and deny lists, so it always prompts for confirmation at the tool-call layer, on top of the chat-level confirmation `shared/core-logic.md` Step 8 requires. It never fires as part of `/linkedin-sync`.

Salesforce and calendar access are read-only lookups (Step 4.5 and Branch A) — this plugin never writes to Salesforce or creates/edits calendar events, only reads them to fill in `URL Salesforce` and to confirm a meeting's real date.

## Setup (one-time, per teammate)

Every teammate does this setup on their own machine before running `/linkedin-sync`:

1. `uvx mcp-server-linkedin@latest --login` — interactive headed login, using *your own* LinkedIn account. Writes your session to `~/.linkedin-mcp/profile/` on your machine only.
2. Confirm the deny-list is actually enforced (try a denied tool once, expect a hard block).
3. Make sure your Notion, Salesforce, and calendar MCP connectors are set up in your own Claude Code environment — the first run will prompt you to approve each tool call.
4. First real run: use a small `limit` on `get_inbox` and manually check the resulting Conversation Hub write before trusting it against the full inbox. Also confirm Step 0 resolved you to the *correct* Monthly Scoreboard row — if it can't find a confident match, it'll ask instead of guessing, but double-check anyway on your first run.

## What it never does

Never runs on a schedule (unlike `unify-sync`) — manual invocation only. Never calls a people/company/job search or discovery tool. Never sends a message without an explicit, distinct go-ahead from the rep in the same conversation turn. Never creates a new All Accounts row — only surfaces people already at accounts the rep is tracking. Never guesses on which Account, Conversation Hub page, or Monthly Scoreboard row a person belongs to — flags it in the run report instead. Never writes to Salesforce or the calendar, only reads them.

## A real risk, not boilerplate

`linkedin-mcp-server`'s own README states LinkedIn's User Agreement prohibits automated access and accounts using automated tools can be restricted or banned, with no guarantee of safety. This drives real professional accounts tied to actual BDR work — every teammate should keep runs infrequent and manual, and treat any authentication hiccups as a signal to stop and check their own account, not to retry harder.
