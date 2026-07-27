# Team resolution (shared)

Every plugin in this package runs this **before** its own Step 0, in place of any hardcoded Notion data
source IDs. It answers one question: *which team's four databases should this run read and write?* The
databases are named identically for every team but live on separate team pages, so resolution is by
**team**, never by database name alone.

The only hardcoded ID in the whole package is the anchor below. Everything else is discovered live from
the Notion page tree, so a database that gets rebuilt self-heals on the next run.

## Fixed anchor
- **Sales page** (`💸 Sales`): `2e650ebb-bec0-8091-8436-c06c04ccecca`. Contains a **"BDR Teams:"**
  section that links each team's Notion page.

Resolve MCP tools by name (no `mcp__<uuid>__` prefixes): Jiminny `get_me`; Notion `notion-fetch`,
`notion-query-data-sources`, `notion-get-users`, `notion-create-pages`, `notion-update-page`.

## Steps

1. **Identify the rep.** Call Jiminny `get_me` → name + email (`{first}.{last}@superside.com`). Jiminny
   also returns the rep's team label, but do not trust it to match the Notion team name — use it only as
   a hint.
2. **Map rep → pod/team via the bundled roster** (`${CLAUDE_PLUGIN_ROOT}/shared/roster.md`, authoritative).
   Fuzzy-match the rep by name/email to a roster row → pod (manager), segment, and team name (if set).
   If the rep isn't in the roster at all, ask them once which pod/team they're on, then continue (and note
   it so the roster can be updated later).
3. **Find the team's Notion page.** `notion-fetch` the Sales page; read its "BDR Teams:" section and
   follow the link whose title matches the resolved team name. If the team name isn't set yet in the
   roster, disambiguate by opening the linked team pages and matching the pod's manager and/or the
   `# The Squad` @mentions (resolve Notion users to emails via `notion-get-users`) against the running rep.
4. **Load the four databases.** On the team page's page list, open the child page titled **"Databases"**
   (accept **"Database"** too — Money Moguls uses the singular). Read its child databases and bind the
   four by name — **Scoreboard** (a.k.a. "Monthly Scoreboard"), **All Accounts**, **Conversation Hub**,
   **All Meetings** (a.k.a. "Meetings (DB)"). Return their data-source (`collection://…`) IDs; the rest of
   the run uses these instead of any hardcoded IDs. Cache them for the run only — never across runs.
5. **Not set up yet →** if the pod has no team page linked under Sales, or the team page has no
   Databases page / is missing databases, hand off to the first-run **bootstrap**
   (`${CLAUDE_PLUGIN_ROOT}/shared/notion-setup.md`, driven by `skills/team-setup`) before proceeding.

## Known teams (fast path)
Money Moguls is already set up; its Databases page is `30650ebb-bec0-80f4-b3a4-cdcbd864379b` and its live
data sources are Scoreboard `30950ebb-bec0-8003-bb35-000b6b021eec`, All Accounts
`a5268c41-9288-4977-92df-f541a790a1bf`, Conversation Hub `39d50ebb-bec0-8054-89c6-000b1f0a5914`, All
Meetings `30950ebb-bec0-80fc-81c8-000b4f854176`. Treat these as a cache/sanity-check, not a substitute
for resolution — always confirm the running rep actually resolves to Money Moguls first.

## Contract
Never write to another team's databases. If resolution is ambiguous (rep matches no team, or two team
pages both seem to match), stop and ask rather than guessing — writing a rep's data into the wrong team's
CRM is unrecoverable-by-cleanup and must never happen on a guess.
