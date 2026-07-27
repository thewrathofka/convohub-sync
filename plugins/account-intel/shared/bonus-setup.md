# Bonus setup — add the Account Intelligence properties to All Accounts

`/account-intel` needs five properties on the team's **All Accounts** database that the ConvoHub Sync
`/team-setup` may not have created (teams onboarded before Account Intelligence existed). This is a
tiny, **idempotent, additive** step — a "bonus setup" on top of the main team setup. Run it once at the
start of any run that will write scores (i.e. the BDR path), before Step 5's writes.

## What to ensure exists
Resolve the team's All Accounts data source (via `convohub-sync-core`'s `team-resolution.md`), `notion-fetch`
its live schema, and **add only the missing** properties (never duplicate, never alter existing ones) via
`notion-update-data-source` DDL:

| Property | Type | DDL |
|---|---|---|
| `ROI Score` | number | `ADD COLUMN "ROI Score" NUMBER` |
| `Prospecting Readiness` | number | `ADD COLUMN "Prospecting Readiness" NUMBER` |
| `Prospect Now` | select | `ADD COLUMN "Prospect Now" SELECT('NOW!':green, 'why not':yellow, 'whatever':gray)` |
| `Leads (2y)` | number | `ADD COLUMN "Leads (2y)" NUMBER` |
| `Last Intel Run` | date | `ADD COLUMN "Last Intel Run" DATE` |

Combine the missing ones into a single `notion-update-data-source` call. If all five already exist, do
nothing.

## Column order
After adding, extend the All Accounts view's `SHOW` list so the new columns sit together near the front —
insert `ROI Score`, `Prospecting Readiness`, `Prospect Now`, `Leads (2y)` right after `Priority Type`, and
keep `Last Intel Run` with the other date fields. (Non-blocking; skip if the view can't be updated.)

## Also in the canonical spec
These five are added to `convohub-sync-core/shared/notion-setup.md` (schema §2 All Accounts + §10 column
order) so that **freshly** bootstrapped teams get them from the start and this bonus-setup becomes a no-op
for them. Bonus-setup exists only to backfill teams already set up before Account Intelligence shipped.

## Non-BDR / no-write runs
Skip this entirely — non-BDR runs never write to Notion, so they never need the properties.
