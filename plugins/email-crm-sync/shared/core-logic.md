# email-crm-sync core logic

This file is the shared logic for all three email-sync commands (`email-sync`, `email-sync7`,
`email-sync30`). Each command's SKILL.md sets two parameters — `EMAIL_WINDOW_DAYS` (1, 7, or 30) and
`DEEP_RESOLUTION` (false for the 1-day command, true for 7/30) — then follows this file. It is the
email-channel sibling of `/convohub` (Jiminny cold calls) and `/linkedin-sync` (LinkedIn inbox), and
writes into the same shared Notion CRM so the three channels produce one consistent picture. Read
`~/code/convohub-plugin/shared/core-logic.md` and `~/code/linkedin-crm-sync/shared/core-logic.md`
for the routines this file reuses by reference (account resolution, page-section conventions,
meeting-booked handling).

Run fully autonomously. Make best-guess judgment calls on **subjective** matters (priority level from
reply tone, whether an inbound is a genuine prospect vs. someone selling to us, follow-up wording) and
note assumptions in the final summary rather than asking mid-run. This does **not** extend to
factual/identity matters — which Account or contact a record belongs to, whether two records are the
same company or person, whether a meeting was actually confirmed, whether an opportunity is really
closed. Never guess or invent those: if you can't confidently determine one, leave the property
unset/unchanged, skip the person if the whole decision hinges on it, and flag it explicitly in the
Step 8 summary.

**This plugin never sends email.** The Gmail connector in this environment exposes `create_draft` but
no send tool at all, so drafting-only is structural, not merely a policy. Every follow-up is a Gmail
draft for the rep to review and send themselves.

## Resolve the team's databases first (multi-team)

Before Step 0, get the four Notion data-source IDs for the **running rep's team** by following
`superside-sales-core`'s `shared/team-resolution.md` (rep → team via the bundled roster + the `💸 Sales`
page → the team's "Databases" page → the four DBs by name), and use those IDs everywhere below in place of
the fixed IDs. If `superside-sales-core` isn't installed or resolution can't complete, fall back to the
Money Moguls defaults below so the plugin still works for that team. Never write into a team you didn't
resolve; if resolution is ambiguous, stop and ask.

## Fixed IDs — Money Moguls defaults (fallback; prefer the resolved IDs above)

