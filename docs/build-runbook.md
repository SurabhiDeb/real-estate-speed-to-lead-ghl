# Build Runbook — Lead-to-Proposal System (Real Estate / GoHighLevel)

Goal: get the demo running end-to-end so you can record the Loom and attach it to the case study. Target build time: ~2–3 focused hours. The demo needs only **OpenAI + Slack** — GoHighLevel is stubbed so you can prove the whole system without a paid GHL sub-account.

---

## 0. What you're proving

A lead lands → it gets enriched, scored, and drafted in seconds → clean consented leads auto-send, everything risky routes to a human with the draft ready. The two demo moments that sell it:

1. **Fast path:** a hot, consented lead → drafted text auto-sent in seconds.
2. **The hold:** a $2M no-consent lead → the system refuses to auto-text, and drops a ready-to-approve draft in Slack with the reason. **This is the money shot.**

---

## 1. Prereqs

- n8n (cloud trial or self-hosted via Docker — either is fine).
- An OpenAI API key.
- A Slack workspace you can add an app to, with a channel `#lead-desk`.
- (Optional, for the live version only) a GoHighLevel account with API access.

---

## 2. Import the workflow

1. In n8n: **Workflows → Import from File →** `l2p-n8n-workflow.json`.
2. You'll see 14 nodes and two sticky notes. Read the purple sticky ("How the gate works") — it's the narration spine for your Loom.

---

## 3. Wire credentials (demo = two creds)

**OpenAI** (on `LLM · Score Lead` and `LLM · Draft First-Touch`)
- Create an *OpenAI* credential with your API key; select it on both nodes.

**Slack** (on `Slack · Human Approval Queue` and `Slack · Log Auto-send`)
- Create a Slack app, add bot scopes `chat:write` (and `chat:write.public` if posting to a channel the bot isn't in), install to workspace, copy the Bot User OAuth token into an n8n *Slack API* credential.
- Invite the bot to `#lead-desk`.

**GoHighLevel** (on the two GHL nodes) — **skip for the demo.**
- Disable both GHL nodes for now: select each → **Disable** (or set them to continue-on-fail). The Slack messages already show what would have sent, so the demo runs clean without GHL.

---

## 4. Seed the demo leads

You'll POST mock leads to the webhook. First, get the **Test URL**: open `New Lead (Webhook)` → copy the Test URL, and click **Listen for test event**.

Send these three (curl, Postman, or n8n's built-in test). They're chosen to trigger all three routing outcomes.

**Lead A — hot + consented → should AUTO-SEND**
```bash
curl -X POST '<TEST_WEBHOOK_URL>' -H 'Content-Type: application/json' -d '{
  "name": "Maria Alvarez", "phone": "(555) 201-3345", "email": "maria@example.com",
  "source": "Facebook Lead Ad", "sms_consent": true, "email_consent": true,
  "property_or_search": "14 Oakline Dr, listed $480k",
  "inquiry_message": "Is this still available? I am pre-approved and want to move in the next month.",
  "budget": "450-500k", "timeline": "0-3 months", "financing_note": "pre-approved"
}'
```

**Lead B — luxury + NO consent → should HOLD (high_value + no_consent)**
```bash
curl -X POST '<TEST_WEBHOOK_URL>' -H 'Content-Type: application/json' -d '{
  "name": "Preston Vale", "phone": "(555) 999-1000", "email": "",
  "source": "Realtor.com", "sms_consent": false, "email_consent": false,
  "property_or_search": "88 Ridgecrest Estate, listed $2.4M",
  "inquiry_message": "Just wondering what my place might be worth if I sold.",
  "budget": "", "timeline": "", "financing_note": ""
}'
```

**Lead C — vague / low-quality → should HOLD (cold_or_low_quality / too_ambiguous)**
```bash
curl -X POST '<TEST_WEBHOOK_URL>' -H 'Content-Type: application/json' -d '{
  "name": "asdf", "phone": "000", "email": "notreal",
  "source": "IDX form", "sms_consent": false, "email_consent": false,
  "property_or_search": "general inquiry", "inquiry_message": "info",
  "budget": "", "timeline": "", "financing_note": ""
}'
```

---

## 5. Verify each path

Run Lead A, B, C one at a time and confirm:

| Lead | Gate `decision` | Slack result |
|---|---|---|
| A (hot, consented) | `auto_send` | ":white_check_mark: Auto-sent first touch" with a personalized SMS referencing 14 Oakline Dr |
| B (luxury, no consent) | `human_approval`, reasons `high_value, no_consent` | ":hand: Needs you" with a careful drafted reply — **nothing auto-sent** |
| C (vague/spam) | `human_approval`, reasons `cold_or_low_quality, too_ambiguous` | ":hand: Needs you" with the read explaining why |

Open the `Send-Decision Gate` node output on each run to show the `decision` and `holdReasons` on camera — that's the auditable judgment.

Also confirm on Lead A that the SMS **ends with an opt-out** even if the model forgot — the `Enforce Opt-out + Validate Draft` node adds it. Good thing to point at in the Loom.

---

## 6. Going live (for a real client — after the demo)

Swap seams, no core-logic changes:

1. **Lead sources → webhook.** Point Facebook Lead Ads, Zillow/Realtor.com ADF, or the IDX form at the **Production URL**. Or subscribe to GoHighLevel's inbound-lead webhook and map its payload to the expected fields.
2. **Enable the GHL nodes.** Add the GoHighLevel Bearer-token credential (`Authorization = Bearer <token>`, header `Version: 2021-07-28`). Fill `locationId` (the sub-account) in the upsert node; use the `contactId` returned by the upsert in the send node. Confirm the sending number/inbox is the client's.
3. **Consent source of truth.** Make sure the `sms_consent` / `email_consent` fields are actually populated by the client's capture (form checkbox, portal consent). This is the legal boundary — don't fake it.
4. **Tune config.** Luxury cutoff (scorer prompt / market), which scores auto-send, agent name/team/voice, booking link.
5. **Approval surface.** For a real team, upgrade the Slack post to interactive Approve/Edit/Skip buttons, or use GHL's conversation view as the approval surface so agents work where they already live.
6. Set the workflow **Active**.

---

## 7. What to capture for the case study

- A screen-recording of Lead A auto-sending and Lead B being held (the two clips are the Loom).
- A screenshot of the `Send-Decision Gate` output showing `decision` + `holdReasons` — proof the logic is explicit and auditable.
- Once live with a pilot client: real speed-to-first-touch numbers and the auto-send/hold split, to replace the illustrative figures in the case study.
