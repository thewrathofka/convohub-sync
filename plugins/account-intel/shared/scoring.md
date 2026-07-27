# Scoring — ROI Score + Prospecting Readiness

Two complementary scores, both shown in the page `### Account Intel Score` block and stored as All
Accounts properties. **ROI = fit/value ("should we sell here").** **Readiness = timing/urgency ("should we
work it NOW").** Sub-scores are 1–5; state the 1-line evidence behind each sub-score in the output, and
never invent a value — if a signal is unknown, score it conservatively (usually 2–3) and say so.

Field sources come from the Salesforce Account pull (see `internal-history.md`) + the external research
(see `external-hygiene.md`).

## Score 1 — ROI Score (0–100)  — Kali's formula, verbatim weights
`ROI = CreativeFit×7 + Trigger/Timing×6 + Reach/Speed×4 + Size/ICP×3`  (each sub-score 1–5 → max 100).

Sub-score inputs (assign 1=Low … 5=High):
- **Creative Fit** — does this account need creative at scale? SF `Rev_Creative_scaling__c`,
  `Rev_Overwhelmed_creative_team__c`, `Rev_High_content_volume__c`, `Rev_Large_ad_function__c`,
  `Rev_Uses_freelancers__c`, `Rev_Frequent_product_launches__c`, `Social_Content_Strategy__c`, `Industry`
  + the external `📣 Marketing & Creative Signals`.
- **Trigger/Timing** — is now a favorable moment? `Rev_Recent_funding__c`,
  `Rev_Recent_key_hire_creative__c`, `Rev_Recent_key_hire_C_level__c`, `Rev_Growth_rate__c`,
  `Unify_Intent_Level__c`, `accountBuyingStage6sense__c`, engagement-recency (`LastActivityDate`,
  `DB_Sales_Engagement_Minutes_30_days__c`) + external triggers.
- **Reach/Speed** — can we get in fast? `Aggregated_Engagement_Score__c`,
  `fibbler_linkedin_engagements_30_days__c`, persona coverage (`Decision_Maker_Buyer_Persona__c`,
  `Champion_Persona__c`), active-contact count + distinct `Task.WhoId`, existing `Type`
  (Active/Former Customer warms this).
- **Size/ICP** — `NumberOfEmployees`, `AnnualRevenue`, `Company_Size__c`, `Industry`,
  `Internal_Vertical__c`, and ICP/priority flags `Account_Tier__c`, `ABM_Tier__c`, `Priority_Account__c`,
  `Strategic_Logo__c`, `Rev_Score__c`/`Rev_Tuned_Score__c`.
(Caveat: many `Rev_*`/`DB_*`/6sense/Unify fields carry a `z_`-prefixed label and are typed as strings —
read the actual value before thresholding; treat null as unknown, not zero.)

→ writes All Accounts **`ROI Score`** (number 0–100).

## Score 2 — Prospecting Readiness (0–100) → `Prospect Now` verdict
Composite of **external hotness + internal engagement**. Sub-scores (1–5):
- **External Triggers** (from `external-hygiene.md`'s research): **5** = a fresh (~last 90d) buy-now
  trigger AND a clear creative need (new funding, rebrand/campaign launch, new CMO/VP Brand-or-Creative
  hire, heavy product-launch cadence, agency switch, major AI/creative-scaling initiative, new-market
  expansion); **4** = a solid current signal (active creative hiring, high content/ad volume, recent
  relevant leadership change); **3** = steady creative fit, no fresh trigger; **2** = faint signal;
  **1** = none, or **negative** context (buying-frozen, mass layoffs w/o overflow angle, bankruptcy, out
  of business).
- **Recency** of the freshest positive *internal* signal: 5=≤30d · 4=1–3mo · 3=3–6mo · 2=6–12mo · 1=12–18mo.
- **Revivable conversations** = # of "Conversations Dropped": 5=3+ · 3=2 · 2=1 · 1=0.
- **Lead footprint** = distinct people engaged in 2y + open MQLs: 5=10+ · 4=5–9 · 3=2–4 · 2=1 · 1=0.

**Raw** = `External Triggers×5 + Recency×5 + Revivable×3 + Lead footprint×3` (max 80) → normalize `/80×100`.

**Closed-lost modifier** — keys off the **validated** closed-lost verdict from `internal-history.md` §4
(the reason cross-checked against the Jiminny call + emails), **not** the raw `Loss_Reason__c`:
- **Reopenable** (validated: post-close positive activity, unresolved warm thread, re-engageable reason,
  or a new external trigger — even if the raw field looked negative) → **×1.1** (a dropped-but-warm loss
  is an *opportunity*, so it lifts readiness).
- Re-engageable reason, no contradicting signal — `Timing / Reconnect Later`, `No Budget / Lost Funding`,
  `Merge (Stage 0)` → **×1.0**
- Neutral — `Price`, `Commercial Reasons`, `Other`, `Unable to discover loss reason` → **×0.9**
- **Confirmed** disqualifying (validated against the call) — `Hard Disqualification`, `Product
  Limitation`, `Rejected AI-Powered Service Model`, `Legal issues` → **×0.6**
- No closed-lost → **×1.0**
(cap the final score at 100 after the modifier.) If the reason couldn't be validated (no Jiminny/Gmail,
e.g. a non-BDR run), fall back to the raw-field bands above and flag it as unverified.

**Overrides:**
- External Triggers = 5 → **floor** the verdict at **why not** even if internal signals are thin.
- Negative structural news (bankruptcy / out of business / buying-frozen) → **cap** the verdict at
  **whatever**, regardless of the number.

**Verdict bands → `Prospect Now`:** ≥70 = **NOW!** · 40–69 = **why not** · <40 = **whatever**.

→ writes All Accounts **`Prospecting Readiness`** (number 0–100) + **`Prospect Now`** (select
NOW!/why not/whatever). Show both scores with their sub-score breakdown + the closed-lost modifier applied
so the number is auditable.

## Per-person temperature (for "Conversations Dropped" + revival Priority)
Independent of the account-level scores; tags each dropped person by the age of their last positive signal:
- **HOT** — ≤ 30 days  → Conversation Hub `Priority` = **High**
- **WARM** — 1–3 months → `Priority` = **Medium**
- **COOL** — 3–6 months → `Priority` = **Low**
- **DORMANT** — 6–18 months → `Priority` = **Low**
Strength bump: a "meeting-held-then-silence" or an explicitly stated interest ranks above a single reply
at the same age. HOT items lead the `### Conversations Dropped` list and each gets a drafted follow-up;
they're the primary revival candidates (see `SKILL.md`).
