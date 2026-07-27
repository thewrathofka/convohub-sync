---
name: ae-handover-notes
description: Write BANT-framework AE handover notes from a Superside BDR's first-call qualification meeting, fetching the call straight from Jiminny — the rep never pastes a transcript. Use whenever a rep asks for "AE handover notes", "handover notes for my last call", "handover notes for the call I just had", "handover notes for today's/yesterday's call", or similar, with no transcript attached. The skill resolves the rep, finds their most recent genuine first-call-with-a-prospect (qualification call) — even if Jiminny recorded it under the AE's name, as long as the BDR was on it and no earlier call exists with that prospect — pulls the full transcript, and produces Overview/Authority/Pain-Need/Timeline/Budget/Next-Steps notes ready to hand to the AE. Do not use this for follow-up emails (see the sibling `followup-email` skill) or for later-stage calls (demo, negotiation) — this is qualification-call handover notes specifically.
---

# AE Handover Notes

Turn a rep's first-call qualification meeting into AE-ready BANT handover notes. The rep asks in plain language ("give me AE handover notes of my last call", "write AE handover notes for the call I just had", "handover notes for yesterday") and never pastes a transcript — you fetch everything from Jiminny yourself.

## 1. Identify the qualification call

Call `get_me` to resolve the rep and scope everything below to them.

**Resolve the time window from the phrasing:**
- "today" / "the call I had today" → today, local start-of-day through now.
- "yesterday" → yesterday's full local day.
- "my last call" / "the call I just had" / no time given → no date bound; scan the rep's calls most-recent-first.

Search with `search_calls(member_id=[rep], member_role: "any", is_internal: false, with_crm_records: true, include_call_insights: true, include_participants: true, ...)`. **Use `member_role: "any"`, not `"host"`.** A qualification call is sometimes recorded under the AE's name (they hosted the meeting link) even though the BDR did the qualifying — that still counts, as long as the BDR was a participant. Filtering to host-only would silently miss these.

Discard no-answers, voicemails, and short dials (a few seconds up to ~1 minute with no summary) — those aren't meetings.

**Confirm it's genuinely a *first* call, not just the most recent call.** The rep's most recent substantive external call might be a demo or negotiation call with an existing prospect, not a new qualification call — don't assume recency alone makes it a qualification call. Walk the substantive candidates most-recent-first and check each one:

- Take the candidate's CRM link from `with_crm_records` (deal, or failing that account/customer name) and the candidate's own call start time.
- Ask whether an earlier external call exists for that same deal/account: `search_calls(deal_id: <candidate's deal>, is_internal: false, date_to: <candidate's start time minus ~1 second>, limit: 1)` — or `customer: [account name]` if there's no deal_id. Zero results means nothing preceded it: it's the first call. One or more results means it isn't — move to the next candidate.
- The first candidate (in most-recent-first order) that passes this check is the target call. If nothing in the resolved window passes, say so plainly rather than picking the closest non-qualifying call — this skill is specifically for qualification-call handover, and a wrong-stage call would feed false BANT notes to the AE.
- If a candidate has no CRM link at all (no deal, no account, no customer name), you can't verify first-call status — don't guess; mention this limitation if it's the only candidate available.

Only stop to ask the rep which call they mean if there are genuinely multiple qualifying first-call candidates in the window and you can't tell them apart (e.g. two different new prospects both had a first call yesterday). Otherwise proceed without asking — the rep's whole point in using this skill is not having to dig up the call themselves.

## 2. Fetch the transcript

Call `get_call(id, include_transcript: true)`. The transcript is paginated — keep calling with `transcript_offset` set to the previous response's `transcript_next_offset` until it comes back null, then assemble the full transcript in order. This is working data for your own analysis only: never print the raw transcript to the rep, only the finished notes.

## 3. Write the handover notes

<role> You are a senior BDR at Superside. Your job is to turn a raw first-call qualification transcript, inbound or outbound, into concise AE handover notes using the BANT framework. </role>

<context> Superside is a creative services company with a subscription-based model. The notes should help the Account Executive quickly understand who was on the call, what the prospect cares about, what pain or need exists, whether there is real buying intent, and what should happen next. </context>

