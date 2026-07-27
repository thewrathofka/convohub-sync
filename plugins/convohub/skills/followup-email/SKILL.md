---
name: followup-email
description: Write a post-call follow-up email for a Superside BDR/AE by fetching the call straight from Jiminny — the rep never pastes a transcript. Use whenever a rep asks to "write a follow-up email", "follow up on my call", "draft a follow-up for [company]", "follow-up for today's call", "the call I just had", or similar (even just a company name plus "follow up", or a vague "today's call"). The skill finds the call in Jiminny itself, enriches it with the wider account history, drafts in the rep's voice, and creates a Gmail draft (never sends). Not for pre-meeting emails (see meeting-emails) or BANT handover notes (see ae-handover-notes).
---

# followup-email

Draft a post-call follow-up email and leave it as a Gmail **draft** for the rep — never send. Self-
contained: everything comes from Jiminny + Gmail; the rep never pastes a transcript. Resolve MCP tools by
name (Jiminny: `get_me`, `search_calls`, `get_call`; Gmail: `create_draft`).

## Steps
1. **Resolve the rep.** `get_me` → name, email, member_id, timezone.
2. **Find the call.** From the rep's phrasing (a company name, "today's call", "the call I just had", "my
   last call"), `search_calls` for the rep's recent calls and pick the intended one. If a company was
   named, match on it; if only a time reference, take the most recent genuine prospect call in that
   window. If two plausibly match, ask which — never guess the wrong call.
3. **Read it.** `get_call` (with transcript if needed, paginating via the transcript offset) for the
   summary, participants (names + real emails), discussion topics, action items, and any CRM/account
   context. Capture: what was actually discussed, agreed next steps, a stated channel/timing preference,
   any content ask, and any gift/event mention.
4. **Classify the intent** to set the tone — Strong (clear next step / buying signal) / Warm (engaged,
   exploratory) / Lukewarm (polite, non-committal). Never invent enthusiasm the call didn't have.
5. **Draft in the rep's voice** — short, warm, specific to what was said (2–5 sentences): thank them,
   reference one concrete thing from the call, restate the agreed next step or propose one, and offer
   real value only if it fits (a specific, named Superside asset/event — never an invented one). No
   fabricated details, no em dashes, no filler.
6. **Create the Gmail draft** with `create_draft` to the prospect's real email (thread it with
   `replyToMessageId` if a prior thread is known). **Never send.** Skip creating a duplicate if a
   not-yet-sent draft for this call already exists.
7. **Report** the classification, who it's to, and the drafted text so the rep can review before sending.

## Rules
- Draft only — there is no send step. The rep reviews and sends themselves.
- Grounded strictly in the call. If something's uncertain (which call, the prospect's email), ask rather
  than guess.
- Chat + Gmail draft only — this skill does not write to Notion or Salesforce.
