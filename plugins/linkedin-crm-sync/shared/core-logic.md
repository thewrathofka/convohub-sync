# linkedin-sync core logic

This is the shared workflow for the `/linkedin-sync` command. Every rep on the team runs this against their own data, same pattern as `/convohub` — but unlike `/convohub`, each rep's LinkedIn login is bound to their own local browser profile (`~/.linkedin-mcp/profile/` on their own machine, created by their own one-time `uvx mcp-server-linkedin@latest --login`). There's no shared identity flag to read, so Step 0 resolves which rep is running by asking LinkedIn who's logged in, not by assuming a fixed person.

Run fully autonomously through Steps 1-7. Never guess on factual/identity matters — which Account a person belongs to, whether two company names are the same company, whether an existing Conversation Hub page is the same person, whether a meeting is actually confirmed. If you cannot confidently determine one, skip it and flag it explicitly in the Step 7 report rather than inventing a value. Step 8 (sending a message) is the one step that always requires an explicit, distinct go-ahead from the rep in chat — never chain it automatically after Steps 1-7 finish, even if their request implied the full pipeline including sends.

## Tool scope (enforced by `.claude/settings.json`, not just this doc)

Allowed: `get_inbox`, `get_conversation`, `search_conversations`, `get_person_profile`, `get_my_profile`, `send_message`, `close_session`.
Denied at the permission layer (cannot be called regardless of what this doc says): `search_people`, `connect_with_person`, `get_sidebar_profiles`, `search_companies`, `get_company_profile`, `get_company_posts`, `get_company_employees`, `search_jobs`, `get_job_details`, `get_feed`.

`get_person_profile` is only ever called on a `linkedin_username` extracted from `get_inbox`/`get_conversation` output — never on a name or company sourced any other way. This is what keeps profile access scoped to "people already in my inbox," per this plugin's original design, on top of the hard deny-list above.

## Resolve the team's databases first (multi-team)

Before Step 0, get the four Notion data-source IDs for the **running rep's team** by following
`superside-sales-core`'s `shared/team-resolution.md` (rep → team via the bundled roster + the `💸 Sales`
page → the team's "Databases" page → the four DBs by name), and use those IDs everywhere below in place of
the fixed IDs. If `superside-sales-core` isn't installed or resolution can't complete, fall back to the
Money Moguls defaults below. Never write into a team you didn't resolve; if ambiguous, stop and ask.

Note: unlike `/convohub` and `/email-sync`, this plugin intentionally does **not** create new All Accounts
rows (Step 4 keeps its tracked-account-only scoping — the LinkedIn inbox is too noisy to backfill
accounts from). Organic account creation is left to the cold-call and email channels.

## Fixed IDs — Money Moguls defaults (fallback; prefer the resolved IDs above)

- Conversation Hub data source: `39d50ebb-bec0-8054-89c6-000b1f0a5914`
- All Accounts data source: `a5268c41-9288-4977-92df-f541a790a1bf` (database `69619f3b-0d0c-4aec-b9e9-cec5d8c6b326`)
- All Meetings data source: `30950ebb-bec0-80fc-81c8-000b4f854176` — shared with `/convohub` and the account-research pipeline; Step 6 Branch A writes here.
- Monthly Scoreboard (Rep) data source: `30950ebb-bec0-8003-bb35-000b6b021eec` — carries `Last linkedin check` (date), set at the end of every run, same pattern as `/convohub`'s `Last Convohub Run` on the same data source.

Schema options and property names can change — re-fetch the data source's live schema if something referenced here isn't found, rather than assuming it's gone. (This doc previously called the sync-timestamp field `Last LinkedIn Sync` and referenced a `Touch points` property on Conversation Hub — neither ever existed in the live schema. Fixed 2026-07-23; if a live re-fetch ever disagrees with this doc again, trust the live schema and fix this doc, not the other way around.)

## Conversation Hub schema (relevant fields — see `/convohub`'s core-logic.md for the full list)

Name (title), Account (relation → All Accounts), Rep (relation → Monthly Scoreboard), Title (text), Connected on LinkedIn? (checkbox), **Connected** (multi_select: Cold call/LinkedIn/Email — this is the touchpoint-channel field; there is no "Touch points" property), Cold call connected? (checkbox), Chatting? (checkbox), Last touched (date), **Suggested follow-up** (date — carries a built-in Notion reminder, default 9am Europe/Belgrade on the date itself; this is the due-date alert mechanism for Step 6 Branch B, no external task tool involved), URL LinkedIn (url), URL Salesforce (url), Quick notes (text), Suggested draft made (checkbox — set when Step 6 Branch B writes a drafted follow-up message onto the page), Campaign (multi_select — closed vocabulary of Superside's actual named webinars/guides/dinners/reports, e.g. "2026Q3 - Webinar - Can Taste be Automated [July 22]"; use this list, never an invented one, when Step 6 Branch B's drafted message offers something of value), Had meeting? (checkbox — if checked, skip this person entirely, same rule `/convohub` follows).

