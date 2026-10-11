# 01 · AI Multi-Platform Content Factory

> Type a topic into a form: AI researches it, writes a separate post for Instagram, Facebook and LinkedIn, waits for your e-mail approval, creates an image, and CEKIVA publishes to all three with one request per network instead of three token-dependent native nodes.

---

## 1. Who is this workflow for?
- **Target Audience:** Solo creators and small marketing teams that want AI-written, human-approved posts on Instagram, Facebook and LinkedIn.
- **Experience Level:** Intermediate n8n user (several credentials: OpenAI, SerpAPI, Gmail, ImgBB).

## 2. What problem does it solve?
- **The Manual Pain:** Publishing the same idea to three networks means three native nodes or Graph API calls, Meta access tokens that expire without warning, a LinkedIn app to keep alive, an image host, and a stack of Merge nodes to join the branches. When one token lapses, one network silently stops.
- **The Automated Solution:** One `POST /v1/posts` per network to CEKIVA, which holds and refreshes the connections. The Merge and routing nodes disappear, a failing network never blocks the others, and every answer comes back in one uniform shape (`error.code`, `error.message`, `error.hint`).

## 3. What do you need? (Prerequisites)
- **Core Platform:** n8n instance (self-hosted or Cloud)
- **Distribution API:** CEKIVA account and API key (`Authorization: Bearer <API_KEY>`, created under *Settings → API keys*). Connect your social accounts once in CEKIVA.
- **External Services & Credentials:** OpenAI API key (the agents and the image); SerpAPI key (research tool); Gmail account (approval and result e-mails; OAuth2); imgbb.com API key (hosts the image; CEKIVA needs a public image URL here); Telegram bot (optional result summary).
- **Setup guide:** step-by-step credentials and options are in [SETUP.md](SETUP.md).
- **Technical Skills:** Basic understanding of importing n8n JSON and setting up credentials.

## 4. Free vs. Paid Components

| Component | Cost Model | Details |
| :--- | :--- | :--- |
| **Workflow File (.json)** | **100% Free** | Open-source under the MIT License. |
| **n8n Engine** | **Free / Variable** | Free if self-hosted (Community edition); paid if you use n8n Cloud. |
| **CEKIVA Account & API Access** | **€0 / Free** | €0 account fee, €0 API access fee: no monthly subscription, no per-account fee, no developer-tier lock. |
| **Social Publishing** | **Pay-as-you-go** | Credits are consumed only for what you publish (one dispatch per network). The price per post is shown in CEKIVA and set there; this README quotes no figure because it can change. |
| **AI Generation Engine** | **Variable** | OpenAI (GPT-4o / GPT-4o-mini and image generation) tokens, plus SerpAPI searches: billed by those providers at your plan's rates. The workflow also contains a disabled *Google Gemini LLM* node (set to `gemini-2.5-flash`) as an optional alternative model; it needs a Google Gemini API key only if you enable it. |
| **Third-Party Services** | **Free / Variable** | imgbb.com has a free tier (image hosting); Gmail and Telegram are free. No other image host is needed for CEKIVA itself. |

*X (Twitter) is not supported: CEKIVA retired that channel.*

## 5. What you'll get (End-to-End Delivery)
- **Input Data:** A topic typed into the form (and, optionally, a CEKIVA client workspace id).
- **Automated Processing:** Web research, one tailored post per network (Instagram, Facebook, LinkedIn), an approval e-mail, AI image generation, image hosting, then one CEKIVA request per network.
- **Final Output:** Three live posts (Instagram with the image, Facebook, LinkedIn) and a results e-mail, plus an optional Telegram summary.
- **Your Remaining Task:** Choose the topic and click *approve* in the e-mail. Nothing is published without it.

## 6. A Real-World Example
- **Step 1: Input:**  
  Form field *Topic*: `Spring offer: 20% off all houseplants until Sunday`
- **Step 2: Processing:**  
  The research agent searches the web, writes three platform-specific versions, mails you the draft; you approve. The image is generated and hosted, then three requests `POST https://cekiva.com/api/v1/posts` run (`platforms: ["instagram"]`, `["facebook_page"]`, `["linkedin"]`) with `publish_now: true`.
- **Step 3: Delivered Result:**  
  Instagram shows the generated image with a short caption and hashtags; the Facebook Page and LinkedIn each show their own wording of the offer. The results e-mail lists `Success` for each network, or the CEKIVA `error_code` (for example `INSUFFICIENT_CREDITS`) for any that failed. *(Illustrative wording; the AI writes new text on every run.)*
