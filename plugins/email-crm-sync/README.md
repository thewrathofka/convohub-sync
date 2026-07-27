# email-crm-sync

Email-reply review for Superside BDRs — the email-channel sibling of `/convohub` (Jiminny cold calls)
and `/linkedin-sync` (LinkedIn inbox). Every run starts by reconciling where existing email threads and
email-booked meetings already sit in the shared Notion "Conversation Hub" and "All Meetings" databases,
then reads the rep's Gmail for external people who replied to outreach — or sent a genuine inbound —
registers only the ones worth acting on, logs them into Conversation Hub / All Accounts / All Meetings
with channel Email, and drafts (never sends) a Gmail follow-up.

Works for whichever rep invokes it — it resolves the current rep automatically from Jiminny (or
Salesforce) and the Monthly Scoreboard rep list at the start of every run.

## Commands

- **`/email-sync`** — replies + genuine inbound since the last run (reads/writes `Last email check` on
  the rep's Monthly Scoreboard row). Fast daily check. Runs the light follow-up cadence.
- **`/email-sync7`** — last 7 days. Full follow-up cadence and nurture hand-off.
- **`/email-sync30`** — last 30 days. Deep monthly sweep to catch anything that fell through the cracks,
  same logic as email-sync7 with a wider window.

All three share one logic file (`shared/core-logic.md`) so they can't drift out of sync — the individual
SKILL.md files just set the window (`EMAIL_WINDOW_DAYS`) and whether the full follow-up cadence runs
(`DEEP_RESOLUTION`).

## Who gets registered

Only **external** senders (any domain other than `superside.com`), gated first on whether they're
already tracked in the Conversation Hub:
- **Not yet in the Conversation Hub** (a genuinely new/untracked replier or inbound) → registered. If
  such a person has already had an *actual completed* meeting, that's narrowed to the Closed-Lost case.
- **Already in the Conversation Hub** → left alone (they're already being handled) — **except** when
  their Salesforce opp is **Closed Lost**, a dead deal that just resurfaced, which gets re-engaged.

An open opp never disqualifies a brand-new no-meeting replier — in fact an open opp with no meeting
booked and no follow-up yet makes their follow-up *more* urgent. The Closed-Lost revival is the one place
the plugin intentionally acts on a contact already in the Hub / with `Had meeting?` checked — the
opposite of what `/convohub` and `/linkedin-sync` do. (Example: someone already in the Hub with an open,
active opp — a reschedule in progress — is skipped as in-flight; a brand-new replier with no Hub page is
registered.)

## What counts as something to act on

- **Replies** to our outreach — including outreach sent via Apollo from a *different* domain, so the
  reply lands in Gmail with no matching sent message. Detected from `Re:`/`In-Reply-To` signals, not by
  finding our original.
- **Genuine inbound** — a real first-contact prospect interested in Superside's creative/design/AI work.
- **Not** anyone selling *to* us — vendors, agencies pitching, recruiters, newsletters, no-reply blasts,
  cold sales. Those are skipped and flagged.

## Inbox noise filtering (shared baseline + per-BDR, learned)

The inbox is mostly automated senders. Two exclusion lists keep the run focused on real human mail:
- **Shared baseline** — hardcoded, excluded for every BDR: the Superside-wide tooling/notification
  senders everyone gets (Apollo, Notion, HubSpot, Chili Piper, Jiminny, Zoom, Reachdesk, Business
  Insider, and the Gmail bounce address). Real Gmail/Google senders are **never** blanket-excluded — only
  the specific `mailer-daemon@` bounce address.
- **Per-BDR learned list** — stored on each rep's Monthly Scoreboard `Email exclusions`. On a rep's
  **first run** the plugin scans their inbox, proposes the personal automated/newsletter senders it finds
  (e.g. a Bloomberg subscription), confirms once, and remembers them. Later runs apply the list silently
  and flag any new automated sender to add — so each rep's filter sharpens over time without affecting
  anyone else's.

## What each registered person gets

- A Conversation Hub page (created or updated, never duplicated): channel **Email** added to `Connected`
  and `Touch points`, **Priority** set from the reply's intent (interested/meeting-request → High,
  engaged question/neutral → Medium, soft-no/OOO/auto-reply → Low), a 1-2 sentence recap in `Quick
  notes`, a `Suggested follow-up` date (which fires Notion's own reminder — the alert), and a conversation
  summary in a `## Conversations` section on the page.
- The same summary logged on the **account's** page under its single `## Conversations` heading.
- If the reply confirms a meeting, an **All Meetings** row instead (Channel = Email, calendar-verified
  date, Status Booked/Reschedule/Completed).
- A **Gmail draft** reply, threaded to their message, for the rep to review and send themselves.

## Requirements

Each rep needs their own Gmail, Notion (with access to the Money Moguls workspace — Conversation Hub,
All Accounts, All Meetings, Monthly Scoreboard), Salesforce, Calendar, and Jiminny connected in whichever
surface they run this from (Cowork or Claude Code). The plugin references tools by name, not by connector
ID, so it works regardless of whose connector instance is behind them. No local MCP server and no extra
permission config are needed (unlike `/linkedin-sync`, whose LinkedIn server runs locally).

> Setup note: this plugin uses two Monthly Scoreboard fields — `Last email check` (date, the daily
> window) and `Email exclusions` (text, the per-BDR noise list). Both were added 2026-07-27. The run
> re-fetches the live schema each time and degrades gracefully (fixed window / shared-baseline-only) if
> either is ever missing.

## What it never does

**Never sends an email** — everything is a Gmail draft for the rep to review and send themselves. This is
structural, not just policy: the Gmail connector exposes `create_draft` but has no send tool at all.
Never touches a contact with an **open** or **Closed Won** opp. Never invents Disposition or Nurture
options beyond the fixed lists. Never guesses on factual matters — which Account/contact a record belongs
to, whether two records match, or whether an opp is really closed — if it can't confidently tell, it
skips and flags instead of writing a guess. Apollo task creation (which `/convohub` does) is intentionally
left out of this plugin for now; it could be added later mirroring `/convohub`'s Step 8.
