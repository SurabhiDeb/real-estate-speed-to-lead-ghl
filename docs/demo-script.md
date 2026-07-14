# Loom Demo Script — Lead-to-Proposal System (Real Estate / GoHighLevel)

Target length: **75–90 seconds.** Two clips. The hold is the money shot — don't rush it. Record at the n8n canvas with Slack visible in a second window.

---

## Cold open (0:00–0:10)

> "In real estate, the agent who replies first usually wins the deal — and most teams lose leads they paid for because nobody replied for hours. This system replies in seconds. But watch *when it decides not to*."

Show the n8n canvas, left to right, for one beat so the shape is visible.

---

## Clip 1 — the fast path (0:10–0:35)

Fire **Lead A** (hot, pre-approved buyer, consented, $480k).

> "New lead — Facebook ad, pre-approved buyer, asking about a specific listing, and they consented to be texted."

Let it run. Open the `Send-Decision Gate` output.

> "The system enriches it, scores it hot, and checks three things: is there consent, is it a clean standard lead, and is it *not* a high-value or ambiguous lead. All three pass — so it sends."

Cut to Slack `#lead-desk`.

> "Personalized text, out in seconds — names the actual property, one clear next step, and an opt-out. The agent didn't touch it."

---

## Clip 2 — the hold (the money shot) (0:35–1:15)

Fire **Lead B** (luxury $2.4M seller, no consent).

> "Now a $2.4 million seller lead — but they never opted in to texts."

Open the `Send-Decision Gate` output. Point at `decision: human_approval` and `holdReasons: high_value, no_consent`.

> "Here's the judgment. The system refuses to auto-text — two reasons. One, no consent: that's a TCPA line, not a setting I let it cross. Two, it's a high-value lead — a $2M seller deserves the agent's own first words, not a bot. So it does *not* send."

Cut to Slack — the ":hand: Needs you" card.

> "Instead it drops the draft in the agent's queue — already written, with the score and exactly why it was held. The agent approves in one tap. It's still fast. It's just not reckless."

Let that sit for a beat.

---

## Close (1:15–1:30)

> "So: instant on the leads you can safely automate, a human on the ones you can't — consent, luxury, anything it can't read. The scoring lives in the open in n8n, it writes back to GoHighLevel, and the whole thing imports as one file. That's the difference between a lead blast and a system you'd actually trust with the leads you paid for."

End on the canvas.

---

## Notes for the recording

- **Do the hold slowly.** The refusal is what separates this from every "instant auto-text" tool a client has already been burned by. Linger on `holdReasons`.
- **Say "the LLM scores and drafts — the workflow decides whether to send" at least once.** That single line is the systems-thinking signal a good client is listening for.
- Have `#lead-desk` pre-scrolled so the auto-send and the hold cards are both visible without hunting.
- Optional third beat if you have 15 extra seconds: fire Lead C (spam) to show it also holds — proves the gate catches junk, not just luxury.
- Proposal opener to pair with the Loom: *"You're losing deals to whoever replies first. This scores every new lead and has a personalized follow-up drafted within two minutes — you just approve and send. And it won't text anyone who didn't opt in, or cold-blast your $2M leads."*
