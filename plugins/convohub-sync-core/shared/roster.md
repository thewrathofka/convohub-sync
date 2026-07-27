# BDR roster (bundled)

Authoritative map of BDR → pod (manager) → segment, used by `team-resolution.md` to resolve which team's
Notion databases a running rep belongs to. Ships inside the package; update it here and republish when
the org changes. **Team names** are filled in per pod at first-run bootstrap (see `team-resolution.md`
Step 5 / `skills/team-setup`); only Money Moguls is known today. Match a running rep to a row by name
(and Superside email `{first}.{last}@superside.com`) — fuzzy on obvious variants, never a wrong-person
match.

| Pod (manager) | Segment | Team name | BDRs |
|---|---|---|---|
| Raphael Smith | Enterprise | **Money Moguls** | _(already set up — excluded from bootstrap; MM BDRs resolve via the existing Money Moguls page)_ |
| Chris Masbernat | Mid-Market | **Sales Sharks** | Eréndira Del Carmen Pita Jimenez · Nencho Peltekov · Robert Stefan Ivan · Yehia Saleh |
| Juan Sebastian Aranguren Sanchez | Enterprise | **Money Lynx** | Alvaro Andres Villavicencio Espinoza · Felipe Jesus Chamorro Mantica · Matias Ulrich · Nicolas Ortiz Mendez · Samuel Güther Duarte · Stefan Džakula |
| Sam Schaut | Enterprise | **Pipeline Pirates** | Alexandra Sanoubane · Bruno de Paula · Gabriela Moncada Madriñán · Hanna Borg · Jose David Belalcázar Bonelo · Ximena Servin Enriquez · Yasmine Arab · Yohann Sébastien Doppia |
| Carla Goliath | SMB | **Wealthy Warriors** | Caio Lopes · Emin Hodzic · Nikolay Spasov |

## Notes
- A rep's **AE** is not in this roster — it lives on the rep's Monthly Scoreboard row (`AE` person
  field), asked at bootstrap and lazy-filled later (see `notion-setup.md`).
- **BU Expension eligibility** (see `salesforce-import.md`): only **Katarina Misovic**, **Alvaro Andres
  Villavicencio Espinoza**, and **Gabriela Moncada Madriñán** import `BU Expansion Priority` / `BU Focus 5`
  accounts. They sit in three different pods (Money Moguls / Juan Sebastian / Sam Schaut), so this is a
  per-rep flag, not a per-team one.
- A rep not found here → `team-resolution.md` asks them once which pod/team they're on and proceeds.
