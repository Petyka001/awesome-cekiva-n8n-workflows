# 08 · Content Calendar (Google Sheets) → AI Copy → CEKIVA

> Plan posts in a Google Sheet; every hour the READY rows get AI copy per network and CEKIVA publishes or schedules them, then the result is written back to the row.

---

## 1. Who is this workflow for?
- **Target Audience:** Agencies and content teams that plan posts in a spreadsheet and want them published (or scheduled) automatically.
- **Experience Level:** Intermediate n8n user (Google Sheets and a Gemini key).

## 2. What problem does it solve?
- **The Manual Pain:** A content calendar in a sheet still means someone copies each row into every network by hand. Automating it with native nodes brings a chain of Merge nodes and Graph API tokens that expire, and a row that fails half way is easy to publish twice.
- **The Automated Solution:** Rows marked `READY` are locked as `PROCESSING` before anything is sent, so a row can never go out twice; Gemini writes per-network copy; **one `POST /v1/posts` per network** publishes or schedules it (`Publish At`). The row ends as `PUBLISHED`, `QUEUED`, `PARTIAL` or `ERROR`, with the reason written back.

## 3. What do you need? (Prerequisites)
- **Core Platform:** n8n instance (self-hosted or Cloud)
- **Distribution API:** CEKIVA account and API key (`Authorization: Bearer <API_KEY>`, created under *Settings → API keys*). Connect your social accounts once in CEKIVA.
- **External Services & Credentials:** Google account (Sheets OAuth2) and a calendar sheet with the documented columns; Google Gemini API key (Google AI Studio).
- **Setup guide:** the sheet columns, statuses and options are in [SETUP.md](SETUP.md).
- **Technical Skills:** Basic understanding of importing n8n JSON and setting up credentials.

## 4. Free vs. Paid Components

| Component | Cost Model | Details |
| :--- | :--- | :--- |
| **Workflow File (.json)** | **100% Free** | Open-source under the MIT License. |
| **n8n Engine** | **Free / Variable** | Free if self-hosted (Community edition); paid if you use n8n Cloud. |
| **CEKIVA Account & API Access** | **No separate fee** | No monthly subscription, no per-account fee, no API surcharge, no developer-tier lock. This is not the same as free: publishing is paid, see the next row. |
| **Social Publishing (CEKIVA)** | **Paid: pay-as-you-go credits** | **CEKIVA is not free.** Every published post costs credits (one dispatch per network and row), billed per post and network; CEKIVA shows the price before you publish and it can change, so check cekiva.com. New accounts can post free during their first 7 days. |
| **AI Generation Engine** | **Variable** | Google Gemini tokens (`gemini-2.5-flash` by default), at most three rows per run by default. Check Google's current terms for free-tier limits. |
| **Third-Party Services** | **Free / Variable** | Google Sheets is free with a Google account. No image host is needed (the `Image URL` column goes straight to CEKIVA). |

*No prices are quoted here because they can change: see cekiva.com. X (Twitter) is not supported: CEKIVA retired that channel.*

## 5. What you'll get (End-to-End Delivery)
- **Input Data:** A calendar row with `Status = READY`, a `Topic`, and optionally `Brief`, `Link`, `Image URL`, `Publish At`, `Client ID` and `Platforms`.
- **Automated Processing:** Validates each row, locks it, asks Gemini for per-network copy, builds the requests, dispatches them, and writes the outcome back.
- **Final Output:** Live or scheduled posts on the chosen networks and an updated row (`Status`, `Result`, `Post IDs`, `Updated`).
- **Your Remaining Task:** Write the rows and set `READY`; fix any `ERROR` or `PARTIAL` rows by hand.

## 6. A Real-World Example
- **Step 1: Input:**  
  Sheet row: `Status=READY · Topic=Black Friday preview · Brief=early access for subscribers · Link=https://shop.example/bf · Image URL=https://shop.example/bf.jpg · Publish At=2026-11-27T08:00:00Z · Platforms=facebook,instagram,linkedin`
- **Step 2: Processing:**  
  The hourly run takes the row, sets it to `PROCESSING`, Gemini writes three versions, and three requests are sent with `scheduled_for: 2026-11-27T08:00:00Z` and `publish_now: false`.
- **Step 3: Delivered Result:**  
  CEKIVA queues three posts for 08:00 UTC on 27 November; the row becomes `QUEUED` with `Result = OK: facebook_page, instagram, linkedin` and the three post ids. If LinkedIn had been refused, the row would read `PARTIAL` and list the failure.