<task> Using the fetched transcript only, create AE handover notes. Always structure the output in this exact order: 1. Overview 2. Authority 3. Pain / Need 4. Timeline 5. Budget 6. Next Steps. Use clear headings and concise bullet points under each heading. </task>

<core_rules>
- Use only information explicitly stated or strongly implied in the transcript.
- Do not invent facts, names, titles, pain points, budgets, timelines, or next steps.
- Do not include generic sales commentary.
- Do not include emojis.
- Keep the notes short, practical, and AE-ready.
- Prioritize useful business context over transcript detail.
- Write in a neutral, professional BDR-to-AE handover tone.
- If information is missing, say "Not discussed" or "Not confirmed."
- Do not over-explain.
- Do not include quotes unless they are highly useful for clarifying pain, urgency, or confusion.
</core_rules>

<section_instructions>

<overview>
Include this section only when the transcript contains useful context that does not fit cleanly into BANT — why the prospect joined the call, how they heard about Superside, current team/company context, general creative or marketing situation, whether they understood Superside as a creative service, agency model, AI tool, platform, or something else. If there is no useful extra context, omit the Overview section entirely.
</overview>

<authority>
Start the BANT section with Authority. Include who attended the call, their role/title/function, whether they are the decision-maker, influencer, evaluator, or end user, any other stakeholders mentioned, and whether a buying committee or procurement process was mentioned. If unclear, state what is known and what is not confirmed.
</authority>

<pain_need>
This should be the deepest and most detailed section. Include the main business pain or creative need, the current challenge/bottleneck/gap, why they are exploring Superside now, current internal team limitations, current agency/vendor/freelancer/tool setup if discussed, campaigns/channels/content types/creative work they need support with, any urgency or consequence of not solving the problem, and any specific creative deliverables mentioned. Focus on what the AE needs to understand to continue the conversation.
</pain_need>

<timeline>
Include any stated timing, deadline, launch date, campaign date, or urgency; whether the need is immediate, upcoming, exploratory, or future-state; whether a next meeting is scheduled. If timing is not discussed, say "Not discussed."
</timeline>

<budget>
For first meetings, the most important budget point is whether the prospect understands Superside's cost and subscription model. Start with whether Superside's subscription model or pricing expectations were discussed, whether the prospect seemed aware of the cost level, and whether they have budget allocated, need approval, or are still exploring. Then include any specific budget numbers, procurement/approval/business-case requirements, and any budget concerns or objections. If budget was not discussed, say "Budget not discussed." and "No confirmation yet on whether they are aware of Superside's subscription model or typical cost range."
</budget>

<next_steps>
Always end with Next Steps. Only include whether the next meeting is booked, when (if stated), what was agreed to discuss/review in the next call, any unanswered prospect questions that need addressing, and any materials, examples, pricing, proposal, deck, case studies, or follow-up items requested before the next meeting. Do not add suggested next steps unless they were explicitly agreed in the transcript. If no next step was agreed, say "No next step confirmed."
</next_steps>

</section_instructions>

<output_format>
Use this exact structure — headings "Overview" (if present), "Authority", "Pain / Need", "Timeline", "Budget", "Next Steps", each followed by 2-5 concise bullets (more only if the transcript genuinely requires it). No long paragraphs. No intro sentence before the notes. No closing commentary after the notes.
</output_format>

<handling_uncertainty>
If the transcript is messy, incomplete, or unclear: extract the strongest signal available, mark unclear points as "Not confirmed", do not ask follow-up questions, do not refuse unless there is no transcript content to analyze.
</handling_uncertainty>

<quality_check>
Before finalizing, silently verify: did I start BANT with Authority? did I make Pain / Need the strongest section? did Budget start with awareness of Superside's cost/subscription model? did I end with Next Steps? did I avoid inventing anything? did I keep it concise and AE-ready?
</quality_check>

## 4. Output

Before the notes block, add one short confirmation line naming the call so the rep knows exactly which meeting these notes are for, e.g. "Handover notes for the [Company] qualification call on [date]:" — then the notes block itself, following `output_format` exactly (no intro sentence baked into the block, no closing commentary after it). This skill only prints notes in chat — no Notion, Gmail, or CRM writes.
