# Speed-to-Lead + Auto-Drafted Follow-Up System — Real Estate (GoHighLevel)

**Catches every new lead the second it lands, scores it, and has a personalized first-touch text and email drafted within two minutes — so the agent just approves and sends instead of losing the deal to whoever replied first.**

Built for real estate teams buying leads from Zillow, Realtor.com, Facebook lead ads, and IDX sites — leads that go cold because nobody replied for hours.

`n8n` `GoHighLevel` `OpenAI or Claude` `Slack`

---

## The problem

The odds of contacting a lead drop roughly 10x after the first five minutes, and whoever responds first usually wins the deal. Teams pay $20–60+ per lead and then lose the return because nobody replied fast enough — agents are at showings, in the car, asleep. The "fix" most teams reach for — auto-blast every lead — creates new problems: generic messages that convert poorly, texts sent without consent (TCPA exposure), and a $2M inquiry treated exactly like a tire-kicker.

## What it does

- Enriches and normalizes the raw lead (property/search context, budget, timeline, financing) regardless of source
- Scores intent with an LLM: buyer/seller, price band, timeline, financing readiness, source quality → hot/warm/cold with reasoning
- **Auto-sends the first touch only if all three hold:** consent captured for the channel **and** score is clean-standard **and** not flagged high-value or too-ambiguous
- Drafts a personalized SMS + email referencing the actual property/inquiry — never quotes price, promises availability, or commits to terms; every SMS carries an opt-out
- Routes anything that fails the gate to one-tap human approval (Slack/GHL) with the draft, score, and hold reason attached
- Writes back to GoHighLevel: upserts contact, sets score/pipeline stage, logs auto-sent vs. human-approved

## Architecture

```
New lead (Zillow/Realtor.com ADF, Facebook lead ad, IDX form)
        │
        ▼
   n8n trigger ──► enrich & normalize (messy multi-source → clean fields)
        │
        ▼
   LLM scores intent → hot/warm/cold + reasoning
        │
        ▼
   Send-decision gate — ALL THREE must hold:
     • consent captured for the channel?
     • score = clean-standard?
     • NOT flagged (high-value OR too-ambiguous)?
        │
        ├─ ALL YES ──► LLM drafts SMS + email ──► auto-send ──► GHL write-back
        │
        └─ ANY NO  ──► LLM drafts SMS + email ──► Slack: Approve / Edit / Skip ──► GHL write-back
                         (no consent → never auto-text, ever · high-value → always a human's first words ·
                          too ambiguous → don't fire a confident message at a lead the system can't read)
```

**The gate is deliberately multi-condition, not a single score threshold** — consent is a legal boundary, high-value is a commercial judgment call, and each guards a different failure mode independently.

## Mock vs. live

| | Demo (this repo) | Client deployment |
|---|---|---|
| Lead source | Mock webhook | Real lead sources → webhook (or GHL's inbound-lead webhook) |
| CRM / send channel | Documented HTTP call against GHL's API | Client's GHL sub-account (real SMS/email sending) |
| Approval destination | Test Slack channel | Slack or GHL's conversation view |
| Consent source of truth | Sample flag | Client's actual consent record |
| Score thresholds, high-value price band, voice | Sample config | Config the client tunes |

## Projected outcome

*Modeled from published speed-to-lead benchmarks — this build is pre-deployment, figures will be replaced with real pipeline data as it goes live.*

First touch under two minutes, 24/7 · higher contact/conversion on leads already paid for · zero non-consented auto-texts and zero auto-sends on high-value leads by design · agent time reclaimed from retyping first-touch messages · every lead scored, routed, and logged from day one.

## What's in this repo

```
workflow/n8n-workflow.json   importable n8n workflow (14 nodes: enrich → score → gate → draft → send/hold → write-back)
docs/case-study.md           full problem → architecture → outcome writeup
docs/agent-prompts.md        scoring + drafting prompts (hard rules: no price/availability/term commitments)
docs/build-runbook.md        step-by-step setup/deploy instructions
docs/demo-script.md          walkthrough script used for the recorded demo
```

## Adapting this pattern

The enrich → score → gate-the-send-on-consent+value+confidence → draft → route-or-send pattern transfers to mortgage/lending (boundary: never auto-send anything reading as a rate/approval commitment), home services (auto-respond to standard jobs, route large/complex ones), and B2B/professional services (high-value boundary set on deal size).

---

Built by Surabhi Deb as part of an AI automation portfolio. [See the full project index](https://github.com/YOUR-USERNAME) for related builds (voice lead qualification, RAG support agents, financial reporting automation).

## License

MIT — see [LICENSE](LICENSE).
