---
name: meeting-emails
description: Draft or write pre-meeting emails for a rep's upcoming booked meetings — a voucher (Breakfast/Lunch/Coffee) email for outbound-booked meetings, or a context-request email for meetings booked through the inbound scheduler form. Use when a rep asks about a pre-meeting email, pre meeting email, meeting confirmation email, meeting voucher email, or pre-meeting emails for booked meetings — e.g. "draft my pre-meeting emails", "write my pre-meeting email for Acme", "send meeting vouchers", "write the meeting email for tomorrow's call".
---

# meeting-emails

Resolve the rep and their AE exactly as Step 0 describes in `${CLAUDE_PLUGIN_ROOT}/shared/core-logic.md`, then run the workflow in `${CLAUDE_PLUGIN_ROOT}/shared/meeting-emails-logic.md` against every qualifying All Meetings row for that rep.

This is a full standing sweep when run standalone — not scoped to any calendar window. Read both files in full before doing anything else in this run; schema options and property names can change.

## Draft vs. write — standalone use only

This distinction only applies when this skill is invoked on its own (never when it's running automatically inside a convohub/convohub7/convohub30 run — those always use the silent draft-only path, `INTERACTIVE_SEND_MODE` = false, no exceptions).

Look at the phrasing the rep actually used to invoke this skill:
- If they said **"draft"** (or didn't specify a verb at all) — `INTERACTIVE_SEND_MODE` = false. Default, silent Gmail-draft behavior, exactly as `meeting-emails-logic.md` describes.
- If they said **"write"** — `INTERACTIVE_SEND_MODE` = true. Pass this through to `meeting-emails-logic.md`, which composes the email and shows it in chat for approval before actually sending it (not a Gmail draft in this case — a real send, only after the rep explicitly approves it in chat).

Pass `INTERACTIVE_SEND_MODE` into the shared logic file for every row processed this run.