## All Accounts schema (relevant fields)

Account Name (title), Rep (relation → Monthly Scoreboard), Parent (text), sister/child (text, comma-joined), Priority Type (multi_select).

## All Meetings schema (relevant fields)

Company (title — the Account Name, not the person's name), Rep (relation → Monthly Scoreboard), Account (relation → All Accounts), Name (relation → Conversation Hub — the person; requires a Conversation Hub page to exist first), Channel (select: Email/LinkedIn/Cold Call/Inbound priority/Inbound none priority), Status (status: Booked/Completed/Rejected/No-show/Canceled/Placeholder sent/Waiting on time-date/Invite not accepted yet/Reschedule), Date of Booking (date), Date of Meeting (date), Persona/Title (select, closed vocabulary — set only when a participant's title clearly maps to one of the existing options, never force a mismatch), LinkedIn of Prospect (url), Notes (text).

## Step 0 — Resolve rep

Every rep runs this against their own LinkedIn login, so resolve dynamically each run — never assume a fixed person. Call `get_my_profile()` to get the name on the currently-logged-in LinkedIn account, then match it against Monthly Scoreboard's `Name` column (same fuzzy-match care as Step 4 — exact or an obvious variant like a nickname). If no confident match, stop and ask the rep in chat which Monthly Scoreboard row is theirs rather than guessing — never write data under the wrong rep's name. Do not cache this across runs; re-resolve every time, since this plugin now runs on multiple people's machines.

Read `Last linkedin check` from the resolved row for Step 2's window.

## Step 1 — Pull inbox

Call `get_inbox(limit=50)`. The result is raw text per conversation row (per the tool's own docs — parse it yourself, there's no structured JSON). Extract each participant's display name and, where visible, a last-message date/snippet. If more than 50 active conversations exist, note the truncation explicitly in the Step 7 report — never silently drop the overflow.

## Step 2 — Load tracked accounts

Query All Accounts filtered to `Rep` = the resolved rep's row. Build a working list of tracked company names, including each account's `Parent`/`sister-child` aliases already recorded there — this is the set Step 4 matches against.

## Step 3 — Skip already-processed threads

If `Last linkedin check` is set and a conversation's last-message date is not after it, skip that conversation for this run (it was already covered last time) — unless the person's Conversation Hub page has `Had meeting?` unchecked and something looks unresolved; when genuinely unsure whether a thread has moved since last sync, err toward re-checking it rather than skipping.

## Step 4 — Match participants to tracked accounts

For each inbox participant not skipped by Step 3: call `get_person_profile(linkedin_username)` to read their current company and title. Match the company against Step 2's list using the same fuzzy-match approach `/convohub`'s core-logic.md Step 6 uses — exact or obvious variant (punctuation, "and" vs. "&", Inc./Incorporated), and check known Parent/sister-child aliases before concluding no match. No confident match → skip this person; they're not on a tracked account. Do not create a new All Accounts row here — unlike `/convohub`, this plugin never discovers brand-new companies, it only surfaces people already inside the tracked list.

## Step 4.5 — Check Salesforce

First check whether this is even necessary: look up the matched participant's existing Conversation Hub page (same Name + Account lookup Step 6 will do). If that page already has a non-empty `URL Salesforce`, skip the Salesforce search entirely for this person — never re-search or overwrite an existing value.

Otherwise, search Salesforce for them before writing to Conversation Hub. Use SOSL (`find` tool) by name — `FIND {"Full Name"} IN NAME FIELDS RETURNING Contact(Id, Name, Email, Title, AccountId), Lead(Id, Name, Email, Company, Title)` — common names return many false positives (seen firsthand with "Tom Smith"), so disambiguate by matching `Title` and the company/`AccountId` against what Step 4 already confirmed, not name alone. If both a Contact and a Lead match the same person, prefer the Contact. If more than one Contact plausibly matches (a real duplicate-record situation, not a wrong-person situation), don't guess which is canonical — use one, but flag the duplicate explicitly in the Step 7 report so the rep can dedup in Salesforce if needed.

