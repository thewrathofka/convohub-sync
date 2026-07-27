# Notion setup spec (exact copy of the team databases)

Declarative blueprint for recreating a team's four databases **exactly** as they exist in Money Moguls
(captured live 2026-07-27, after Kali's edits). Used by `skills/team-setup` during first-run bootstrap.
Every property, type, and select option below is verbatim — preserve exact strings, including the typos
`Nutrure?` and `BU Expension`, the empty `Sequence` select, and any trailing spaces.

Build via the Notion MCP by name: `notion-create-pages` (team page + "Databases" page),
`notion-create-database` / `notion-update-data-source` (schema DDL), `notion-query-data-sources`
(idempotency checks). Never hardcode a `mcp__<uuid>__` prefix.

## Build order (relations require targets to exist first)
1. Create the four databases **empty** (title property only) inside the team's "Databases" page:
   **Monthly Scoreboard**, **All Accounts**, **Conversation Hub**, **All Meetings**.
2. Add all non-relation properties to each (tables below).
3. Add relations last, wiring the graph in §5 (two-way pairs use `DUAL`).
4. Create the **All Leads** page with a linked view of Conversation Hub (§6).
5. Formulas (§7) are **not API-creatable** — skip; document for manual re-entry.
Idempotent: before creating anything, `notion-fetch` the target; if a database/property/option already
exists, leave it; only add what's missing. Never duplicate an option or a database.

DDL type reference: `TITLE, RICH_TEXT (text), DATE, PEOPLE (person), CHECKBOX, URL, NUMBER,
SELECT('opt':color,…), MULTI_SELECT('opt':color,…), STATUS, RELATION('ds_id') /
RELATION('ds_id', DUAL 'PairedProp' 'paired_id')`.

---

## 1. Monthly Scoreboard  (title property: `Name`)
Plain-settable:
- `Name` — TITLE
- `Level` — SELECT(`3a`:green, `2a`:yellow)
- `Monthly SQL target` — NUMBER
- `Monthly MC target` — NUMBER
- `AE` — PEOPLE
- `Email exclusions` — RICH_TEXT
- `Last email check` — DATE
- `Last Convohub Run` — DATE
- `Last linkedin check` — DATE

Relations (see §5): `All Meetings 1` (DUAL ↔ All Meetings.`Rep`), `All Meetings (legacy)` (one-way → All Meetings).
Formulas (skip, §7): `Meetings`, `Meetings Booked`, `Meetings Completed`, `Fattest Mogul`, `Monthly SQLs Bar`, `Priority SQL Count`, `Total Meetings Booked`, `Total SQL Count`.

## 2. All Accounts  (title property: `Account Name`)
Plain-settable:
- `Account Name` — TITLE
- `Account URL` — URL   *(desc: "Website / homepage")*
- `Cluster` — RICH_TEXT
- `Company Structure` — RICH_TEXT
- `Notes` — RICH_TEXT
- `Parent` — RICH_TEXT
- `sister/child` — RICH_TEXT
- `Strategy` — RICH_TEXT
- `Structure Notes` — RICH_TEXT
- `CL or CW?` — SELECT(`CL`:blue, `CW`:green)
- `Prospecting Status` — SELECT(`Prospecting`:gray, `SQL`:blue, `SAL`:purple, `Qualified`:green, `Neglected`:red)
- `Research Confidence` — SELECT(`high`:green, `medium`:yellow, `low`:orange, `failed`:red)
- `Research Status` — SELECT(`pending`:gray, `done`:green, `needs_review`:yellow, `failed`:red, `out_of_scope`:brown)
- `Size` — SELECT(`<1000`:gray, `1000-2000`:blue, `2000-5000`:green, `5000+`:purple)
- `Temperature` — SELECT(`Hot`:red, `Warm`:orange, `Cool`:blue, `Cold`:gray)
- `Needs Attention` — MULTI_SELECT(`structure-event-new`:red, `structure-type-change`:orange, `funding-round-new`:green, `agency-switch-new`:purple, `rebrand-campaign-new`:pink, `ai-initiative-new`:blue, `tier1-role-open`:yellow, `layoff-new`:brown, `pain-tag-shift`:gray)
- `Pain Point Tags` — MULTI_SELECT(`creative production`:blue, `localization`:orange, `new territory`:green, `strategy`:purple, `audience education`:yellow, `competitive displacement`:red, `brand evolution`:pink, `launch surge`:brown, `AI receptivity`:gray, `post-layoff overflow`:default)
- `Priority Type` — MULTI_SELECT(`Growth`:brown, `Inbound`:blue, `MQA`:green, `MQL`:purple, `A`:red, `B`:orange, `BDM Selected`:yellow, `BU Expension`:gray)
- `Last Hygiene Check` — DATE
- `Last Prospected` — DATE
- `Last Researched` — DATE
- `Last Touch Point` — DATE

Relations (see §5): `Meetings` (DUAL ↔ All Meetings.`Account`), `Rep` (one-way → Scoreboard), `Campaigns` (one-way → Conversation Hub).
System (auto, not created): `Created` (created_time), `Last Edited` (last_edited_time).
Formulas (skip, §7): `Number of Connects`, `Total Number of Leads`.

## 3. Conversation Hub  (title property: `Name`)
Plain-settable:
- `Name` — TITLE
- `Title` — RICH_TEXT
- `Quick notes` — RICH_TEXT
- `CL or CW?` — SELECT(`close lose`:yellow, `close won`:blue)
- `Priority` — SELECT(`High`:red, `Medium`:yellow, `Low`:gray)
- `Suggested channel next` — SELECT(`Email`:green, `LinkedIn`:blue, `Call again`:red)
- `Sequence` — SELECT()  *(no options — create empty)*
- `Disposition` — SELECT(`MB Rejected`:default, `MB Canceled`:pink, `CP send email`:blue, `CP not now`:gray, `CP no authority`:red, `Meeting booked`:green, `MB no show`:orange, `Chatting`:purple, `CP no follow-up`:brown, `CP call back`:yellow, `CN call back`:pink, `CN send email`:orange, `CN not now`:purple, `CN no authority`:purple, `CN linkedin`:brown, `CP linkedin`:gray)
- `Nutrure?` — SELECT(`BDR-led nutrure`:yellow, `Superside connect/dinner Atlanta`:default, `Superside connect/dinner Toronto`:gray, `Superside connect/dinner Chicago`:purple, `Superside connect/dinner NYC`:green, `Superside connect/dinner London`:pink, `Superside connect/dinner SF`:brown, `webinar`:orange)
- `Connected` — MULTI_SELECT(`Cold call`:blue, `LinkedIn`:purple, `Email`:green)
- `Campaign` — MULTI_SELECT (59 options, verbatim, in order): `2023Q2 - Lifecycle - Intent Signal: Pricing Page`, `2025Q1 - Content - Overcommitted Report 2025 [Guide]`, `2025Q3 - Content - Inside Great Creative Partnerships [Guide]`, `2025Q3 - Content - The No-Hype AI Report [Guide]`, `2025Q3 - Lifecycle - Intent Signal: Case Studies`, `2025Q3 - Lifecycle - Intent Signal: Our Work`, `2025Q3 - Summit - Crunch Mode Summit [Aug 13]`, `2025Q4 - Content - Shift Happens [Guide]`, `2025Q4 - Customer Marketing - AI Consulting Workshop`, `2025Q4 - Gifting - Customer Gratitude`, `2025Q4 - Lifecycle - AI Video Cross Sell [Customer Marketing]`, `2025Q4 - Lifecycle - Intent Signal: Ad Design Services`, `2025Q4 - Summit - AI Shift`, `2025Q4 - Webinar - Vishal Kapoor - Oct 15`, `2026Q1 - Connect - Seattle [March 25th]`, `2026Q1 - Content - Breakpoint`, `2026Q1 - Content - Misaligned Guide`, `2026Q1 - Field Events - CES Sport Beach (Stagwell)`, `2026Q1 - Gifting - Post-event - Reachdesk`, `2026Q1 - Gifting - Pre event - Reachdesk`, `2026Q1 - Gifting - Seller's Choice - Reachdesk`, `2026Q1 - Summit - Shift March 26th`, `2026Q1 - VIP Dinner - Amazon Seattle [March 11]`, `2026Q1 - VIP Dinner - Boston [March 4th]`, `2026Q1 - VIP Dinner - New York City [Feb 4]`, `2026Q1 - VIP Dinner - New York City [March 5th]`, `2026Q1 - Webinar - Where do I even start? w/ Julia Jaskólska`, `2026Q2 - Conference - Cannes ADWEEK Creative 100 Reception [June 24]`, `2026Q2 - Conference - Cannes Luma House Panel [June 23]`, `2026Q2 - Conference - Cannes Pre-event Meetings [June 22-24]`, `2026Q2 - Conference - Gartner Marketing Symposium Session [Denver June 9]`, `2026Q2 - Conference - Gartner Marketing Symposium [Denver June 9-10]`, `2026Q2 - Connect - San Francisco [May 5th]`, `2026Q2 - Content - AI Reset`, `2026Q2 - Content - AI Reset Checklist`, `2026Q2 - Content - AI Survival Tips for Marketers`, `2026Q2 - Content - Frictionless Future [Creatives]`, `2026Q2 - Content - Leading in the AI Era`, `2026Q2 - Operational - Cannes invitation lists`, `2026Q2 - Summit - Shift June 17th`, `2026Q2 - VIP Dinner - London [April 23]`, `2026Q2 - VIP Dinner - San Francisco [April 7th]`, `2026Q3 - VIP Dinner - London B [July 9th]`, `2026Q3 - VIP Dinner - London [July 9th]`, `2026Q3 - VIP Dinner - Toronto [August 13th]`, `2026Q3 - Webinar - Can Taste be Automated [July 22]`, `AI Readiness Quiz - ToFu Content`, `AI SDR`, `Always On Tech ICP Nurture`, `Amazon Always On Nurture`, `Book a Call - 1-10 Employees`, `Book a Call - Website [New]`, `Customer Marketing- Superside It`, `Guide - AI Guide 2024 - ToFu Content`, `Linkedin Content Downloads - Ad Guide`, `Product Demo`, `SMB Marketing Nurture`, `The Creative Brief Subscription [Blog]`, `Video Solutions Launch - Video Capabilities LP`.
- `Date of last campaign` — DATE
- `Last touched` — DATE
- `Suggested follow-up` — DATE  *(set the property's default reminder to 09:00, day-of, tz Europe/Belgrade — the follow-up alert mechanism)*
- `URL Jiminny` — URL
- `URL LinkedIn` — URL
- `URL Salesforce` — URL
- Checkboxes: `Callback talk track made`, `Chatting?`, `Cold call connected?`, `Connected on LinkedIn?`, `First follow-up sent`, `Had meeting?`, `Inbounded?`, `LinkedIn note on page`, `Second follow-up sent`, `Suggested draft made`

Relations (see §5): `Account` (one-way → All Accounts), `Rep` (one-way → Scoreboard).

## 4. All Meetings  (title property: `Company`)
Plain-settable:
- `Company` — TITLE
- `Channel` — SELECT(`Email`:red, `LinkedIn`:blue, `Cold Call`:default, `Inbound priority`:orange, `Inbound none priority`:purple)
- `Persona/Title` — SELECT(`Creative Director`:brown, `Marketing Director`:green, `CMO`:red, `Business Bob`:gray, `Brand Lead`:purple, `Head of Brand`:orange, `Communications Director`:yellow, `VP of Brand Marketing`:blue, `Head of Creative Operations`:default, `Brand Strategy`:pink, `Head of Marketing`:yellow, `Marketing Operations`:orange)
- `Status` — STATUS, options by group:
  - To-do: `Rejected`:red, `No-show`:red, `Canceled`:red
  - In progress: `Placeholder sent`:orange, `Waiting on time/date`:orange, `Invite not accepted yet`:orange, `Reschedule`:orange, `Booked`:default
  - Complete: `Completed`:green
- `Date of Booking` — DATE
- `Date of Meeting` — DATE (display format `ll`)
- `LinkedIn of Prospect` — URL
- `SF Opp URL` — URL
- `Notes` — RICH_TEXT
- Checkboxes: `Pre-meeting email drafted`, `Pre-meeting email sent`, `SAL`, `SQL`

Relations (see §5): `Account` (DUAL ↔ All Accounts.`Meetings`), `Rep` (DUAL ↔ Scoreboard.`All Meetings 1`), `Name` (one-way → Conversation Hub).

---

## 5. Relation graph (wire after all four DBs exist)
Two-way (create with `DUAL`, one side each):
- **All Accounts.`Meetings` ↔ All Meetings.`Account`**
- **Monthly Scoreboard.`All Meetings 1` ↔ All Meetings.`Rep`**

One-way (single side, `RELATION('target_ds')`):
- All Meetings.`Name` → Conversation Hub
- Conversation Hub.`Account` → All Accounts
- Conversation Hub.`Rep` → Monthly Scoreboard
- All Accounts.`Campaigns` → Conversation Hub
- All Accounts.`Rep` → Monthly Scoreboard
- Monthly Scoreboard.`All Meetings (legacy)` → All Meetings

## 6. All Leads (linked view, NOT a database)
Create a page titled **"All Leads"** under "Databases" with an inline **linked view of Conversation Hub**
(same collection — do not create a new database). View type: table. Show, in order: `Name`, `Rep`,
`Title`, `Account`, `URL Salesforce`, `Campaign`, `Date of last campaign`, `Priority`, `Connected`,
`Last touched`, `Quick notes`, `Sequence`, `Connected on LinkedIn?`, `Chatting?`, `Inbounded?`,
`CL or CW?`. Filters: `Rep` (relation contains), `Account` (relation contains), `Campaign` (multi-select
contains). Sort: `Date of last campaign` descending.

## 7. Formulas (omitted — document for manual re-entry)
Not creatable via the API (expressions aren't API-readable). Leave out of the automated build; note in the
setup report that a team wanting the analytics dashboard must add these by hand in the Notion UI.
- **Monthly Scoreboard:** `Meetings`, `Meetings Booked`, `Meetings Completed`, `Fattest Mogul`,
  `Monthly SQLs Bar`, `Priority SQL Count`, `Total Meetings Booked`, `Total SQL Count`.
- **All Accounts:** `Number of Connects`, `Total Number of Leads`.

## 8. Known API limitations (verified in the bootstrap write-test 2026-07-27)
The DDL build is faithful for everything except these — the bootstrap must flag them in its report as
**manual finish-up steps** for the rep/admin in the Notion UI:
1. **All Meetings `Status` options/groups.** `STATUS` in DDL creates only Notion's default options
   (Not started / In progress / Done). The real custom options — To-do: `Rejected`, `No-show`,
   `Canceled`; In progress: `Placeholder sent`, `Waiting on time/date`, `Invite not accepted yet`,
   `Reschedule`, `Booked`; Complete: `Completed` — **cannot** be set via `create_database` /
   `update_data_source` DDL. After creating the DB, rename/add the status options and assign groups by
   hand (or via a status-capable API call if one becomes available). Everything downstream that reads
   `Status` assumes these exact option names, so this step is required, not optional.
2. **Empty `Sequence` select (Conversation Hub).** A `SELECT()` with no options may be rejected by the
   DDL parser. If so, create `Sequence` as a bare select in the UI (or add a throwaway option and delete
   it) — it intentionally has no options in the source schema.
3. **Formulas.** The `FORMULA('…')` DDL works for **scalar** formulas (verified: `1 + 1` created fine),
   but **any formula that references the `All Meetings 1` relation fails at create time** with
   `"Type error with formula"` (verified: `prop("All Meetings 1").length()` rejected). Every Scoreboard
   formula here walks that relation, so none can be built via the API — they must be pasted into the
   Notion UI's formula editor (which resolves relation types fine). The expressions are stored verbatim in
   §9 so `team-setup` can hand the admin a paste-ready checklist.
Everything else (all other selects/multi-selects incl. their full option lists and colors, checkboxes,
dates, urls, people, text, and the entire relation graph incl. two-way duals) builds correctly via DDL,
as confirmed by the write-test.

## 9. Scoreboard formula expressions (paste into the Notion UI — API can't build these)
Create each as a Formula property on Monthly Scoreboard, then paste the expression. They reference the
`All Meetings 1` relation (present after §5), so they resolve verbatim. **Still needed** (not yet
captured): `Fattest Mogul` (Scoreboard) and `Number of Connects` + `Total Number of Leads` (All Accounts).

**Meetings**
```
lets(
  thisMonthMeetings, prop("All Meetings 1")
    .filter(
      not empty(current.prop("Date of Meeting")) and
      month(current.prop("Date of Meeting")) == month(now()) and
      year(current.prop("Date of Meeting")) == year(now())
    ),
  companyList, thisMonthMeetings
    .map(current.prop("Company"))
    .filter(not empty(current))
    .unique(),
  if(thisMonthMeetings.length() == 0,
    "No meetings this month",
    format(thisMonthMeetings.length()) +
    if(thisMonthMeetings.length() == 1, " meeting: ", " meetings: ") +
    companyList.join(", ")
  )
)
```

**Total Meetings Booked**
```
prop("All Meetings 1")
  .filter(
    not empty(current.prop("Date of Booking")) and
    month(current.prop("Date of Booking")) == month(now()) and
    year(current.prop("Date of Booking")) == year(now())
  )
  .length()
```

**Meetings Booked** (LinkedIn / Cold Call / Email only)
```
prop("All Meetings 1")
  .filter(
    ["LinkedIn", "Cold Call", "Email"].includes(current.prop("Channel")) and
    not empty(current.prop("Date of Booking")) and
    month(current.prop("Date of Booking")) == month(now()) and
    year(current.prop("Date of Booking")) == year(now())
  )
  .length()
```

**Meetings Completed**
```
prop("All Meetings 1")
  .filter(
    current.prop("Status") == "Completed" and
    not empty(current.prop("Date of Meeting")) and
    month(current.prop("Date of Meeting")) == month(now()) and
    year(current.prop("Date of Meeting")) == year(now())
  )
  .length()
```

**Priority SQL Count** (SQL, not Inbound none priority)
```
prop("All Meetings 1")
  .filter(
    current.prop("SQL") and
    current.prop("Channel") != "Inbound none priority" and
    not empty(current.prop("Date of Meeting")) and
    month(current.prop("Date of Meeting")) == month(now()) and
    year(current.prop("Date of Meeting")) == year(now())
  )
  .length()
```

**Total SQL Count**
```
prop("All Meetings 1")
  .filter(
    current.prop("SQL") and
    not empty(current.prop("Date of Meeting")) and
    month(current.prop("Date of Meeting")) == month(now()) and
    year(current.prop("Date of Meeting")) == year(now())
  )
  .length()
```

**Monthly SQLs Bar**
```
lets(
  outboundSQLCount, prop("All Meetings 1")
    .filter(
      current.prop("SQL") and
      current.prop("Channel") != "Inbound" and
      not empty(current.prop("Channel")) and
      not empty(current.prop("Date of Meeting")) and
      month(current.prop("Date of Meeting")) == month(now()) and
      year(current.prop("Date of Meeting")) == year(now())
    )
    .length(),
  percentage, ifs(
    outboundSQLCount == 0, 0,
    outboundSQLCount == 1, 14,
    outboundSQLCount == 2, 29,
    outboundSQLCount == 3, 43,
    outboundSQLCount == 4, 57,
    outboundSQLCount == 5, 71,
    outboundSQLCount == 6, 86,
    outboundSQLCount == 7, 100,
    outboundSQLCount == 8, 114,
    outboundSQLCount == 9, 125,
    135
  ),
  filledBars, repeat("█", min(outboundSQLCount, 7)),
  emptyBars, repeat("░", max(0, 7 - outboundSQLCount)),
  progressBar, filledBars + emptyBars,
  displayText, progressBar + " " + format(percentage) + "% (" + format(outboundSQLCount) + "/7)",
  if(outboundSQLCount > 7,
    style(displayText, "b", "red"),
    if(percentage >= 100,
      style(displayText, "b", "green"),
      displayText
    )
  )
)
```

## 10. Column order + visibility (per-view — required)
**Column order is a property of the table *view*, not the schema.** A freshly-built database defaults its
columns to alphabetical order (verified in the write-test — the scratch DB showed columns A→Z, nothing
like the source). So after building each database's schema + relations, the bootstrap **must set the
default view's column order** to match the source, via `notion-update-view` with a `SHOW "…"` directive
listing the columns in order (this both orders *and* hides — any property not listed becomes hidden,
which is what we want since the sources hide many columns). Verified working: applying the SHOW list below
to a recreated Conversation Hub view reproduced the source order exactly.

Apply the lists below to each new database's single default view. **Only list properties that already
exist** when you run it — formula columns (§7/§9) don't exist until manually added, so position them by
hand at that point per the target order noted here.

**Conversation Hub** (20 shown; all others hidden):
`SHOW "Name", "Rep", "Title", "Account", "URL Salesforce", "Priority", "Connected", "Last touched", "Quick notes", "Disposition", "Suggested follow-up", "Suggested channel next", "Suggested draft made", "Callback talk track made", "LinkedIn note on page", "First follow-up sent", "Second follow-up sent", "Nutrure?", "Date of last campaign", "Campaign"`

**All Accounts** (11 shown; all others hidden). Note `Number of Connects` + `Total Number of Leads` are
formulas — omit from the automated SHOW, insert at positions 9–10 when added manually:
`SHOW "Account Name", "Rep", "Priority Type", "Prospecting Status", "Last Touch Point", "Needs Attention", "Notes", "Temperature", "Campaigns"`
(target full order incl. formulas: Account Name, Rep, Priority Type, Prospecting Status, Last Touch Point, Needs Attention, Notes, Temperature, **Number of Connects**, **Total Number of Leads**, Campaigns.)

**All Meetings** (14 shown; `LinkedIn of Prospect` + `SF Opp URL` hidden). No formulas, so this is exact:
`SHOW "Company", "Rep", "Name", "Persona/Title", "Date of Booking", "Date of Meeting", "Channel", "Status", "SAL", "SQL", "Notes", "Pre-meeting email sent", "Pre-meeting email drafted", "Account"`
(The source also has month-slice views — July 2026, June 2026, …, and analytics views — that are
Money-Moguls-specific; a new team gets just this one default table. Recreate the extra views only if a
team asks.)

**Monthly Scoreboard** (13 shown in the source; the operational fields are hidden). All but Name/AE/Level/
targets are formulas, so the automated SHOW covers only the five real columns; position the formula
columns per the target order when they're added manually:
`SHOW "Name", "AE", "Level", "Monthly MC target", "Monthly SQL target"`
(target full order incl. formulas: Name, AE, Level, Monthly MC target, Monthly SQL target, **Meetings**,
**Total Meetings Booked**, **Meetings Booked**, **Meetings Completed**, **Priority SQL Count**,
**Total SQL Count**, **Monthly SQLs Bar**, **Fattest Mogul**. Hidden: All Meetings 1, All Meetings
(legacy), Email exclusions, Last Convohub Run, Last email check, Last linkedin check.)
