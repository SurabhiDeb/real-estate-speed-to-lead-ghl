# Speed-to-Lead + Auto-Drafted Follow-Up System — Real Estate (GoHighLevel)

**A system that catches every new lead the second it lands, scores it, and has a personalized first-touch text and email drafted within two minutes — so the agent just approves and sends instead of losing the deal to whoever replied first.**

> _Built for real estate teams buying leads from Zillow, Realtor.com, Facebook lead ads, and their own IDX site — leads that go cold because nobody replied for four hours. It enriches and scores each lead, drafts a follow-up that references the actual property and inquiry, and routes it through a one-tap approval — while refusing to auto-text anyone who never opted in and never auto-blasting a high-value lead without a human._

**Who it's for:** Real estate teams and solo agents on GoHighLevel who pay for internet leads and lose them to slow response.
**What it replaces:** an agent checking their phone between showings, retyping the same "Hi, saw you were interested in 123 Main St…" from scratch, hours late.
**The guardrail that makes it safe:** it never auto-texts a lead without a captured consent/opt-in, never makes a price or availability commitment, and never auto-sends on a luxury/high-value lead — those always wait for a human.

---

## The Problem

In real estate, speed-to-lead isn't a nice-to-have — it's the whole game. The industry statistic every team lead already knows: the odds of contacting a lead drop roughly 10x after the first five minutes, and the agent who responds first wins the conversation most of the time. Teams pay $20–60+ per lead from Zillow, Realtor.com, and Facebook, and then a huge share of that spend evaporates for one reason: nobody replied fast enough.

The reason isn't laziness. It's that an agent's job is physically incompatible with instant response. They're at a showing, in the car, on another call, asleep. A lead comes in at 9:14pm from a Facebook ad and gets a first text at 8:30 the next morning — by then that buyer has already talked to two other agents. The lead was never bad. The response was late.

So teams try to fix it with a blast: an instant auto-text to every new lead. And that creates its own problems. It fires the same generic "Thanks for your interest, an agent will contact you shortly" to everyone, which converts poorly and reads like a bot. Worse, it auto-texts people who never actually consented to be texted — a TCPA exposure that can mean real fines. And it treats a $2M luxury inquiry exactly like a tire-kicker who typed a fake number into a home-value widget. The blast is fast but dumb; the human is smart but slow.

The real need is a system that is *both*: instant like the blast, judgment-driven like the agent. Something that responds in seconds, but responds *appropriately* — the right message to the right lead, and a deliberate stop before it does anything that a human should own.

---

## What It Does

When a new lead hits GoHighLevel — from a Zillow/Realtor.com ADF feed, a Facebook lead ad, or an IDX website form — the system:

- **Enriches and normalizes** the raw lead: parses the inquiry, the property or search they were looking at, and any stated budget, timeline, or financing detail; standardizes phone/email; and flags what's missing.
- **Scores intent** with an LLM: buyer vs. seller, price band, timeline (now / 3-months / just-looking), financing readiness (pre-approved vs. unknown), and source quality — into a hot / warm / cold score with the reasoning attached.
- **Decides how to respond** — and this is the part that isn't just "send a text." Three things must line up for the system to send on its own: the lead has a captured consent/opt-in for the channel, the score is a clean standard lead, and it isn't flagged high-value or too-ambiguous-to-touch. If all three hold, the drafted first-touch goes out instantly. If any fails, it routes to a human with the draft already written.
- **Drafts a personalized first-touch** — a short SMS and a matching email — that references the *specific* property or search and the inquiry, in the agent's voice, with a clear next step (a question that invites a reply, or a link to book a showing). It never quotes a price, promises availability, or commits to terms, and every SMS carries an opt-out line.
- **Routes to one-tap approval** for anything the system won't send itself: the draft lands in the agent's Slack (or GHL) with the score, the reason it needs a human, and Approve / Edit / Skip. The agent taps once and it goes.
- **Writes back to GoHighLevel** — upserts the contact, sets the lead score and source, moves the opportunity into the right pipeline stage, and logs whether the touch was auto-sent or human-approved, so response time and outcomes are measurable.