- Conversation Hub data source: `39d50ebb-bec0-8054-89c6-000b1f0a5914`
- All Accounts data source: `a5268c41-9288-4977-92df-f541a790a1bf` (database `69619f3b-0d0c-4aec-b9e9-cec5d8c6b326`)
- All Meetings data source: `30950ebb-bec0-80fc-81c8-000b4f854176`
- Monthly Scoreboard (Rep) data source: `30950ebb-bec0-8003-bb35-000b6b021eec`. Carries two per-rep
  fields this plugin reads/writes (both added 2026-07-27): **`Last email check`** (date — the daily
  window) and **`Email exclusions`** (text — this rep's learned sender-exclusion list, see Step 2a).
- Superside email convention: everyone's email is `{first_name}.{last_name}@superside.com`.

Do not hardcode any MCP server/connector-ID prefix in tool calls (e.g. `mcp__<uuid>__search_threads`)
— resolve tools by name, the exact connector ID differs per rep's own account:
- **Gmail:** `search_threads`, `get_thread`, `get_message`, `create_draft`, `list_drafts`, `list_labels`.
- **Notion:** `notion-query-data-sources`, `notion-fetch`, `notion-create-pages`, `notion-update-page`.
- **Salesforce:** `find`, `soqlQuery`, `getRelatedRecords`, `getUserInfo`.
- **Calendar:** `list_events`.
- **Jiminny:** `get_me` (rep resolution only).

Schema options and property names on all databases can and do change — the schema sections below are a
snapshot. If a select/status option or property name you need isn't here, re-fetch the data source's
live schema before assuming it doesn't exist or guessing at a name.

## Conversation Hub schema (relevant fields — re-fetch live before trusting names)

Name (title), Account (relation → All Accounts), Rep (relation → Monthly Scoreboard), Title (text),
**Connected** (multi_select: Cold call/LinkedIn/Email — the touchpoint-channel field; append "Email"),
Touch points (multi_select: Cold call/LinkedIn/Email — append "Email"), Chatting? (checkbox — genuine
back-and-forth only), Cold call connected? (checkbox), Connected on LinkedIn? (checkbox), Inbounded?
(checkbox — check for genuine inbound-sourced contacts), Last touched (date), **Suggested follow-up**
(date — carries a built-in Notion reminder, default 9am Europe/Belgrade on the date itself; this is the
due-date alert mechanism, no external task tool involved), Suggested channel next (select: Email/
LinkedIn/Call again — set "Email"), **Priority** (select: High/Medium/Low — set by reply intent, see
Step 6), **Quick notes** (text — the 1-2 sentence recap, i.e. the "notes property"), Disposition
(select, fixed list — see Step 6's mapping), First follow-up sent (checkbox), Second follow-up sent
(checkbox), Suggested draft made (checkbox), Had meeting? (checkbox — see the Step 3 override), Nutrure?
(select, fixed list), Campaign (multi_select, open-ended), Date of last campaign (date), URL Salesforce
(url), URL LinkedIn (url), URL Jiminny (url).

## All Accounts schema (relevant fields)

Account Name (title), Rep (relation → Monthly Scoreboard), Parent (text — parent company name; self-name
if the account is itself a parent), sister/child (text, comma-joined child/sister brand names, lives on
the parent's page), Priority Type (multi_select: "Growth","Inbound","MQA","MQL","A","B","BDM Selected"),
Meetings (relation → All Meetings, auto-populated — never set directly).

## All Meetings schema (relevant fields)

Company (title — the Account Name, not the person), Rep (relation → Monthly Scoreboard), Account
(relation → All Accounts — every row must have this), Name (relation → Conversation Hub — the person;
requires a Hub page to exist first), Persona/Title (select, closed vocabulary — set only on a clear
match, never force a mismatch), Status (status — to_do: Rejected/No-show/Canceled; in_progress:
Placeholder sent/Waiting on time-date/Invite not accepted yet/Reschedule/Booked; complete: Completed),
**Channel** (select: Email/LinkedIn/Cold Call/Inbound priority/Inbound none priority — this plugin sets
**Email**), Date of Booking (date), Date of Meeting (date), LinkedIn of Prospect (url), SF Opp URL (url),
Notes (text — short gist).

---

## Step 0 — Resolve the rep

Gmail exposes no "who am I" tool, so resolve identity another way: call Jiminny `get_me` (name, email,
member_id) — or Salesforce `getUserInfo` if Jiminny isn't connected — and match the result against the
Monthly Scoreboard `Name` column (same fuzzy-match care as everywhere else — exact or an obvious
variant like a nickname). If the match is clean, proceed silently using that person's Monthly Scoreboard
page URL as the `Rep` relation value everywhere below. If ambiguous or no match, ask the person running
the command to confirm/pick their name from the Monthly Scoreboard list — this is the one point in the
whole run allowed to interrupt. Do not cache across runs; re-resolve every time (this runs on multiple
people's machines).

Capture the rep's own `{first}.{last}@superside.com` email — **"external" means any sender whose domain
is not `superside.com`**; only external senders are ever registered.

Read **`Last email check`** (the daily window) and **`Email exclusions`** (this rep's learned
sender-exclusion list — see Step 2a) from the resolved row.
> Both properties exist on the live Monthly Scoreboard schema (added 2026-07-27, alongside the siblings
> `Last linkedin check` / `Last Convohub Run`). Still re-fetch the live schema each run — if either is
> ever missing, fall back gracefully (fixed-window when `Last email check` is gone; shared-baseline-only
> when `Email exclusions` is gone) and flag it in the Step 8 summary rather than failing the run.

**Window:**
- `email-sync` (daily, `EMAIL_WINDOW_DAYS`=1): use the `Last email check` value as the window start
  through now. If empty/absent, fall back to the start of the previous working day — if today is Monday,
  the preceding Friday at 00:00 local time (skip the weekend); any other weekday, yesterday at 00:00.
- `email-sync7` / `email-sync30`: `today − EMAIL_WINDOW_DAYS` at 00:00 local time through now.

## Step 1 — Conversation Hub + All Meetings reconciliation (runs FIRST, every session, all three commands)

Before pulling any new mail, establish where existing email threads and email-booked meetings already
sit in the funnel — this is the mandatory session-opening sweep. It is independent of the Step 2 window.

### 1a. All Meetings cleanup (email channel)
For every All Meetings row where Rep = the resolved rep and `Channel` = "Email" (plus any row with no
Channel set that this plugin previously touched or that is clearly an email booking) whose `Date of
Meeting` has passed without the Status already being a resolved terminal state:
1. If the row's `Name` relation points to a Hub page with `Had meeting?` checked → skip the row entirely,
   no further action anywhere for that person.
2. Verify what actually happened via Calendar (was the event on the calendar, did it occur) and/or
   Salesforce (opportunity activity/notes). Never assume from the row alone.
3. Set `Status`: **Completed** if it happened; otherwise **Canceled** (prospect emailed to cancel, no
   reschedule, no stated disinterest), **No-show** (time passed, no word at all), or **Rejected** (they
   declined the invite outright). "User did not reply" to a rep's own nudge, with silence from the
   prospect, is not a resolution — leave it unresolved and flag it.
4. If the row is missing its person, identify and link/create the Hub page (same method as `/convohub`
   Step 1b/5 — never leave `Rep` blank on the created page).
5. If it happened, check `Had meeting?` on the Hub page and stop for that person. If it didn't, this
   person may still be eligible for re-engagement — but only through the Step 3 gate (a Closed-Lost opp),
   not automatically.

### 1b. Conversation Hub email-thread funnel scan
Scan Conversation Hub rows for this rep that carry an "Email" touch (in `Connected` or `Touch points`)
and read their current funnel state: `First follow-up sent` / `Second follow-up sent`, `Had meeting?`,
`Disposition`, and — by checking Gmail for the thread — whether the prospect has actually replied since
the rep's last touch (this is how "user did not reply" cases surface). Carry this state into Step 7's
cadence. Do not re-draft anything here; this substep only builds the funnel picture.

### 2a. Build the sender-exclusion list (shared baseline + per-BDR, learned on first run)

The inbox is mostly automated noise (tool notifications, newsletters, calendar/meeting bots). Exclude it
at the query level so the run only reasons over real human mail. The effective exclusion set is the union
of two lists:

**Shared baseline — hardcoded here, excluded for every BDR** (the Superside-wide tooling/notification
senders every rep receives). Apply as `-from:` negations in the Gmail query:
`tryapollo.io`, `mail.notion.so`, `notifications.hubspot.com`, `chilipiper.com`, `jiminny.com`,
`zoom.us`, `reachdesk.com`, `email.businessinsider.com`, and the bounce sender
`mailer-daemon@googlemail.com`.
> ⚠️ **Never exclude `gmail.com`/`googlemail.com` as a domain** — real prospects email from Gmail (a live
> reply from `jasmin.kaur.uk@gmail.com` was seen in testing). Only the specific `mailer-daemon@…` bounce
> address is excluded. Excluding `chilipiper.com`/`jiminny.com`/`zoom.us` here is safe: Step 1 gets
> meeting status from All Meetings + Calendar + Salesforce, never from these emails.

**Per-BDR learned list — stored in the rep's Monthly Scoreboard `Email exclusions`** (comma-joined sender
domains/addresses, unique to each rep; e.g. a personal newsletter like `news.bloomberg.com`):
- **First run for a rep** (`Email exclusions` is empty): after fetching the window with only the shared
  baseline applied, cluster the senders that are automated/newsletter/non-prospect (bulk/no-reply
  patterns, `unsubscribe` footers, newsletter/digest senders, notification bots) and are **not** already
  on the shared baseline. Present that proposed personal-exclusion list to the rep once for confirmation
  — this is an allowed interruption, like `/convohub`'s AE ask — then write the confirmed comma-joined
  list to `Email exclusions`. Never add a sender that looks like a real prospect/person.
- **Subsequent runs**: read `Email exclusions` in Step 0, union it with the shared baseline, apply all as
  `-from:` negations, silently. When a brand-new automated/newsletter sender appears that's on neither
  list, don't drop it silently — flag it in the Step 8 summary and append it to `Email exclusions` once
  the rep confirms (or on an obvious bulk/no-reply signal), so the list keeps learning.

### 2b. Pull replies + genuine inbound

Search the rep's inbox for inbound external mail in the window: `in:inbox` + the window
(`after:YYYY/MM/DD` for the exact daily start, else `newer_than:Nd`) + `-in:sent` + every `-from:` from
the 2a exclusion set, and lean toward excluding obvious noise categories (`-category:promotions
-category:social`) without discarding genuine business mail. Page through results; if the window is
large, note any truncation in the Step 8 report rather than silently dropping overflow. For each thread,
`get_thread` (FULL) and read it.

For the latest inbound **external** (non-superside.com) message in each thread, classify it:
- **REPLY** — the message is a reply to our outreach. Signals: subject starts `Re:`, an `In-Reply-To`/
  `References` header is present, or the thread contains a prior outbound from us/Apollo. Because Apollo
  outreach ships from other sending domains, the original sent message is often **absent from this
  Gmail** — so the `Re:`/header signal is what catches these; don't require a matching sent message.
- **GENUINE INBOUND** — a first-contact message from an external person who is plausibly a **prospect
  interested in Superside's creative/design/AI services** (a potential buyer). Register these too.
- **SELLER / NOISE — skip and flag.** Anyone selling *to* us or otherwise not a prospect: vendor/agency
  pitches, recruiters, newsletters and marketing blasts, no-reply/automated senders, generic cold sales.
  When a first-contact inbound is clearly someone selling to us, skip it and note it in the summary.
  When genuinely unsure whether a first-contact inbound is a real prospect, err toward skipping and flag
  it for the rep rather than registering a seller.
- Drop internal (superside.com) senders entirely — never registered.

## Step 3 — Eligibility gate (per external person)

The primary gate is: **is this person already tracked in the Conversation Hub?** This plugin surfaces
repliers who are *not already being handled* — a genuinely new/untracked contact — plus one revival
case. Match the candidate to an existing Conversation Hub page by Name + Account (the same dedup match
Step 6 uses — match on the real person, not a fuzzy name-only hit; a different person with a similar
name is not a match).

**A. Already has a Conversation Hub page** → they're already being tracked/worked, so **SKIP** — with a
single exception: if their Salesforce Opportunity is **Closed Lost** (`IsClosed = true`, `IsWon =
false`), they're a dead deal that just resurfaced → **ELIGIBLE, re-engage**. Any other opp state (open,
Closed Won, or none) → SKIP. Flag every "already in Hub, skipping" in the Step 8 summary with the reason
(e.g. "active — open opp", "booked meeting in progress").

**B. No Conversation Hub page yet** (a genuinely new contact — the main case) → decide by whether
they've had an *actual* meeting. "Had an actual meeting" means one genuinely took place — an All Meetings
row for them is `Status = Completed`, or a completed calendar event with them exists. A meeting that was
merely *booked* and then canceled, no-showed, or is being rescheduled does **not** count (an upcoming
confirmed meeting is Branch A / active-booking territory in Step 6, not a fresh registration).
- **No actual meeting →** ELIGIBLE. Register them (Step 4 onward) **regardless of opportunity state** —
  an open opp never disqualifies a new no-meeting contact. On the contrary: an **open opp with no meeting
  date set and no follow-up made yet** makes the follow-up *more* urgent, not less — surface it that way
  (a nearer-term `Suggested follow-up` in Step 6, flagged in Step 8). Read the opp only to judge urgency,
  never to gate.
- **Had an actual completed meeting →** query the opp (`find`/`soqlQuery`: `IsClosed`, `StageName`,
  `IsWon`; disambiguate by contact + Account, never name alone): Closed Lost → ELIGIBLE (re-engage);
  open → SKIP (active pipeline, AE owns it); Closed Won → SKIP (customer); none → SKIP; can't determine →
  SKIP + flag, never guess.

> ⚠️ **The Closed-Lost revival (case A's exception) overrides the sibling rule "skip if already
> tracked / `Had meeting?` is checked."** `/convohub` and `/linkedin-sync` skip anyone already
> booked/tracked unconditionally; this plugin deliberately re-engages an already-in-Hub contact **when
> and only when** their opp is Closed Lost. Everyone else already in the Hub is left alone — they're being
> handled. Do not "fix" this back to the sibling behavior. (Helen-in-Aon is the canonical skip: she has a
> Hub page and an open opp, so case A skips her even though her booked meeting was canceled — she's
> in-flight, not new.)

## Step 4 — Resolve the account

Derive the company from the external sender's email domain and resolve it to All Accounts using
`/convohub`'s **Step 6** match/create/parent-child routine verbatim (match on exact or obvious variant —
punctuation, "and" vs "&", Inc./Incorporated — cross-check the Salesforce Account name as source of
truth when unsure; detect parent/subsidiary via naming/domain/quick web search and record it in
`Parent`/`sister/child`). Unlike `/linkedin-sync` (which never creates accounts), genuine inbound can
come from a company not yet tracked, so **creating** a new All Accounts row here is allowed, exactly as
`/convohub` does. Never leave a registered person without an Account relation. When you create a new All
Accounts row, follow the **Organic backfill** in `superside-sales-core`'s `shared/salesforce-import.md`:
set `Priority Type` from the account's Salesforce `Priority_Account__c` (mapped A→A, B→B, C→BDM Selected,
BU tiers→BU Expension only for eligible reps) plus descriptive-only fields (Account URL, Size, Last
Prospected, Parent) — never buying-signal/lead fields. If no confident SF match, leave `Priority Type`
unset and flag; never guess a tier.

## Step 4.5 — Salesforce contact lookup

First check whether it's needed: if the person's existing Hub page already has a non-empty `URL
Salesforce`, skip this. Otherwise SOSL-search by name — `find` with
`FIND {"Full Name"} IN NAME FIELDS RETURNING Contact(Id, Name, Email, Title, AccountId), Lead(Id, Name,
Email, Company, Title)` — and disambiguate by matching `Title` + Account against what Step 4 confirmed,
never name alone (common names return false positives). Prefer a Contact over a Lead. Found → set
`URL Salesforce` = `https://superside.my.salesforce.com/<recordId>`. Not found → leave empty and flag by
name in the summary. Real duplicate records → use one, flag the duplication.

## Step 5 — Summarize the thread + detect meeting-booked

`get_thread` (FULL) and write a short, plain summary of what's actually being discussed — analyst-brief
tone, no fluff, no em dashes, never fabricated. While reading, determine whether a meeting is already
booked: only treat it as booked when the thread shows an explicit date/time **both sides confirmed** (not
a one-sided proposal or a vague "let's find a time"). Ambiguous → treat as not-booked and flag. This
routes Step 6's branch.

## Step 6 — Write the CRM

**Branch A — a meeting is booked over email** (mirrors `/linkedin-sync` Branch A, with `Channel` =
"Email"):
1. Match an existing Hub page by Name + Account (never duplicate). If none, create a minimal one (Name,
   Rep, Title, Account) — All Meetings' `Name` relation needs a Hub page to point at.
2. Cross-check the rep's calendar before trusting the email-parsed date: `list_events` for the person's
   name/company. A matching calendar event's date/time is the source of truth (email text can be stale
   after a reschedule). No matching event → fall back to the email-parsed date/time but flag "not
   confirmed on calendar."
