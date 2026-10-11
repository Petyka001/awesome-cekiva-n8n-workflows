# 02 · CEKIVA Core Publisher

> A protected webhook that validates a post and publishes it through CEKIVA, answering with a clean `SUCCESS` or a structured `ERROR`. Put any trigger in front of it.

---

## 1. Who is this workflow for?
- **Target Audience:** Developers, technical marketers and agencies that want one publishing endpoint behind forms, sheets, schedulers, other workflows or AI agents.
- **Experience Level:** Beginner friendly (two credentials, no AI).

## 2. What problem does it solve?
- **The Manual Pain:** Every automation ends up re-implementing the same publishing logic: per-network nodes, expiring Meta tokens, per-account fees, and a different error shape for each service. Ten automations mean ten copies to maintain.
- **The Automated Solution:** One workflow that every other automation calls with a JSON body. It validates the input, resolves the client workspace, makes **one `POST /v1/posts`** to CEKIVA and returns a uniform answer, with the CEKIVA `error.code` and `hint` passed through on failure.

## 3. What do you need? (Prerequisites)
- **Core Platform:** n8n instance (self-hosted or Cloud)
- **Distribution API:** CEKIVA account and API key (`Authorization: Bearer <API_KEY>`, created under *Settings → API keys*). Connect your social accounts once in CEKIVA.
- **External Services & Credentials:** A random secret for the webhook (`X-Webhook-Secret`), created by you.
- **Setup guide:** credentials, the call format and the answers are in [SETUP.md](SETUP.md).
- **Technical Skills:** Basic understanding of importing n8n JSON and setting up credentials.

## 4. Free vs. Paid Components

| Component | Cost Model | Details |
| :--- | :--- | :--- |
| **Workflow File (.json)** | **100% Free** | Open-source under the MIT License. |
| **n8n Engine** | **Free / Variable** | Free if self-hosted (Community edition); paid if you use n8n Cloud. |
| **CEKIVA Account & API Access** | **No separate fee** | No monthly subscription, no per-account fee, no API surcharge, no developer-tier lock. This is not the same as free: publishing is paid, see the next row. |
| **Social Publishing (CEKIVA)** | **Paid: pay-as-you-go credits** | **CEKIVA is not free.** Every published post costs credits (one dispatch per network in the request), billed per post and network; CEKIVA shows the price before you publish and it can change, so check cekiva.com. New accounts can post free during their first 7 days. |
| **AI Generation Engine** | **Variable** | None. This workflow uses no AI. |
| **Third-Party Services** | **Free / Variable** | None. No image host or other service is needed. |

*No prices are quoted here because they can change: see cekiva.com. X (Twitter) is not supported: CEKIVA retired that channel.*

## 5. What you'll get (End-to-End Delivery)
- **Input Data:** A JSON body: `text`, `platforms`, optional `media_urls`, `workspace_id` and `scheduled_at`.
- **Automated Processing:** Header-secret check, validation of platforms and media, optional client-workspace lookup, scheduling, and the CEKIVA dispatch.
- **Final Output:** A JSON answer: `status: SUCCESS` with the created post ids and credits charged, or `status: ERROR` with `kind`, `message` and `hint`.
- **Your Remaining Task:** Choose what to send and decide what your calling system does with a failure.

## 6. A Real-World Example
- **Step 1: Input:**  
  `curl -X POST https://YOUR-N8N/webhook/cekiva-publish -H "X-Webhook-Secret: …" -H "Content-Type: application/json" -d '{"text":"Our spring collection is live!","platforms":["linkedin","instagram"],"media_urls":["https://example.com/spring.jpg"],"scheduled_at":"2027-03-01T09:00:00Z"}'`
- **Step 2: Processing:**  
  The webhook checks the secret, the Code node validates the platforms (X/Twitter is refused: CEKIVA retired it) and the media, then **CEKIVA: POST /v1/posts** schedules one post per network for 09:00 UTC on 1 March 2027.
- **Step 3: Delivered Result:**  
  Both posts are scheduled in CEKIVA and go live at the given time. The caller receives `{"status":"SUCCESS","post_ids":[…],"total_credits_charged":…}`; with no credits it receives `status: ERROR` and the `INSUFFICIENT_CREDITS` code instead.
