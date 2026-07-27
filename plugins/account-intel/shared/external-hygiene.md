# External research + hygiene (recreated from the Superside extension)

This is the **external** half of `/account-intel`: current-company research + CRM hygiene. The prompts
below are **recreated** from the Superside Assistant Chrome extension (`~/Downloads/superside-extension`)
— we reproduce its output *structure*, not its code, and **never** read or reuse its hardcoded API key.
Here the research runs on **Claude Code's built-in `WebSearch`/`WebFetch`** (no external API key, no
Anthropic-API call), and the output is written to the team's Notion — the account page block + the All
Accounts hygiene properties — instead of Salesforce/clipboard.

Resolve tools by name: `WebSearch`, `WebFetch`; Notion `notion-fetch`, `notion-update-page`,
`notion-create-pages`, `notion-update-data-source`. Account + team databases come from
`convohub-sync-core`'s `team-resolution.md` (resolved by the orchestrator before this runs).

## Freshness
If the account's `Last Intel Run` is **< 3 days** old, reuse the existing `## External Research` block
already on the page (don't re-search). Otherwise run the two passes below, then set `Last Intel Run` = today.
Keep web searches lean (≤3 targeted searches per pass), begin output immediately at the first header, no
process narration.

## Pass A — CRM Hygiene → writes All Accounts properties + the Corporate Structure/Profile sections
Research the company and return **exactly** this structure:

```
## 🏢 Company Profile

| Field | Suggested Value | Confidence |
|-------|----------------|------------|
| Legal Name |  | High/Medium/Low |
| HQ Country |  |  |
| HQ City |  |  |
| Employee Count |  |  |
| Annual Revenue (est.) |  |  |
| Industry (Salesforce category) |  |  |
| Company Type | Public / Private / Non-profit / Government |
| Founded |  |  |
| Website |  |  |

## 🏗️ Corporate Structure
- **Parent Company:** [name or "Independent"]
- **Key Subsidiaries:** [list or "None identified"]
- **Recent M&A activity:** [any acquisitions or mergers in the last 2 years]

## 🔍 Data Sources Used
[sources]

## ⚠️ Data Conflicts or Uncertainties
[flag anything low-confidence so the rep can verify]
```
Use current web data; if a value can't be found with reasonable confidence, write "Verify manually."

**Write-back to All Accounts** (only confidently-determined values; never guess an identity/structure):
- `Parent` ← parent company name (its own name if it is itself a parent / has child brands; else empty).
- `sister/child` ← comma-joined subsidiary/sister brands.
- `Structure Notes` ← recent structural news in the constrained vocab (`recent IPO`, `about to IPO`,
  `merged with X`, `acquired X`, `acquired by X`, `mass layoffs`, `bankruptcy`, `split from X`,
  `buying-frozen`, `buying-friendly`, `out of business`); append with ` · ` if a note already exists.
- `Company Structure` ← a one-line plain-text structure descriptor (this field exists but was unused; we
  populate it with the hygiene structure summary).
- `Size` ← bucket the employee count into `<1000` / `1000-2000` / `2000-5000` / `5000+`.
- Anything genuinely noteworthy (a fresh acquisition, layoffs, IPO, agency switch) is **flagged inline**
  in the page block's Caveats, and — if it's a transition worth a human's eye — added to `Needs Attention`.
Do **not** touch `Cluster`, `Strategy`, `Buying Intent`, `Prospecting Status`, `Temperature`,
`Priority Type`, `Rep`, or `Account Name` — those are BDR/manual-owned.

## Pass B — Account Research → the discovery-prep sections (page block only)
Superside context for the model: *Superside is the AI-powered creative service that acts as an
enterprise's "creative team's creative team" — flexible subscription access to world-class creative
talent enhanced by AI workflows, serving 500+ top brands (brand design, video, motion, performance-
marketing creative, and more); it competes with in-house teams, agencies, freelance platforms (Toptal,
99designs) and tools like Canva for Teams.*

Return **exactly** this structure (start at the first header, ≤3 searches, 2–4 sentences/bullets each):
```
## 🔍 Company Snapshot
## 📣 Marketing & Creative Signals
## ⚠️ Caveats & Watch-Outs
## ✅ Why Superside Could Win Here
## 💬 Suggested Discovery Questions
## 🏷️ Competitor Tools to Watch For
## 🌐 Sources
```
`## 🌐 Sources` lists every page consulted as `- [Title](URL)`.

## Feeding the score
The **External Triggers** sub-score (see `scoring.md`) is derived from *this* research — specifically the
`Marketing & Creative Signals`, `Caveats`, and the hygiene `Structure Notes` / M&A. A fresh (~90-day)
buy-now trigger + clear creative need = 5; negative structural news (bankruptcy / buying-frozen / out of
business) = 1 and caps the verdict at **whatever**. Because the sub-score reads the same research written
to the page, the number and the narrative always agree.

## Output placement
This whole block is the **`## External Research`** section of the account page, written **below** the
`## Internal Intelligence` section (which is below the untouched `## Conversations` section). See the
orchestrator (`skills/account-intel/SKILL.md`) for exact placement and the BDR-vs-non-BDR rule (BDRs get
the page write; non-BDRs get this same content in chat only, and no All Accounts write-back).
