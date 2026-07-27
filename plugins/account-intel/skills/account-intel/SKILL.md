---
name: account-intel
description: Account Intelligence for a single account — the reverse of the channel syncs. Pulls a full internal + external picture of a company and (for BDRs) writes it onto the account page, with an ROI Score and a Prospecting Readiness verdict (NOW! / why not / whatever). Internal = past comms, dropped conversations to revive, leads/MQLs, closed-lost with stall analysis; external = company profile, corporate structure, creative signals (also fills All Accounts hygiene properties). Use when someone wants to understand or size up an account — e.g. "account intel on Acme", "/account-intel Granicus", "what's the story on this account I just inherited", "is this account worth prospecting". Works for anyone, even outside sales (read-only chat for non-BDRs).
---

# account-intel

Produce a complete intelligence picture of one account. Read all four shared files fully before running
(`${CLAUDE_PLUGIN_ROOT}/shared/…`) — they hold the queries, prompts, property maps, and scoring:
`internal-history.md`, `external-hygiene.md`, `scoring.md`, `bonus-setup.md`. Resolve every MCP tool by
name; schema/options drift, so re-fetch live when unsure.

## Guiding rules
- **Never guess identity/factual matters** (which account, which person, whether a deal really stalled) —
  flag and skip instead, same discipline as the sync plugins.
- **Draft-never-send**, and **never auto-revive** — the revival step is always ask-gated (Step 6).
- **BDR vs non-BDR is the master switch:** a BDR (on the roster) gets page writes + the revival offer; a
  non-BDR gets the exact same intelligence **in chat only** — no Notion writes, no revival, no drafts.

## Steps
0. **Resolve.** Identify the running person and whether they're a BDR (via `convohub-sync-core`'s
   `team-resolution.md` + `roster.md`). Resolve the account → the team's Notion All Accounts page + the
   Salesforce Account (name/domain). If the person is a BDR, also resolve which team's databases to use.
   If the account can't be confidently resolved, ask rather than guess.
1. **Freshness / mode.** Read `Last Intel Run` on the All Accounts page. If **< 3 days**, reuse the
   existing `## External Research` block (skip re-searching); internal history + scores are always
   recomputed. Set the output mode: **BDR → page + chat**; **non-BDR → chat only**.
2. **Internal history** — follow `internal-history.md`: 18-month activity timeline; cold calls >60s
   connected-positive; **Conversations Dropped** (positive + no/stale follow-up, temperature-tagged);
   **Leads (2y)** + Apollo MQLs (from Salesforce); **Close Lose** (closed-lost 2y + Jiminny stall
   analysis). Jiminny/Gmail enrichment is BDR-only; degrade gracefully otherwise.
3. **External research + hygiene** — follow `external-hygiene.md`: Pass A (CRM Hygiene → writes
   `Parent`/`sister-child`/`Structure Notes`/`Company Structure`/`Size` to All Accounts + Company
   Profile/Corporate Structure sections) and Pass B (Account Research sections). BDR only writes the All
   Accounts properties; for a non-BDR, produce the same content for chat without writing.
4. **Score** — follow `scoring.md`: **ROI Score** (0–100) and **Prospecting Readiness** (0–100) →
   **`Prospect Now`** verdict (NOW! / why not / whatever), with the External-Triggers input and the
   closed-lost modifier + overrides. Show each sub-score's 1-line evidence.
5. **Assemble + write.** Build the page block in this order, placed **directly below the untouched
   `## Conversations` section** (ConvoHub Sync owns Conversations — never edit it):
   - `## Internal Intelligence` → `### Account Intel Score` (both scores + breakdown) → `### Conversations
     Dropped` (newest-first; HOT ≤30d at top, each with a **Suggested Follow-up** sub-heading + drafted
     text) → `### Leads` → `### Close Lose`.
   - `## External Research` (the extension-structured sections).
   On re-run, refresh these two sections in place (newest data replaces the prior Intel block); never
   duplicate a heading, never touch Conversations or other plugins' content. Also set `Last Intel Run` =
   today and write the score properties (`ROI Score`, `Prospecting Readiness`, `Prospect Now`,
   `Leads (2y)`). **BDR only.** Non-BDR → emit the whole thing in chat, no writes.
   - Before any write, ensure the score properties exist — run `bonus-setup.md` (adds them to All Accounts
     if missing).
6. **Revival (BDR only, ASK FIRST).** Present the Conversations-Dropped people (lead with the HOT ≤30d
   ones and their drafts) and **ask whether to add them to Conversation Hub.** Only on an explicit yes:
   for each chosen person follow ConvoHub Sync's logic fully — create/dedup the Hub page (Name + Account)
   **under the running BDR** as `Rep`, complete every inferable Conversation Hub property (Title,
   `Account`, `Connected` = the channel, `Disposition`, `Priority` from temperature, `Suggested
   follow-up` date, `Suggested channel next`, `Quick notes`, `URL Salesforce`, page content), and create
   the **Gmail draft** follow-up (never send; thread it if the channel is email). On "no", they stay only
   in chat + on the account page. Non-BDRs never reach this step.
7. **Report.** Chat recap: the two scores + verdict, top dropped conversations (with ages + temperature),
   lead/MQL count, closed-lost reasons, the notable external flags, what was written (or "chat only"), and
   — if revival ran — who was added to Conversation Hub and which drafts were created.
