# Instant Lead Responder — n8n + AI Agent

A real, working automation for an AI automation agency. It answers leads in
seconds instead of hours — the exact problem Fahad posts about ("the fastest
quote wins the job").

**Live status (Nov 5, 2026): deployed on n8n Cloud and verified end-to-end.**
A hot test lead scored 95 and got its personalised reply in seconds; every
lead is logged in Google Sheets.

## What it does

1. A lead submits a form (or any app POSTs to the webhook): name, email,
   service, message.
2. A Code node (**Build AI Request**) builds the AI request, and an AI agent
   (Groq `qwen/qwen3.8-27b`) qualifies the lead: score 0–100, category
   hot/warm/cold, reason, and a short personalised reply.
3. **Hot leads (score 70+)** get the personalised reply by email instantly.
4. Everyone else gets a polite acknowledgement email.
5. **Every lead is logged** in a Google Sheet with score, category and status.
6. The webhook answers `{"received": true}` so the form can confirm receipt.

## Files

- `n8n-instant-lead-responder.json` — import this into n8n (Workflows →
  three-dot menu → Import from file).

## What you need

- An n8n instance (n8n Cloud or self-hosted).
- A Groq API key (free tier at console.groq.com — no card needed; the
  workflow calls the OpenAI-compatible endpoint
  `https://api.groq.com/openai/v1/chat/completions`).
- A Gmail account connected to n8n.
- A Google Sheet with a tab named **Leads** and this header row:
  `Timestamp | Name | Email | Service | Message | Score | Category | Status`

## Setup (10 minutes)

1. Import the JSON into n8n.
2. **AI Qualify Lead** node: create a Header Auth credential —
   Name: `Authorization`, Value: `Bearer YOUR_GROQ_API_KEY` — and select it.
   (The **Build AI Request** Code node builds the request body in JavaScript;
   do not hand-edit the HTTP node's body — `{{ }}` expressions inside a JSON
   body are not evaluated by n8n and reach the model as literal text.)
3. Both Gmail nodes: select your Gmail credential.
4. Both Google Sheets nodes: select your Google Sheets credential and pick
   your sheet from the list (the workflow logs to the **Leads** tab).
5. Click **Execute Workflow** once with the test payload below, check the
   sheet row and the emails, then flip the workflow **Active**.

## Test it

Replace `YOUR-N8N-URL` with your instance's webhook URL (production URL
appears after activation; the test URL works while you are testing):

```bash
curl -X POST "https://YOUR-N8N-URL/webhook/lead" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Ahmed Raza",
    "email": "ahmed@example.com",
    "service": "AI chatbot",
    "message": "We get 200+ customer messages a day and my team cannot keep up. Can an AI agent answer them?"
  }'
```

A hot lead should receive the AI-drafted reply within seconds and appear in
the sheet with its score and category.

## Honest limits

- Verified end-to-end on n8n Cloud (Nov 5, 2026): a hot test lead scored 95
  and received the personalised reply; cold leads receive the
  acknowledgement; all rows log with full fields.
- On import, n8n will ask you to re-link every credential (Groq Header Auth,
  Gmail, Google Sheets) — that is normal and takes a minute each.
- The AI reply is a draft: it never invents prices or timelines, and it
  promises a human follow-up, not a closed deal.
- If you change the score threshold, edit the **Is Hot Lead?** node (70).
- The deployed instance runs on an n8n Cloud trial (14 days, 1,000
  executions); on expiry the Cloud workspace is deleted unless upgraded.
  This JSON is the backup — re-import it anywhere.

## Why it fits Fahad's profile

It uses exactly his three services together: **n8n** for the workflow,
an **AI agent** for qualification and replies, and it is the kind of system
his posts sell — instant response, no missed leads, everything logged.