A typical fast path: a Facebook lead ad captures a buyer at 9:14pm who ticked the consent box, asked about a $480k listing, and said "pre-approved, looking to buy in 30 days." The system scores it hot, sees valid consent and a clean standard lead, and sends a drafted text at 9:14pm — "Hi Maria, it's Jordan with [Team] — saw you're interested in 14 Oakline Dr and that you're pre-approved. Want me to line up a showing this week? Reply STOP to opt out." The agent wakes up to a conversation already started.

A typical held path: a lead comes in on a $2.4M listing with no phone consent and a vague "just wondering what it's worth" message. The system does **not** text. It drafts a careful reply, tags it *high-value + no-consent*, and drops it in Slack for the agent to personally handle — because a luxury seller lead is exactly the one you don't want a bot cold-texting.

---

## Architecture & Decision Logic

The point of this build isn't "it can send an auto-text." Any tool can blast a lead. The point is that it makes an explicit, defensible decision about **when it's allowed to send on its own** — and deliberately hands off to a human on the leads where an automated touch is a mistake.

**Trigger → orchestration.** A new lead fires an n8n workflow. n8n is the brain: it enriches, scores, applies the send-decision gate, drafts, routes, and writes back to GoHighLevel. The LLM drafts and scores; it does not decide whether to send. The workflow does.

**Enrichment + normalization.** The raw lead payload is messy and inconsistent across sources (Zillow ADF vs. a Facebook form vs. an IDX widget). A normalization step standardizes the fields, pulls the property/search context and any stated budget/timeline/financing, and marks what's missing — so the scorer and the drafter are working from clean, comparable input regardless of source.

**Intent scoring.** An LLM classifies the lead into structured signals — buyer/seller, price band, timeline, financing readiness, source quality — and returns a hot/warm/cold score *with its reasoning*. This is a signal for routing, not the decision itself.

**The send-decision gate (the part that proves judgment).** The system auto-sends the first touch **only if all three hold**: `consent captured for the channel AND score = clean-standard AND not flagged (high-value OR too-ambiguous)`. This is deliberately a multi-condition gate, not a single score threshold, because each condition guards a different failure:

- **No consent** → never auto-text, at any score. TCPA is a legal boundary, not a tuning knob. It routes to a human (who can call, or send email if email consent exists).
- **High-value lead** (luxury price band) → never auto-send, even with consent and a great score. A $2M lead deserves a human's first words. Draft and route.
- **Too little info / ambiguous** → don't fire a confident text at a lead the system can't actually read. Draft and route.
- **Clean, consented, standard** → send instantly. This is the volume the system is trusted with.

This is the "fast where it should be fast, careful where it should be careful" design a team lead needs to see before they'll let automation touch leads they paid for. The system is trusted with the routine majority; humans keep control of anything legally or commercially sensitive.

**Human-in-the-loop draft.** A held lead is not a dead end that dumps a raw lead on the agent. The system still does the work — it drafts the SMS and email, attaches the score and the reason it's being held — so the agent's job is *approve or lightly edit*, not *write from scratch*. That's what keeps the held leads fast to handle instead of just shifting the delay back onto a person.

**Write-back + measurement.** Every lead is upserted to GoHighLevel with its score, source, and pipeline stage, and every touch is logged as auto-sent or human-approved with a timestamp — so speed-to-first-touch and the auto-send/hold split are reportable from day one.

