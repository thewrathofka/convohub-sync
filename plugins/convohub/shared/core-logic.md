# Convohub core logic

This file is the shared logic for all three convohub commands (`convohub`, `convohub7`, `convohub30`). Each command's SKILL.md sets two parameters — `CALL_WINDOW_DAYS` (1, 7, or 30) and `RUN_NO_SHOW_RESOLUTION` (false for the 1-day command, true for 7/30) — then follows this file. Run fully autonomously; make best-guess judgment calls on subjective matters (e.g. priority level, follow-up channel, meal-word choice) and note assumptions in the final summary rather than asking mid-run, except where explicitly told to ask below. This does not extend to factual/identity matters — which Account or contact a record belongs to, whether two records are the same company, or any property value with no real evidentiary basis: never guess or invent these. If you cannot confidently determine one, leave the property unset/unchanged and flag it explicitly in the Step 9 summary instead.

## Resolve the team's databases first (multi-team)

Before Step 0, get the four Notion data-source IDs for the **running rep's team** by following
`superside-sales-core`'s `shared/team-resolution.md` (rep → team via the bundled roster + the `💸 Sales`
page → the team's "Databases" page → the four DBs by name), and use those IDs everywhere below in place of
the fixed IDs. If `superside-sales-core` isn't installed or resolution can't complete, fall back to the
Money Moguls defaults below. Never write into a team you didn't resolve; if ambiguous, stop and ask.

**AE resolution:** prefer the resolved team's Monthly Scoreboard `AE` person field (populated at
`team-setup`, lazy-filled otherwise). The legacy `BDRs - Money Moguls` DB below is Money-Moguls-only —
use it as a fallback for that team only.

## Fixed IDs — Money Moguls defaults (fallback; prefer the resolved IDs above)

