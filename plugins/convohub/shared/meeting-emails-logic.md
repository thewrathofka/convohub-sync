# Meeting emails logic

Shared logic for drafting pre-meeting emails on All Meetings rows. Used two ways: automatically inside every convohub/convohub7/convohub30 run (via core-logic.md Step 1e), and standalone via the `meeting-emails` skill. Never invent data you don't have; skip a row rather than guess.

Takes one parameter, `INTERACTIVE_SEND_MODE` (boolean, default false):
- **convohub/convohub7/convohub30 invocations always pass `false`** — no exceptions, this is an autonomous run with no one present to approve a send mid-flight.
- **The standalone `meeting-emails` skill** sets this per its own "draft vs. write" rule (see that SKILL.md) — `false` when the rep said "draft" or didn't specify, `true` when they said "write".

When `false`: draft only, silently, never send (see "Both cases — finishing up" below). When `true`: compose the email, show it in chat, get explicit approval, then actually send it — no silent Gmail draft in this mode.

## Scope of the sweep

Every All Meetings row for the resolved rep where:
- `Date of Meeting` is set and in the future,
- `Status` is not a terminal state (not "Rejected", "Canceled", or "No-show"),
- and neither `Pre-meeting email drafted` nor `Pre-meeting email sent` is checked.

## Timing gate

Skip any row where the meeting is less than 1 hour away, or already in the past. No draft, no flag, no checkbox change for rows that fail this gate — re-check them on a future run instead.

## Classify inbound vs. outbound

`Channel` = "Inbound priority" or "Inbound none priority" → **inbound**. `Channel` = "Email", "LinkedIn", or "Cold Call" → **outbound**. `Channel` blank → **skip this row and flag it** in the summary; don't guess, and never independently query Salesforce here — the single Salesforce check already happened once, in core-logic.md Step 1c (see that file's reordered Step 1e, which only invokes this file after Channel is resolved for every row in the window).

## Resolve names

- `{Name-of-prospect}` — All Meetings row's `Name` relation → the Conversation Hub page's `Name`. If `Name` is empty, skip this row rather than guess who the prospect is.
- `{Name-of-BDR}` — the resolved rep (core-logic.md Step 0).
- `{Name-of-AE-of-that-BDR}` — the resolved rep's AE (core-logic.md Step 0's AE resolution).

## Outbound → voucher email

Determine the meal word from the prospect's likely local time at the meeting's start:
1. Infer the prospect's timezone from their company's likely location — same web-search pattern as core-logic.md Step 6 (check the company's own site/LinkedIn for HQ or office location; don't guess without at least one piece of evidence).
2. Convert the meeting's start time to that local time. Map to a word using these buckets:
   - 6:00–10:29 → **Breakfast**
   - 10:30–11:59 → **Coffee**
   - 12:00–13:59 → **Lunch**
   - 14:00–17:00 → **Coffee**
   - Outside this range, or timezone couldn't be confidently determined → **Coffee** (default)

Draft this exact template, filling in the placeholders:

```
Hi {Name-of-prospect}!

{Name-of-AE-of-that-BDR} and I look forward to meeting you shortly.

{Breakfast/Lunch/Coffee} is on us this session. You can use **this voucher link**.

Can't wait to learn more about your creative process.

Best,
{Name-of-BDR}
```

"this voucher link" must be bolded and hyperlinked to exactly: `https://redeem.reachdesk.com/gifts/42a0661e-609b-4521-b5a3-9fcaf81ac4cd`

## Inbound → context-request email

Draft this exact template, filling in the placeholders:

```
Hey {Name-of-prospect},

Thank you for booking a meeting with us.

We are excited to meet you and learn a little bit more about your operation, challenges and how we could help.

To come as best prepared as we can, would you mind sharing a little bit more context on what it is about Superside that you're interested in?

Is it around creative support in general? Perhaps AI consulting? Let me know what those challenges you're facing and trying to address and we'll come ready to address those.

Looking forward to your comments.

Best,
{Name-of-BDR}
```

## Both cases — finishing up

**If `INTERACTIVE_SEND_MODE` is false (default; always false for convohub/convohub7/convohub30):**
1. Create the Gmail draft (never send) addressed to the prospect's real email.
2. Check `Pre-meeting email drafted` on the All Meetings row.
3. Add a line to the run summary for each row processed: company, prospect name, which template was used (voucher/outbound or context-request/inbound), and — for outbound — which meal word was picked and why (inferred timezone + local time, or "defaulted to Coffee, timezone unclear").

**If `INTERACTIVE_SEND_MODE` is true (standalone `meeting-emails` skill only, rep said "write"):**
1. Compose the email (same templates, same placeholder resolution and meal-word logic as above) and show the full text in chat, addressed to the prospect's real email — including which template/meal word was used and why.
2. Ask the rep to approve it as-is, request edits, or skip it. If they request edits, revise and re-show it before proceeding — don't send anything not yet approved.
3. Only once explicitly approved: actually send the email via Gmail (a real send, not a draft) to the prospect's real email.
4. Check `Pre-meeting email sent` (not `Pre-meeting email drafted`) on the All Meetings row, since it was genuinely sent.
5. If the rep skips or never approves it, leave both checkboxes unchecked and note in the summary that it's still pending.