If found: set `URL Salesforce` = `https://superside.my.salesforce.com/<recordId>`. If not found: leave `URL Salesforce` empty and flag it by name in the Step 7 report — never leave a "no Salesforce match" silently unreported.

## Step 5 — Summarize matched conversations

For each matched participant: call `get_conversation(linkedin_username=...)` for the full thread (use `index`/`thread_id` per the tool's own disambiguation logic if a name has multiple threads). Write a short, plain summary of what's actually being discussed — analyst-brief tone, no fluff, no em dashes. Never fabricate detail the thread doesn't contain.

While reading the thread, also determine whether a meeting is already booked with this person: only treat it as booked when the thread shows an explicit date/time both sides confirmed (not a vague "let's find a time" or a proposal only one side has agreed to). This feeds Step 6's branch. Never guess — if the thread is ambiguous about whether a proposed time was actually confirmed, treat it as not-yet-booked and flag the ambiguity in the Step 7 report rather than picking a side.

## Step 6 — Update the CRM

**Branch A — a meeting is already booked:**

1. Match an existing Conversation Hub page by Name + Account (same dedup rule as `/convohub`'s Step 1b/5). If none exists yet, create a minimal one (Name, Rep, Title, Account only) — All Meetings' `Name` relation needs a Conversation Hub page to point to, even in this branch.
2. Cross-check the rep's calendar before trusting the LinkedIn-parsed date: search calendar events for the person's name (and/or company). If a matching event exists, its date/time is the source of truth — LinkedIn message text can be stale (a thread that says "Thursday 2:30 PM" may have been rescheduled again after that message, and the calendar reflects the latest state; this happened for real in testing — a LinkedIn thread and an existing All Meetings row both said one date while the actual calendar event was a full week later). If no matching calendar event is found, fall back to the LinkedIn-parsed date/time but flag this explicitly in the Step 7 report as "not confirmed on calendar" so the rep can double-check.
3. Query All Meetings via its `Name` relation to that Conversation Hub page. Determine the final date first (calendar-confirmed from step 2, or LinkedIn-confirmed if no calendar match), then:
   - **No row exists**: create one. `Company` = the Account's name, `Rep` = the resolved rep's row, `Account`, `Name` → the Conversation Hub page, `Channel` = LinkedIn (the meeting was booked over LinkedIn, so this is always LinkedIn in this branch, regardless of what the calendar event itself might suggest), `Date of Meeting` = the final date, `Date of Booking` = today.
   - **A row exists with the same date**: no change needed, it's already accurate — but still apply the "meeting has passed" check below.
   - **A row exists with a different date (it's been rescheduled)**: never create a second row for the same meeting — update the existing one. Set `Date of Meeting` = the new final date and `Status` = **Reschedule** (not Booked) to reflect that this moved from its original date. Set `Channel` = LinkedIn (same rule as above, applies here too even if the row previously had a different channel recorded).
   - **Status finalization (applies after the above, create or update)**: if the final `Date of Meeting` is in the past relative to today, `Status` = **Completed**, overriding whatever the create/update logic above would otherwise set (Booked or Reschedule) — a meeting that already happened is never left as "Booked". Otherwise use Booked for a brand-new row or Reschedule for a moved one, per the rules above.
   - `Persona/Title`: map the person's title to the closest fitting option on this constrained select — err toward picking the nearest reasonable bucket (e.g. a "Global Marketing Manager" reasonably maps to "Head of Marketing" or "Marketing Operations") rather than leaving it blank, but never pick an option that's a genuine mismatch (don't call an engineer a "Creative Director"). Set this on every create or update, not just new rows.
   - Set `LinkedIn of Prospect` to their profile URL if known.
4. Do not touch Conversation Hub's `Connected`, `Connected on LinkedIn?`, `Last touched`, `Suggested follow-up`, or `Quick notes` in this branch, and do not draft a follow-up message — the All Meetings row is the record of this touchpoint, and the next real event for this person is the meeting itself, not a follow-up nudge.

**Branch B — no meeting booked yet:**

1. Match an existing Conversation Hub page by Name + Account (same dedup rule as `/convohub`'s Step 1b/5 — check before creating, never duplicate). Skip entirely if `Had meeting?` is already checked on that page.
2. If a page exists: check `Connected on LinkedIn?` (messaging on LinkedIn implies a 1st-degree connection); check `Cold call connected?` for every LinkedIn-matched person written here this way, regardless of channel or reply status (explicit team rule — this property is being used as a broader "connected" flag here, not literally "a phone call connected"); check `Chatting?` only if the thread shows genuine back-and-forth — the person has actually replied at least once — never for one-sided outreach that got no response; append `"LinkedIn"` to `Connected` if not already present (no duplicates); set `Last touched` = the actual date of the last message in the thread (not necessarily today); set `Suggested follow-up` = a near-term follow-up date, judged from the conversation's momentum — this is what triggers the built-in Notion reminder, so it must be a genuine date, not a placeholder; set `URL LinkedIn` if not already set. Leave `Disposition` and `Priority` untouched unless the thread contains an unambiguous next-step signal — never guess. Write the page content per the "Page content format" rule below.
3. If no page exists: create one with Name, Rep (the resolved rep's row), Title, Account (already resolved in Step 4 — never leave this blank), `Connected on LinkedIn?` checked, `Cold call connected?` checked (same rule as above — every LinkedIn-matched person gets this), `Chatting?` checked only on genuine back-and-forth (same rule as above), `Connected` = ["LinkedIn"], `Last touched`, `Suggested follow-up`, `URL LinkedIn`, and page content per the "Page content format" rule below.
4. Draft a short follow-up message: short, warm, casual — 2 to 4 sentences; grounded only in what the thread actually discussed, never fabricated; ask a genuine question where one fits naturally, or offer something of concrete value (an invite to an upcoming webinar, a recording of a past event, a relevant report or guide) — when offering something, pull the specific name from Conversation Hub's `Campaign` property options (the closed, real vocabulary of Superside's actual named events/content), never invent an event or report title. If nothing in that list fits the conversation, ask a question instead of forcing an offer. This draft goes in the page content per the format below; check `Suggested draft made` once it's written.

**Page content format (applies to both the person's Conversation Hub page and their Account page, both branches):**

The block to write is always:

```
## Conversations
**LinkedIn:** <one-line summary — on an Account page with more than one matched person, one bolded "LinkedIn:" line per person, name and title included>

## Follow-up
<the suggested follow-up date, and the drafted message from item 4 above — for Branch A (meeting booked), or when a fresh draft isn't warranted (e.g. a page already carries a current follow-up plan from another source like /convohub), write a short line instead, e.g. "meeting already booked, see All Meetings" or "see the existing follow-up plan above">
```

Where to put it:
- If a heading containing the word "Conversations" already exists on the page (case-insensitive — this includes `/convohub`'s call log heading), insert this block at the *top* of that existing section, right after the heading, rather than creating a second "Conversations" heading.
- If the page has other content but no heading containing "Conversations" (e.g. `/convohub`'s cold-call notes, which use headings like `## Summary of last communication`), prepend the whole block above to the very top of the page — before all existing content, never overwriting or deleting it.
- If the page is blank, the block above is the entire content.
- If the page already has a `## Follow-up` heading from a prior run of this skill, add the new line into that existing section instead of creating a duplicate heading.

Never duplicate the `## Conversations` or `## Follow-up` heading on a single page.

## Step 7 — Report back

Plain-text summary: how many inbox conversations were pulled, how many were skipped (already-synced per Step 3, or no account match per Step 4 — list both counts separately), and for every matched person: name, company, one-line summary, which branch applied (meeting logged to All Meetings, vs. Conversation Hub updated + follow-up drafted), and — for Branch B — the drafted follow-up message so the rep can see it before it's ever sent. Also call out explicitly: any Branch A meeting whose date wasn't confirmable on the calendar, any person with no Salesforce match, and any Salesforce duplicate-record situation. Never repeat full message text from the original thread — names, accounts, and the gist only.

Before finishing, set `Last linkedin check` on the resolved rep's Monthly Scoreboard row to the current time.

## Step 8 — Sending a message (only when explicitly asked, never automatic)

If the rep asks to message someone from this run's results (or any tracked contact), draft the message text and show it in chat first. Only call `send_message(linkedin_username, message, confirm_send=true, ...)` after they give an explicit, distinct confirmation in that turn (e.g. "send it," "yes go ahead") — never as a follow-on action to Steps 1-7 in the same breath, even if their original request implied the full pipeline including sends. `send_message` is not on the permission allow/deny list on purpose: it should always prompt for confirmation at the tool-call layer too, on top of this chat-level check. If `get_person_profile` was already called on this person in Step 4, prefer passing its `profile_urn` to `send_message` — the tool's own docs note this is more reliable than the Message-button lookup.