- Conversation Hub data source: `39d50ebb-bec0-8054-89c6-000b1f0a5914`
- All Accounts data source: `a5268c41-9288-4977-92df-f541a790a1bf` (database `69619f3b-0d0c-4aec-b9e9-cec5d8c6b326`)
- All Meetings data source: `30950ebb-bec0-80fc-81c8-000b4f854176`
- Monthly Scoreboard (Rep) data source: `30950ebb-bec0-8003-bb35-000b6b021eec`. Also carries a `Last Convohub Run` (date/datetime) property, set at the end of every run by all three commands — used by the daily command to compute its call-pull window (see Step 2).
- BDRs - Money Moguls data source (BDR → AE pod pairing): `30a50ebb-bec0-811a-80c0-000bba20845d` — schema: Rep (title, the BDR's first name), Level (text), AE (person — a Notion user reference), Place (place).
- Superside email convention: everyone's email is `{first_name}.{last_name}@superside.com`.
- Marketing Resources Notion page (curated guide/report links): `30650ebb-bec0-80e4-ae7e-c7807de58052`
- Superside public resource hubs: superside.com/events, superside.com/guides, superside.com/reports, superside.com/videos

Do not hardcode any MCP server/connector-ID prefix in tool calls (e.g. `mcp__<uuid>__search_calls`) — resolve tools by name (Jiminny: `search_calls`, `get_call`, `get_deal_activities`, `get_me`, `search_members`; Notion: `notion-query-data-sources`, `notion-create-pages`, `notion-update-page`, `notion-fetch`; Gmail: `create_draft`, `search_threads`, `get_thread`; Salesforce: `soqlQuery`, `find`, `getRelatedRecords`; Calendar: `list_calendars`, `list_events`). The exact connector ID differs per rep's own account.

Schema options and property names on all three databases can and do change — the schema sections below are a snapshot as of this file's last edit. If a select/status option or property name you need isn't here, re-fetch the data source's live schema before assuming it doesn't exist or guessing at a name.

## Reachdesk gift links

| Context | Link |
|---|---|
| Asking for/angling toward a meeting | https://redeem.reachdesk.com/lp/superside/cookies-for-you |
| General thanks for talking on the call | https://redeem.reachdesk.com/lp/superside/thank-you |
| Prospect referred someone else to talk to | https://redeem.reachdesk.com/lp/superside/referrals |
| Confirmed coming to an event | https://redeem.reachdesk.com/lp/superside/pre-event |
| Attended an event (post-event thank you) | https://redeem.reachdesk.com/lp/superside/post-event |
| Anything else gift-shaped | No link — insert a `[REP: insert gift link here]` placeholder instead |

## Above-the-line / below-the-line title tiers

Only applies to contacts flagged Positive/Neutral in Step 3 (Meeting-booked and soft-nos are unaffected — see Step 7).

**Above the line** — an exact/obvious match to any of: Chief Marketing Officer, CMO, VP of Marketing, Vice President of Marketing, Head of Marketing, Director of Marketing, Marketing Director, Senior Marketing Manager, Global Head of Marketing, Chief Brand Officer, VP of Brand, Head of Brand, Brand Director, Director of Brand, Brand Marketing Director, Head of Brand Marketing, Global Brand Director, Chief Creative Officer, CCO, VP of Creative, Head of Creative, Creative Director, Executive Creative Director, Global Creative Director, Head of Design, Design Director, VP of Design, Head of Studio, Studio Director, Head of In-House Agency, Head of Content, Director of Content, VP of Content, Content Marketing Director, Chief Communications Officer, VP of Communications, Head of Communications, Director of Communications, Communications Director, Head of Corporate Communications, VP of Corporate Communications, Head of Performance Marketing, Director of Performance Marketing, Head of Growth Marketing, VP of Growth, Head of Demand Generation, Director of Demand Generation, Head of Digital Marketing, Director of Digital Marketing, VP of Digital Marketing, Head of Product Marketing, Director of Product Marketing, VP of Product Marketing, Head of Marketing Operations, Director of Marketing Operations, Head of Creative Operations, Director of Creative Operations, Head of Creative Production — **or** a title you judge to be a clear equivalent in seniority not literally on this list (e.g. an SVP/EVP variant, a regional/global variant, an obvious synonym). This is a subjective call, not a factual-identity one — use judgment freely here.

**Below the line** — anyone with "Designer," "Consultant," or "Researcher" in their title, or any title not on the above-the-line list that you judge to be a working/individual-contributor level role rather than a functional-leadership role. This is the default bucket: when genuinely unsure, below the line, not above.

Determine this live from the contact's `Title` (Conversation Hub) every time it's needed — nothing new is stored on the page for this.

## Conversation Hub schema (current live state — use exact names)

Account (relation → All Accounts), Callback talk track made (checkbox), Campaign (multi_select, open-ended — create new options as real Salesforce campaign names are found, do not constrain to a fixed list), Chatting? (checkbox), Cold call connected? (checkbox), Connected on LinkedIn? (checkbox), Date of last campaign (date), Disposition (select — fixed list only, never add new options, see Step 8's mapping for exactly which one to pick: "MB Rejected", "MB Canceled", "CP send email", "CP not now", "CP no authority", "Meeting booked", "MB no show", "Chatting", "CP no follow-up", "CP call back", "CN call back", "CN send email", "CN not now", "CN no authority", "CN linkedin", "CP linkedin"), First follow-up sent (checkbox), Had meeting? (checkbox), Inbounded? (checkbox), Last touched (date), LinkedIn note on page (checkbox), Meeting booked - other (select: "email", "linkedin" — see Step 8's meeting-booked handling for exactly when this gets set), Name (title), Nutrure? (select — fixed list only, never add new options: "BDR-led nutrure", "Superside connect/dinner Atlanta", "Superside connect/dinner Toronto", "Superside connect/dinner Chicago", "Superside connect/dinner NYC", "Superside connect/dinner London", "Superside connect/dinner SF", "webinar"), Priority (select: High/Medium/Low), Quick notes (text), Rep (relation → Monthly Scoreboard), Second follow-up sent (checkbox), Sequence (select — currently no options defined; leave untouched unless the BDR has set one up), Suggested channel next (select: "Email", "LinkedIn", "Call again"), Suggested draft made (checkbox), Suggested follow-up (date), Title (text), Touch points (multi_select: Cold call/LinkedIn/Email), URL Jiminny (url), URL LinkedIn (url), URL Salesforce (url).

## All Accounts schema (relevant fields)

Account Name (title), Rep (relation → Monthly Scoreboard), Parent (text — "parent company name; self-name if the account itself is a parent with children"), sister/child (text — comma-joined child/sister brand names, lives on the parent's page), Priority Type (multi_select: "Growth", "Inbound", "MQA", "MQL", "A", "B", "BDM Selected" — **A, B, and BDM Selected are the priority tiers**; Growth/Inbound/MQA/MQL/blank are not, for purposes of the Channel-routing logic in Step 1c below), Meetings (relation → All Meetings, auto-populated from the reverse of All Meetings' Account relation — never set this directly, set Account on the All Meetings row instead). Leave other research/priority fields (Buying Intent, Temperature, Research Status, etc.) untouched unless you have real data — those are agent-owned by a different workflow or BDR-managed.

## All Meetings schema (relevant fields)

Company (title — company/account name, NOT the person), Rep (relation → Monthly Scoreboard), Account (relation → All Accounts — **every All Meetings row must have this set**, see Step 1d/Step 6), Name (relation → Conversation Hub — the actual person), Persona/Title (select: Creative Director, Marketing Director, CMO, Business Bob, Brand Lead, Head of Brand, Communications Director, VP of Brand Marketing, Head of Creative Operations, Brand Strategy, Head of Marketing, Marketing Operations), Status (status — to_do group: "Rejected", "No-show", "Canceled"; in_progress group: Placeholder sent, Waiting on time/date, Invite not accepted yet, Reschedule, Booked; complete group: "Completed"), Channel (select: "Email", "LinkedIn", "Cold Call", "Inbound priority", "Inbound none priority"), Date of Booking, Date of Meeting, LinkedIn of Prospect (url), SF Opp URL (url), Notes (text — short gist, NOT the full call recap; the full recap goes on the page as content, see Step 8), SQL/SAL (checkboxes — the agent sets these under one specific condition, see Step 5 item 6) / Everstage (checkbox — **agent never checks this**, always the BDR's to verify manually), Pre-meeting email sent / Pre-meeting email drafted (checkboxes).

Caveat: many existing All Meetings rows are missing Rep, Persona/Title, Name, Account, Channel, LinkedIn of Prospect, SF Opp URL, and/or other fillable properties — Step 1 below is a standing sweep specifically to close ALL such gaps wherever confidently determinable, not just a one-time cleanup, and never skips a row. Name/Account/Channel have their own dedicated substeps (1b/1c/1d) below; any other missing-but-determinable property on a row you're already touching should be filled in the same pass rather than left for later — only if it can be confidently determined, never guessed.

---

## Step 0 — Resolve the rep

Call Jiminny `get_me` to identify who is running this command (name, email, member_id, team). Search the Monthly Scoreboard data source for a row whose Name matches. If the match is clean, proceed silently using that person's Monthly Scoreboard page URL as their Rep relation value everywhere below. If ambiguous or no match, ask the person running the command to confirm/pick their name from the Monthly Scoreboard list — this is the one point in the whole run allowed to interrupt for rep identity.

Also read `Last Convohub Run` from the resolved rep's Monthly Scoreboard row — Step 2 needs it for the daily command's window.

**Resolve the rep's AE** (needed for Step 1's organizer check). Query the BDRs - Money Moguls data source for the row whose Rep matches the resolved rep's first name, and read its `AE` property.
- If `AE` is populated, resolve that Notion user reference to an email address (via the Notion users lookup) — it will follow `{first_name}.{last_name}@superside.com`.
- If `AE` is empty or the rep has no row there at all, ask the person running the command who their AE is (name or email). Once they answer, resolve/confirm the email using the `{first_name}.{last_name}@superside.com` convention, find or create the matching row in BDRs - Money Moguls, and set its `AE` property to that person via `notion-update-page` so future runs never have to ask again for this rep. This is the only other point in the run allowed to interrupt.

## Step 1 — Calendar & All Meetings reconciliation (runs FIRST, every session, all three commands, fixed window regardless of command)

This is the mandatory session-opening sweep: before pulling any cold calls, sync the calendar and close every known gap in All Meetings and Conversation Hub. Pull the resolved rep's calendar (Calendar connector, primary calendar) for a fixed window: the start of the current week through 2 weeks from today.

### 1a. New meetings from the calendar

For every event in that window, check whether it already has a matching All Meetings row (by attendees/company/date). For any that don't, evaluate this pipeline in order — stop and do nothing further with that event the moment a step fails:

1. **Organizer check.** The event organizer's email must be exactly the resolved rep's email or their AE's email (from Step 0's AE resolution). Anyone else organizing it — stop, not this rep's meeting to log.
2. **External attendee check.** At least one attendee must be on a non-superside.com domain. No external attendees — stop, it's an internal meeting.
3. **Title screen (case-insensitive).** Stop if the title contains any of: "follow up", "follow-up", "monthly check-in", "monthly checkin", "qbr", "onboarding", "weekly", "sync", "status", "catch up", "catch-up", "standup", "stand-up", or otherwise clearly reads as a recurring/existing-account touchpoint. If genuinely ambiguous, continue rather than stop.
4. **Create the All Meetings record.** Company = a clean company name derived from the external attendee's email domain. Date of Booking = when the event was added to the calendar (event creation time). Date of Meeting = the event's date. Rep = the resolved rep's relation value. Status = "Booked". Channel = leave blank here — Step 1c below determines it (a same-run event is often still ambiguous between Email/LinkedIn/Inbound until that check runs).
5. **Resolve the Account** via Step 6 below (never leave this row without an Account).
6. **Check whether this newly-found meeting was originally booked via a cold call.** Search Jiminny for a call from this rep to this contact/company, classified "Connected - Meeting booked" (Step 3's classification), dated before this meeting's Date of Booking — verify a genuine match (contact/company/date proximity); never assume one exists just because the meeting does. If found: pull its summary via `get_call` and add it to the Account's `## Conversations` log exactly as Step 8's Account-level conversation log describes (date = the cold call's date, disposition = "Meeting booked", summary from that call). If no matching cold call is found (booked via email, LinkedIn, inbound scheduler, or unclear), do not write anything to Conversations for this meeting.

### 1b. All Meetings rows with no person linked

For every All Meetings row (not just ones in this run's calendar window — this is a standing gap-closer) where `Name` is empty: cross-reference the linked Salesforce Opportunity (SF Opp URL) if present, or query Salesforce Contacts on the row's Account if there's no direct Opportunity signal, to identify who the meeting is actually with. Before creating a new Conversation Hub page, check whether that person already has one (match by name + Account) — link the existing page rather than duplicating. If no page exists, create one with every determinable property (Name, Title, Account, Rep, URL Salesforce, etc. — same fill-what-you-can-infer approach as Step 9). Set the All Meetings row's `Name` relation to point to it, and fill in `Persona/Title` if you can determine it.

### 1c. All Meetings rows with no channel selected

For every All Meetings row where `Channel` is empty:

0. **Account must already be resolved.** Never determine Channel (or its inbound-priority split) for a row whose Account isn't resolved yet — if it isn't, resolve it via Step 6 first, then proceed.
1. Check Salesforce for the external person on this meeting — via the linked Opportunity's `LeadSource` if `SF Opp URL` is set, else query Salesforce Contacts on the Account for this person's record. `LeadSource = "Book-A-Call Form"` (exact value) → inbound; anything else, including no clear signal → outbound. Do exactly one such Salesforce check per row — this is the *only* signal for inbound vs. outbound now. Do not use calendar-title conventions (e.g. "Let's Chat") or any other heuristic. If inbound: read `Priority Type` on the row's (already-resolved) Account: if it contains "A", "B", or "BDM Selected" → Channel = "Inbound priority"; otherwise → Channel = "Inbound none priority".
2. If it's not inbound, check Salesforce activity history on the contact/opportunity to determine whether the meeting was booked via email or LinkedIn outreach, and set Channel to "Email" or "LinkedIn" accordingly.
3. Either way, if the person this meeting is with inbounded, check `Inbounded?` on their Conversation Hub page — this is independent of the priority split above; it applies regardless of whether the account is a priority account or not.
4. If you genuinely can't determine the channel from any of the above, leave it blank rather than guessing, and flag the row explicitly in the Step 9 summary for the BDR to fill in.

### 1d. Account relation is never optional

Any time this step, Step 6, or Step 8 creates or updates an All Meetings row, its `Account` relation must be resolved via Step 6 — match/create in All Accounts, including the parent/child subsidiary check. Never leave an All Meetings page without an Account relation, and if evidence later points to a more specific account (e.g. the row is currently linked to a parent but the actual contact works at a subsidiary), correct it rather than leaving the stale value.

Flag every new-from-calendar record, every orphan-person fix, every channel determination, every `Inbounded?` check, and every "meeting on the calendar but missing from All Meetings" case explicitly in the Step 9 summary — this is a required disclosure, not optional detail.

### 1e. Pre-meeting emails

Once every row in the window has Channel resolved (1c) and Account resolved (1d), run the workflow in `${CLAUDE_PLUGIN_ROOT}/shared/meeting-emails-logic.md` across the window. This same sent/drafted check also applies to every existing All Meetings row in the window, not just newly-created ones — if a drafted pre-meeting email was never marked sent, check whether it actually went out (Salesforce activity + Gmail, same method as Step 7) and check `Pre-meeting email sent` if so.

## Step 2 — Pull cold calls

Call Jiminny `search_calls` with member_id=[resolved rep's Jiminny member_id], member_role="host", type="voice", date_to = now, with_crm_records=true, limit=50 (page with cursor if needed). `date_from`: for the 7- and 30-day commands, today − CALL_WINDOW_DAYS at 00:00 local time, same as always. For the 1-day command: use the `Last Convohub Run` value read in Step 0 as `date_from` directly. If that property is empty (no prior run recorded), fall back to the start of the previous working day instead — if today is Monday, the preceding Friday at 00:00 local time (skip Saturday/Sunday entirely); any other weekday (Tue–Fri), yesterday at 00:00 local time. This command only ever runs Monday–Friday, so no other fallback case is needed. Do not rely on the tool's `duration_min` parameter for the 30-second cutoff — it only accepts whole minutes. Instead fetch broadly (omit duration_min or set it to 0) and then discard, yourself, any call under 30 seconds by parsing each result's actual duration — this deliberately keeps quick "call me back" connects that a 1-minute cutoff would have thrown away. If zero calls remain after this filtering, skip straight to Step 5 and Step 7 (the standing sweeps aren't gated on new calls existing) rather than stopping the whole run.

## Step 3 — Classify each call by disposition

Disposition appears in the call title as "[Apollo] [Call] >> [Disposition]".

- IGNORE: "No Answer", "Gatekeeper", "Connected - Negative".
- FLAG — Meeting booked: "Connected - Meeting booked".
- FLAG — Positive/Neutral zero follow-up: "Connected - Positive" or "Connected - Neutral", when the contact has zero follow-up so far (no existing Conversation Hub row, or an existing row where nothing has progressed).
- FLAG — Soft-no needing acknowledgment: "CP not now" or "CP not authority" (this is Jiminny/Apollo's own call-tag wording — do not confuse with the Notion Disposition picklist below, which spells the equivalent option "CP no authority"; when writing to Notion for one of these calls, always use the Notion option's exact current spelling, never Jiminny's). These are no longer ignored.

For Meeting-booked and Positive/Neutral calls, call `get_call` for the summary/discussion topics/action items. This is also where you look for: an explicit content ask (Step 8), a gift/thank-you mention (Step 8), an event invite (Step 8), a stated channel preference or callback agreement (Step 7), and any campaign-worthy context. For Meeting-booked calls specifically, this is also where you capture the agreed meeting date/time and enough of the discussion to write both the short gist (All Meetings `Notes`) and the fuller recap (page content) referenced in Step 8.

## Step 4 — No-show resolution for calls pulled in this run (only if RUN_NO_SHOW_RESOLUTION is true — i.e. convohub7 and convohub30, not the daily convohub)

For each Meeting-booked call with a deal ID, call `get_deal_activities(deal_id, type="video")`. A video activity with real duration (>0) and a populated summary = the meeting happened — log it, no draft needed. A video activity with 0 duration and no summary = a no-show. If the meeting date is still in the future, log as upcoming, no verdict yet.

For a no-show: only treat it as resolved if the prospect gave a genuine reply (not just an outbound rep follow-up) that led to either (a) a new meeting actually appearing on the calendar, or (b) an explicit statement of no interest. Anything short of that — silence, an unresolved back-and-forth, only a rep-side nudge — still counts as unresolved: set Disposition = "MB no show" and proceed to Step 7/Step 8 treatment for it.

**Keep All Meetings in sync with what Jiminny/Salesforce say.** Whatever this step determines about a booked meeting (happened / no-show / resolved-by-reschedule / resolved-by-decline), check whether a matching All Meetings row exists (match by Account + Rep + date proximity). If it doesn't exist, create it with the correct Status (and resolve Account via Step 6, per Step 1d). If it exists but its Status doesn't match reality, correct it. This is the same upsert used in Step 1a/Step 5 below — don't duplicate a row that's already there, just bring it in line.

## Step 5 — All Meetings cleanup (runs in ALL THREE commands, including daily — not gated on the call-pull window)

This is a standing sweep, independent of Step 2's window. For every All Meetings row where Rep = the resolved rep (or reasonably inferable as theirs) and Date of Meeting has passed without the Status already being a resolved terminal state:

1. First check the row's Conversation Hub `Name` relation (if set) — if that Conversation Hub page has `Had meeting?` checked, skip this row entirely, no further action of any kind anywhere in this workflow for that person.
2. Verify what actually happened via Jiminny (matching video call activity — real duration + summary = it happened) and/or Salesforce (opportunity activity/notes).
3. Set All Meetings Status accordingly: "Completed" if it happened; otherwise "Canceled" (prospect emailed to cancel, no reschedule, no explicit disinterest stated), "No-show" (time passed, no word at all), or "Rejected" (they declined the calendar invite outright, no email).
4. Identify the actual contact if the row is missing one — same method as Step 1b. Create or update that person's Conversation Hub page (correct Name, Title, and — always — the resolved rep's Rep relation value, never leave Rep blank or copy a null value from the source row), then set the All Meetings row's `Name` relation to point to it, and fill in `Persona/Title` on the All Meetings row if you can determine it.
5. On the Conversation Hub page: if it happened, check `Had meeting?` and stop — no follow-up drafting for this person, this run or any future run. If it didn't, set Disposition = "MB Canceled", "MB Rejected", or "MB no show" to match, and route it into Step 8's drafting the same as any other flagged contact.
6. **SQL/SAL update for completed meetings.** Once this row's Status is "Completed" (set just now or already set from a prior run), check Salesforce for an open Opportunity (`IsClosed = false`) tied to the same Account and the same contact (this row's `Name` person). If one exists at Stage `"1 - Discovery"` → check `SQL` on this All Meetings row. If one exists at *any* open stage for that same person (Stage 1 - Discovery included) → check `SAL` on this All Meetings row, regardless of stage — both boxes can end up checked together. If no open Opportunity is found for that person, leave both unchecked — don't guess. This is the one exception to "agent never touches SQL/SAL/Everstage" — Everstage remains agent-never-touched in every case.

## Step 6 — Account resolution (All Accounts)

This routine is shared by every step above and below that needs to set or verify an Account relation — on a Conversation Hub row **or** an All Meetings row. It's the same underlying match/create/parent-child logic either way, since both databases relate to the same All Accounts data source.

1. Query All Accounts filtered to Rep = the resolved rep. Try to match the contact's/meeting's company to an existing Account Name (exact or obvious variant — including punctuation differences: apostrophes ("Talbots" vs. "Talbot's"), ampersand vs. "and", "Inc"/"Incorporated", etc.). If genuinely unsure whether an existing row is the same company, cross-check the company name as it appears in Salesforce — the Salesforce Account name is the source of truth and should match what's in All Accounts — before concluding no match exists.
2. If no match: check whether it's a child/subsidiary of one of the rep's existing accounts.
   - First, try obvious naming pattern (e.g. shared brand root, "X Digital"/"X Creative" suffixes) or shared email domain pattern (the contact's email domain matches or is a known variant of an existing account's domain).
   - If not obvious from naming/domain, do a quick web search to confirm corporate ownership (parent company, acquisition history, "a brand of X" language on the company's own site, etc.) before concluding either way — don't guess a parent/child relationship without at least one of naming, domain, or search evidence.
   - If it is a subsidiary: create a new All Accounts page (Account Name = new company, Rep = resolved rep) with Parent = the existing parent's Account Name. Also update the parent's own page: append the new child's name to its `sister/child` field (comma-joined with whatever is already there), and set the parent's own `Parent` field to its own name if not already set (never overwrite an existing value there).
   - If not a subsidiary: create a new All Accounts page with just Account Name + Rep.
3. Use the resulting page as the `Account` relation value — on the Conversation Hub row, the All Meetings row, or both, whichever triggered this resolution.
4. **Organic backfill on newly-created accounts.** Whenever this routine *creates* a new All Accounts row, follow the **Organic backfill** in `superside-sales-core`'s `shared/salesforce-import.md`: set `Priority Type` from the account's Salesforce `Priority_Account__c` (A→A, B→B, C→BDM Selected, BU tiers→BU Expension only for eligible reps) plus descriptive-only fields (Account URL, Size, Last Prospected, Parent) — never buying-signal/lead fields. No confident SF match → leave `Priority Type` unset and flag; never guess a tier.

## Step 7 — Conversation Hub follow-up cadence (every run, all three commands — scans existing rows for this rep, not just ones touched by Steps 1-5)

Skip any row where `Had meeting?` is already checked.

**Above/below-the-line split (Positive/Neutral contacts only — see the tier table above; Meeting-booked and soft-no contacts skip this branch and keep working exactly as described below).**
- **Above the line:** no change — proceed through First follow-up / Second follow-up / Nurture hand-off exactly as written below.
- **Below the line:** do not create any follow-up unless the call explicitly demanded one (an explicit ask — "send me pricing," an agreed callback, "send me a case study," etc. — the same signal Step 3 already looks for). If nothing explicit was said: skip all follow-up drafting for this person entirely — their Conversation Hub page, Quick notes, and the Account-level Conversations log still get created/updated exactly as Step 8 always does, this branch only skips the follow-up machinery. If something explicit was said: draft exactly one follow-up (Step 8, whichever channel the ask calls for), forcing `Priority` = Low regardless of Step 8's normal account-size-based priority logic, and stop there permanently — no second follow-up, no 7-day wait, no Nurture hand-off, ever, for this person.

**First follow-up.** For rows with `Suggested draft made` checked but `First follow-up sent` not yet checked: check whether it actually went out — primary signal is Salesforce activity history on the contact (captures email, LinkedIn touches, and calls in one place), filtered strictly to activities logged by the resolved rep's name; cross-check Gmail (sent mail / draft still present or not) when Salesforce is inconclusive. If sent, check `First follow-up sent`.

**CP not now / CP not authority contacts specifically:** these get exactly one acknowledgment email (tracked the same way via `First follow-up sent`), then skip directly to the Nurture assignment below once that single email is confirmed sent — no second follow-up, no 1-week wait.

**Everyone else (Meeting-booked-no-show / Canceled / Rejected / Positive / Neutral):** once `First follow-up sent` is checked and 7+ days have passed with no reply, recommend the next follow-up:
- Channel: if the prospect explicitly agreed to LinkedIn or a specific callback time on the call, use that. Callback → Suggested channel next = "Call again", Suggested follow-up date/time = exactly what was agreed, Priority = High always. Otherwise use judgment on Email vs. LinkedIn for `Suggested channel next`.
- If the callback has real context behind it (something specific to raise, not just "call me back") and `Callback talk track made` isn't checked, write a short talk track onto the person's page and check that box. No context shared → skip, no talk track needed.
- Always draft the backup follow-up email regardless of channel (Step 8). If the channel is LinkedIn and `LinkedIn note on page` isn't already checked, also draft a connection-request note as page content and check that box — never redraft it if already present.
- Once sent (same Salesforce/Gmail check as above), check `Second follow-up sent`.

**Nurture hand-off.** Once `Second follow-up sent` is checked (or, for CP not now/not authority, once the single acknowledgment is sent): assign `Nutrure?` from its fixed existing option list only (never create a new one) — match the prospect's location against the pre-set cities (Atlanta, Toronto, Chicago, NYC, London, SF) for an in-person Superside Connect/Dinner in the next 1-2 months (check superside.com/events), else "webinar" if one's coming in the next month, else "BDR-led nutrure". Anyone currently on "BDR-led nutrure" gets rechecked every run in case a matching event now exists — upgrade if so.

## Step 8 — Draft/update the Conversation Hub page and the Gmail draft; sync meeting-booked calls to All Meetings

Create or update the Conversation Hub page with every property you can respectably infer: Name, Rep (resolved rep), Title, Account (Step 6), Priority (High for known large/strategic accounts or an existing active deal, Medium for solid mid-market, Low if the deal is Closed Lost — except callback agreements, which are always High per Step 7), Touch points (append "Cold call" — and "LinkedIn"/"Email" as later touches happen), Last touched = the call date, Quick notes = 1-2 sentence recap, Disposition (see the mapping immediately below), Suggested follow-up, Suggested channel next, Suggested draft made = true once a draft exists, URL Jiminny, URL Salesforce if available. Page content includes "## Summary of last communication", "## Suggested follow-up", and "## Draft email (created in Gmail as draft, not sent)" sections (append rather than replace on updates). If the person already has a page, only change what needs changing and add what's missing — never blindly overwrite an existing value.

**Disposition mapping for Connected-Positive/Connected-Neutral calls.** `Disposition` is a fixed picklist — choose from it, never add a new option (same rule as `Nutrure?`). Current live options: `MB Rejected`, `MB Canceled`, `CP send email`, `CP not now`, `CP no authority`, `Meeting booked`, `MB no show`, `Chatting`, `CP no follow-up`, `CP call back`, `CN call back`, `CN send email`, `CN not now`, `CN no authority`, `CN linkedin`, `CP linkedin` (re-verify live before assuming this list is current — it can change). For a call Step 3 flagged Positive (CP) or Neutral (CN), pick based on what was actually said about next steps — this is the same signal Step 3 already gathers via `get_call` and Step 7 already looks for (a stated channel preference or callback agreement):
- Explicit callback agreed (a time discussed) → `CP call back` / `CN call back`.
- Email named/agreed as the next channel → `CP send email` / `CN send email`.
- LinkedIn named/agreed as the next channel → `CP linkedin` / `CN linkedin`.
- Explicit soft deferral ("not right now," "check back later," etc., short of an outright no) → `CP not now` / `CN not now`.
- Explicitly not the decision-maker/authority → `CP no authority` / `CN no authority`.
- Explicitly said they don't want any follow-up → `CP no follow-up` for a Positive call. **There is no `CN` equivalent of this option right now** — if a Neutral call explicitly declines follow-up, do not force it into `CP no follow-up` (wrong prefix) or invent a new option; use `CN not now` as the closest real match and flag the gap explicitly in the Step 9 summary so it can be added to the picklist if wanted.
- Nothing concrete decided either way → `Chatting` (matches its existing "still ongoing/unresolved" meaning) — this doesn't block drafting a follow-up; `Suggested channel next` still gets set by ordinary judgment (Email vs. LinkedIn) exactly as Step 7 already describes.

Keep `Suggested channel next` in sync with whichever Disposition you just picked — `call back` → "Call again", `send email` → "Email", `linkedin` → "LinkedIn" — never let the two fields imply different channels for the same person.

**Meeting-booked cold calls — the All Meetings side.** For every call flagged Meeting-booked in Step 3, in addition to the Conversation Hub update above: set Disposition = "Meeting booked" (this applies regardless of channel — it's the universal "a meeting exists with this person" flag). Then reconcile All Meetings:
- Resolve the matching All Meetings row the same way Step 5 does (match by Account + Rep + date proximity).
- If a matching row exists and its page is not empty (already has content on it, from any source): prepend a new `## Cold call notes (<date>)` section to the *top* of the page's content, above whatever's already there — never overwrite or push existing content further down than necessary, and never delete it.
- If no matching row exists: create one with every property filled in — Company, Rep, Account (Step 6 — never skip this per Step 1d), Name (relation to this Conversation Hub person), Persona/Title if determinable, Status = "Booked", Channel = "Cold Call", Date of Booking = the call date, Date of Meeting = the date agreed on the call, Notes = a 1-2 sentence gist of the call (this is the short version — the fuller recap goes on the page as content, same split as Conversation Hub's Quick notes vs. page sections). Leave SAL, SQL, Everstage, and Pre-meeting email sent/drafted unchecked at creation time. SAL/SQL may get set later, once Status becomes "Completed", by Step 5 item 6's Salesforce open-opportunity check. Everstage and Pre-meeting email sent/drafted stay the BDR's to verify/set manually, never agent-set.
- Separately, if this meeting was booked through a follow-up email or LinkedIn touch rather than the original cold call (i.e. detected via Step 7's follow-up cadence, not this step), set Conversation Hub's `Meeting booked - other` to "email" or "linkedin" accordingly — this field stays blank for cold-call and inbound bookings, since those are already captured by Touch points/Channel.

**Contact no longer at the company.** If the prospect states on the call that they've left/are no longer at this company: do not create or link a Conversation Hub page tying them to this Account. Instead, only log it on the Account's own page — add a `## Conversations` entry the same way as the Account-level conversation log below, clearly flagged, e.g. `**{Name}** — {date} — No longer at company: {short summary}`. Skip all other Step 8 handling for this call (no follow-up drafting, no cadence, no campaign context) — there's no one at this account to follow up with from this call.

**Account-level conversation log.** For every call classified in Step 3 as Meeting-booked, "Connected - Positive", or "Connected - Neutral" (not for CP-not-now/CP-not-authority, and not for ignored calls), also log it on the account's own page in All Accounts — this is in addition to, not instead of, the Conversation Hub and All Meetings updates above. Resolve the Account via Step 6 (it's likely already resolved for this contact earlier in the pipeline). On that Account page:
- Find the existing `## Conversations` heading if there is one. There should only ever be one on a page — never create a second.
- Add the new entry as the first item directly under that heading (newest first), formatted: `**{Name of person}** — {date of call} — {Disposition}: {short summary}`. Never delete or reorder existing entries underneath it.
- If the page has other content but no `## Conversations` heading yet, prepend a new one (with this as its first entry) to the very top of the page, above whatever else is there — same "never overwrite, always prepend to the top when the page isn't empty" rule as the Cold call notes handling above.
- If the page is genuinely empty, the `## Conversations` heading and this entry become the page's content.
- The summary written here always comes from the cold call itself (via `get_call`, same source as Step 3) — never from the content of a qualification meeting/call. This section only ever reflects cold-call history; it is never populated from a meeting/qualification call, whether that call is found via Steps 2-4 or via Step 1a's gap-filling (item 6 above).

**Campaign context — cold-call-sourced contacts only, never for contacts added via Step 1's All Meetings sweep.** For every contact newly entering Conversation Hub through Steps 2-4, check Salesforce campaign history for that contact. Write the campaign name into `Campaign` (create a new select option if this is a campaign not seen before — this field is intentionally open-ended, unlike Nurture). Note on the page whether that campaign touch happened before or after the first cold-call connect. Set `Date of last campaign` to when they actually entered that campaign (the real download/signup/response date from Salesforce), not today's date. Use the actual campaign as the stated reason for reaching out in the drafted email, callback talk track, or LinkedIn note (e.g. "since you downloaded the Shift Happens Guide...") rather than generic language.

**Content-aware drafting.** Only when the call surfaced an explicit content ask from the prospect: pull a real, specific asset via the `case-study-finder` skill (case studies) or the Marketing Resources Notion page / superside.com/guides /reports (guides/reports), and link the actual asset. Not every email needs this.

**Gift/thank-you handling.** If the call discussion is gift-shaped (see the Reachdesk table above), format the email as: a contextual opening sentence ("Here's that gift I promised:", "Thanks for talking today, hope we meet soon:", "Here's those treats for the event:", or similar, chosen to fit what was actually said) followed by a short bracket-hyperlinked word — e.g. `[treats]`, `[thanks]` — linking to the matching Reachdesk URL. If nothing on the list fits, use a `[REP: insert gift link here]` placeholder instead and flag it explicitly in the run summary.

**Event invites.** If the BDR invited the prospect to an event on the call, link the actual event (from superside.com/events) the same bracketed way — e.g. "Here's that signup link: [event_name]" — and always close that email with: "Reply JOINING if you've signed up and I'll send over some treats before the event."

Create the Gmail draft (never send) addressed to the contact's real email. Skip creating a duplicate draft for anyone who already has the relevant "sent" checkbox checked for that stage (First/Second follow-up) or a not-yet-sent draft already on file for that stage.

**Apollo task creation.** Whenever this run drafts any follow-up (first or second touch, a soft-no acknowledgment, a Meeting-booked no-show follow-up, or a below-the-line contact's single explicit-demand follow-up), also create a matching Apollo task — in addition to, never instead of, the Notion writes above.
1. Resolve the rep's Apollo `user_id` once per run (not per task) — match the resolved rep's Superside email against Apollo's own user list.
2. Resolve the contact's Apollo `contact_id` by searching Apollo contacts by email (the person's real email, already known from Conversation Hub/Salesforce). If it can't be confidently resolved, skip the Apollo task for this person, keep the Notion draft as normal, and flag it in the Step 9 summary — never guess an ID.
3. Dedup check — call `apollo_tasks_search` filtered to this `contact_id` and the task type you're about to create. If a matching open (scheduled) task already exists, skip creating a new one (note it in the summary) rather than duplicating.
4. Map the follow-up channel to a task type and content:
   - Email → `type = "outreach_manual_email"`, `standalone_outreach_task_message.subject`/`.body_text` = the exact drafted email.
   - LinkedIn → `type = "linkedin_step_connect"` (this plugin always drafts LinkedIn follow-ups as a connection-request note, per the `LinkedIn note on page` handling above), `standalone_outreach_task_message.body_text` = the connection-request note text.
   - Call again / callback → `type = "phone_call"`. Calls have no `standalone_outreach_task_message` field — put the callback talk track directly in `note` instead.
5. `note` field: always start with 1-2 lines of context (name, title, company, why this follow-up exists); for `phone_call` tasks, append the callback talk track directly below that context.
6. `title`: short, e.g. "Follow up: {Name} at {Company}".
7. `due_at`: exactly the same date/time as Notion's `Suggested follow-up`.
8. `priority`: exactly the same tier as Notion's `Priority` for this contact (High/Medium/Low → high/medium/low).
9. Report every Apollo task created, skipped-as-duplicate, or skipped-as-unresolvable in the Step 9 summary, folded into that person's existing report line — not a separate list.

## Step 9 — Report back

End with a concise plain-text summary: how many cold calls happened in the window, how many were flagged (broken down by category — new no-shows/canceled/rejected, newly-confirmed resolutions from Step 4/5, new Positive/Neutral/CP-not-now-or-authority contacts, second-follow-up recommendations, nurture hand-offs, meetings booked and synced to All Meetings), and the list of names for each — always paired with their company (never a bare name), covering everything done for that person in one line, including whether a `## Conversations` entry was added to their Account page and whether it was prepended into an existing section or the section had to be created. Report this once, here — don't repeat it in a separate list elsewhere. Explicitly call out every property change made to an *existing* row (not just new rows) — checkbox flips, Nurture upgrades, Disposition changes, Status/Channel changes in All Meetings, Campaign additions — with a one-line reason.

Separately and explicitly list, from Step 1: any meeting found on the calendar that wasn't in All Meetings, any All Meetings row that had no person linked and what contact was identified/created for it, any All Meetings row whose Channel was determined (and how — Salesforce Book-A-Call-Form check plus priority-tier lookup for inbound, or Salesforce activity signal for Email/LinkedIn) versus left blank for the BDR, any `Inbounded?` checks made on Conversation Hub, and any new All Accounts parent/child relationships created or corrected (name the parent, the child, and the evidence — naming pattern, domain, or web search).

Also list: any Rep corrections made on an All Meetings row, and whether the rep's AE had to be asked this run (and what was recorded going forward). Do not repeat full email bodies; just names, accounts, and reasons.

Before finishing, set `Last Convohub Run` on the resolved rep's Monthly Scoreboard row to the current time — every command does this, regardless of `CALL_WINDOW_DAYS`, since all three do the same call-review work over their respective windows.

Do not ask the user any other questions during the run — make reasonable judgment calls per the guidance above and note assumptions in the summary instead.