3. Query All Meetings via its `Name` relation to that Hub page; determine the final date, then:
   - No row → create: `Company` = the Account's name, `Rep`, `Account`, `Name` → the Hub page, `Channel`
     = **"Email"** (booked over email, always Email in this branch), `Date of Meeting` = final date,
     `Date of Booking` = today.
   - Row, same date → no change, but still run the passed-meeting check.
   - Row, different date (rescheduled) → update the existing row (never create a second): `Date of
     Meeting` = new date, `Status` = **Reschedule**, `Channel` = "Email".
   - Status finalization: if the final `Date of Meeting` is in the past → `Status` = **Completed**
     (overrides Booked/Reschedule). Otherwise Booked (new) or Reschedule (moved).
   - `Persona/Title`: best-fit to the closest existing option, never a genuine mismatch. Set on every
     create/update.
4. Do not draft a follow-up in this branch — the All Meetings row is the record of this touchpoint. Still
   write the Account-page `## Conversations` entry (below) so the account history reflects it.

**Branch B — no meeting booked (new reply/inbound, or Closed-Lost re-engagement):** upsert the
Conversation Hub page (match by Name + Account, never duplicate):
1. Set/confirm `Name`, `Rep` (the resolved rep — never blank), `Title`, `Account` (from Step 4).
2. Append **"Email"** to `Connected` and to `Touch points` (no duplicates). Check `Chatting?` only on
   genuine back-and-forth (the person actually replied — true for every REPLY, and for an inbound that's
   become a two-way thread). Check `Inbounded?` for a genuine-inbound-sourced contact.
