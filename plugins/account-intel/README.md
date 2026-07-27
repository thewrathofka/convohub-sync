# account-intel

**Account Intelligence** — the reverse of the ConvoHub Sync channel plugins. They *log* conversations
into the CRM; this *pulls the full picture back out* of one account. Run **`/account-intel <company>`** to
understand or size up an account — ideal when a BDR inherits a dormant book, or when anyone (even outside
sales) needs a fast read on a company.

## What it produces
- **Internal Intelligence** (on the account page, below the untouched `## Conversations` section):
  - **Account Intel Score** — an **ROI Score** (fit/value: Creative Fit ×7 · Trigger/Timing ×6 ·
    Reach/Speed ×4 · Size/ICP ×3) and a **Prospecting Readiness** score → verdict **NOW! / why not /
    whatever** (external hotness + engagement recency + revivable conversations + lead footprint, adjusted
    by the closed-lost reason).
  - **Conversations Dropped** — positive conversations (cold calls >60s, emails, LinkedIn/Sales Nav) from
    the last 18 months with no/stale follow-up, newest-first, each with a summary, age in months, and a
    temperature (HOT ≤30d · WARM · COOL · DORMANT). HOT ones lead, each with a drafted follow-up.
  - **Leads** — everyone who engaged in the last 2 years + Apollo MQLs (website visits, call requests) and
    campaigns, newest-first.
  - **Close Lose** — closed-lost opps (2y) with loss reason/details, conversation summaries, and
    where/why the deal stalled (enriched from the Jiminny calls on that opp).
- **External Research** (recreated from the Superside extension): Company Profile · Corporate Structure ·
  Company Snapshot · Marketing & Creative Signals · Caveats · Why Superside Could Win · Discovery
  Questions · Competitor Tools · Sources. The hygiene pass also fills All Accounts properties (`Parent`,
  `sister/child`, `Structure Notes`, `Company Structure`, `Size`) and flags anything noteworthy.

## Who can run it
- **BDRs** (on the team roster) → writes the intelligence block + hygiene + score properties to the
  account page, gives a chat recap, and **asks** whether to revive the dropped conversations into
  Conversation Hub (drafted follow-ups, under the running BDR — never sent without a yes, never sent at
  all by the plugin).
- **Anyone else (incl. non-sales)** → the same intelligence **in chat only** — no Notion writes, no
  revival, no drafts. (Jiminny/Gmail enrichment is BDR-only and degrades gracefully.)

## How it fits the bundle
Part of the ConvoHub Sync marketplace. Reuses `convohub-sync-core` for team resolution + BDR detection,
and the ConvoHub/email Conversation-Hub-feed + Gmail-draft logic for revival. External research runs on
Claude Code's built-in WebSearch (no external API key). It adds five All Accounts properties (`ROI Score`,
`Prospecting Readiness`, `Prospect Now`, `Leads (2y)`, `Last Intel Run`) — created for new teams by
`/team-setup`, or backfilled on already-set-up teams by a one-time bonus-setup.

## Requirements
Salesforce + Notion (Money Moguls workspace / the team's space) for the core intelligence; Jiminny + Gmail
add call/email enrichment and follow-up drafts for BDRs. Tools are referenced by name, so it works for
whichever team + person invokes it, in Claude Code or Cowork.

## Guarantees
Never edits the `## Conversations` section or other plugins' content. Never auto-revives — always asks.
Never sends email. Never guesses identity/account matches — flags instead.