**The demo-to-live swap point.** In the portfolio build, the incoming lead is a mock webhook, enrichment is a local normalization step, approvals post to a test Slack channel, and the GoHighLevel write-back is a documented HTTP call against GHL's API. For a real client, four things swap in with no change to the core logic: (1) their real lead sources pointed at the webhook (or GHL's inbound-lead webhook), (2) their GHL sub-account for contact/opportunity write-back and for actually sending the SMS/email, (3) their approval destination (Slack or GHL's conversation view), and (4) their consent source of truth. The score thresholds, the high-value price band, and the agent's voice are config the client tunes — not hard-coded. Clean seams, so a demo becomes a deployment in days.

---

## Projected Outcomes

_These figures are illustrative targets based on published speed-to-lead benchmarks, not results from a live client deployment — this build is being finished now. They're framed the way I'd model ROI for a real team, and I'll replace them with actual pipeline data as the system goes live._

- **First touch in under two minutes, 24/7** — every paid lead gets a personalized, relevant reply the moment it lands, including nights and weekends, instead of hours later.
- **Higher contact and conversion on leads already paid for** — speed-to-lead research consistently ties sub-5-minute response to multiples-higher contact rates; this pulls first-touch to near-instant on the clean majority.
- **Zero non-consented auto-texts and zero auto-sends on high-value leads** — by design, because those never fire automatically. The compliance and the commercial judgment are built into the gate.
- **Agent time reclaimed** — no more retyping the same first-touch from scratch; the held leads arrive as a one-tap approval instead of a blank text box.
- **Every lead scored, routed, and logged** — speed-to-first-touch, auto-send vs. hold split, and score distribution are measurable from day one, so the team can see exactly what the paid-lead spend is now returning.

The headline claim I'd put on a proposal: **"You're losing deals to whoever replies first. This scores every new lead and has a personalized follow-up drafted within two minutes — you just approve and send. And it won't text anyone who didn't opt in, or cold-blast your $2M leads."**

---

## Adapting to Your Business

The architecture is real-estate-specific in its scoring signals and its high-value boundary, but the pattern — enrich, score, gate the send on consent + value + confidence, draft, and route-or-send — transfers cleanly to any business where speed-to-lead wins deals but a wrong automated touch is costly:

- **Mortgage / lending:** same speed-to-lead urgency; the boundary becomes "never auto-send anything that reads as a rate or approval commitment," routed to a licensed loan officer.
- **Home services / contractors:** enrich and score inbound quote requests; auto-respond to standard jobs, route large or complex jobs to the owner with a drafted reply.
- **B2B agencies / professional services:** the same enrich → score → draft-a-personalized-reply flow, with the high-value boundary set on deal size so enterprise inquiries always get a human first.

In each case the swap is the scoring signals and the high-value boundary — the enrich, gate-the-send, and human-in-the-loop backbone stays the same.

---

## How It's Built

Stack: n8n (orchestration + the send-decision gate) · GoHighLevel (CRM, contact/opportunity write-back, SMS/email sending) · OpenAI or Claude (scoring + drafting) · Slack (one-tap human approval queue)

- **n8n** orchestrates everything and owns the decision: it enriches the lead, calls the LLM to score and draft, computes the consent + value + confidence gate, and routes to either the auto-send branch or the draft-and-approve branch. Self-hostable, so there's no per-workflow platform lock-in. The gate logic lives here, in the open, not buried inside a CRM automation nobody can audit.
- **GoHighLevel** is the client's system of record and the send channel: the workflow upserts the contact, sets the score and pipeline stage, and — on the auto-send path or after approval — sends the SMS/email through the client's GHL sub-account so it's on their number and in their conversation history.
- **The LLM** (OpenAI or Claude) does two narrow jobs: score the lead into structured signals, and draft a first-touch grounded in the actual inquiry — with hard instructions never to quote price, promise availability, or commit to terms, and always to include an opt-out on SMS. It is explicitly not trusted to decide whether to send — that's the workflow's job.
- **Slack** is the human-in-the-loop destination: held leads arrive as a drafted SMS + email plus the score and the reason for the hold, ready to approve, edit, or skip in one tap.

The whole system is built to be importable and configurable: the n8n workflow exports as a JSON file a client can drop into their own instance, and the score thresholds, the high-value price band, the consent rule, and the agent's voice are parameters — not hard-coded — so it adapts to a specific team without touching the core logic.