3. `Last touched` = the actual date of the person's last message (not necessarily today).
4. **`Priority` — reply-intent only** (ignore account size):
   - **High** — clear buying intent or a meeting request ("can we set up a call", "send me pricing",
     "we're evaluating vendors", an explicit yes).
   - **Medium** — engaged but exploratory: a substantive question, "tell me more", neutral interest.
   - **Low** — soft-no / "not right now" / "reach out next quarter", out-of-office/auto-reply, or a
     Closed-Lost re-engagement with no fresh buying signal.
5. `Suggested channel next` = "Email". `Suggested follow-up` = a genuine near-term date judged from the
   thread's momentum (this fires the Notion reminder — the alert mechanism; it must be a real date, not
   a placeholder). Pull this date **sooner** for the urgent case flagged in Step 3 — a no-meeting
   contact with an open opp, no meeting date set, and no follow-up made yet — and note the urgency in the
   Step 8 summary.
6. `Quick notes` = 1-2 sentence recap (**the notes property**). Set `Disposition` only on an unambiguous
   signal, choosing the closest existing picklist option — never invent one: `Chatting` for engaged
   back-and-forth with nothing concrete decided; `CP not now` / `CN not now` for a soft deferral; `CP
   send email` / `CN send email` when email is the agreed next channel; `Meeting booked` only in Branch
   A. The picklist is cold-call-shaped (`CP/CN…`, `MB…`) so email intents map imperfectly — pick the
   nearest real option and flag any genuine gap in the summary rather than forcing a wrong one.
7. Set `URL Salesforce` from Step 4.5 if newly found.
8. Draft the follow-up (see below), then check `Suggested draft made`.

**Page content format (both the person's Hub page and their Account page, both branches)** — identical
convention to `/linkedin-sync`, prepend-newest, never overwrite, one heading only:

```
## Conversations
**Email:** <one-line summary — on an Account page with more than one matched person, one bolded
"Email:" line per person, name and title included>

## Follow-up
<the suggested follow-up date, and the drafted message from step 8 — for Branch A (meeting booked), or
when a fresh draft isn't warranted (e.g. the page already carries a current follow-up plan from another
source), write a short line instead, e.g. "meeting already booked, see All Meetings">
```
Placement rules (verbatim from the siblings): if a heading containing "Conversations" already exists
(case-insensitive — includes `/convohub`'s and `/linkedin-sync`'s), insert this block at the top of that
section rather than creating a second one; if the page has other content but no "Conversations" heading,
prepend the whole block to the very top, never overwriting; if blank, the block is the whole content; if
a `## Follow-up` heading already exists, add the new line into it. Never duplicate either heading.

The Account page also gets the newest-first log entry used by the siblings:
`**{Name of person}** — {date} — {Disposition}: {short summary}` under its single `## Conversations`
heading.

**Drafting the follow-up (Gmail draft, never sent):** create it with `create_draft`, passing
`replyToMessageId` = the person's inbound message ID so it threads correctly, addressed to their real
email. Short, warm, casual — 2 to 4 sentences; grounded only in what the thread actually discussed, never
fabricated; ask a genuine question where one fits, or offer something of concrete value — when offering,
pull the specific name from Conversation Hub's `Campaign` property options (the real vocabulary of
Superside's named events/content), never an invented title; if nothing fits, ask a question instead.
**Idempotency:** skip creating a draft if the relevant "sent" checkbox for that stage is already checked,
or a not-yet-sent draft for that stage is already on file (check `list_drafts`).

## Step 7 — Follow-up cadence + nurture hand-off

Mirror `/convohub` Step 7, scoped to email. The daily `email-sync` runs the light version (only advance
rows this run just touched, or with an obvious state change); `email-sync7`/`email-sync30`
(`DEEP_RESOLUTION` = true) run the full cadence scan across all of this rep's email-touch rows.

Skip any row where `Had meeting?` is checked (except a Closed-Lost re-engagement the Step 3 gate already
admitted). For rows with `Suggested draft made` checked but `First follow-up sent` not yet checked,
verify whether it actually went out — primary signal is Salesforce activity history filtered to the
resolved rep; cross-check Gmail (was the draft sent / is it still sitting in drafts) when inconclusive —
and check `First follow-up sent` if so. Once `First follow-up sent` is checked and 7+ days have passed
with no reply, draft the backup follow-up (Step 6's draft rules), keep `Suggested channel next` = "Email"
unless the thread agreed otherwise, and check `Second follow-up sent` once confirmed sent. Once
`Second follow-up sent` is checked, assign `Nutrure?` from its fixed existing option list only (never
create a new one) — match the prospect's location against the pre-set cities (Atlanta, Toronto, Chicago,
NYC, London, SF) for a Superside Connect/Dinner in the next 1-2 months (check superside.com/events), else
"webinar" if one's coming in the next month, else "BDR-led nutrure"; recheck anyone on "BDR-led nutrure"
each run and upgrade if a matching event now exists.

## Step 8 — Report back + stamp

End with a concise plain-text summary:
- how many inbound threads were pulled in the window; how many were skipped and why, broken out:
  excluded senders (baseline + per-BDR), internal senders, sellers/noise inbound, and Step-3
  already-in-Hub skips (with the reason each — "active, open opp" / "booked meeting in progress" /
  "Closed Won") — list the already-in-Hub open-opp ones by name so the rep knows they're in-flight.
- exclusion-list learning: on a first run, the personal senders proposed and written to `Email
  exclusions`; on later runs, any new automated/newsletter sender flagged for addition.
- any urgent case flagged per Step 3 (new no-meeting contact with an open opp, no meeting date, no
  follow-up yet) and the nearer-term follow-up date set for it.
- for every registered person: name + company (never a bare name), a one-line summary, which branch
  applied (meeting logged to All Meetings vs. Conversation Hub updated + follow-up drafted), the Priority
  set and why (reply intent), and the drafted follow-up text so the rep can see it before sending.
- every change made to an *existing* row (checkbox flips, Status/Channel changes in All Meetings,
  Disposition/Priority changes, Nurture upgrades) with a one-line reason.
- from Step 1: any email-channel meeting whose status was resolved (and to what), any "user did not
  reply" case surfaced, any row missing a person that was linked/created.
- any person with no Salesforce match, any Salesforce duplicate-record situation, any Disposition-gap
  flagged, and (first run) whether `Last email check` was missing from Monthly Scoreboard.
Do not repeat full message bodies from the original threads — names, accounts, and the gist only.

Before finishing, set **`Last email check`** on the resolved rep's Monthly Scoreboard row to the current
time (every command does this). If the property doesn't exist yet, note that in the summary instead of
failing the run.
