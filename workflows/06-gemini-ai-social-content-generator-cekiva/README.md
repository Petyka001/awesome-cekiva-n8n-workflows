# 06 · Form + Gemini AI Social Generator → CEKIVA

> Fill in a short form and Gemini writes a separate post for Facebook, Instagram, LinkedIn, Threads and Bluesky; CEKIVA publishes each. Seven nodes, three credentials, MIT.

---

## 1. Who is this workflow for?
- **Target Audience:** Solo creators, small businesses and anyone who wants AI-written posts from a simple form.
- **Experience Level:** Beginner friendly.

## 2. What problem does it solve?
- **The Manual Pain:** Writing five versions of one announcement by hand takes time, and the usual no-code answer is a paid template plus a per-network node chain with expiring Meta and LinkedIn tokens and an image-host step.
- **The Automated Solution:** A password-protected form, one Gemini call that returns structured JSON, and **one `POST /v1/posts` per network**. No image host is needed (CEKIVA fetches your image link itself) and one failing network never stops the others.

## 3. What do you need? (Prerequisites)
- **Core Platform:** n8n instance (self-hosted or Cloud)
- **Distribution API:** CEKIVA account and API key (`Authorization: Bearer <API_KEY>`, created under *Settings → API keys*). Connect your social accounts once in CEKIVA.
- **External Services & Credentials:** Google Gemini API key (Google AI Studio); A form password of your choice (Basic Auth on the form).
- **Setup guide:** the form fields, options and credentials are in [SETUP.md](SETUP.md).
- **Technical Skills:** Basic understanding of importing n8n JSON and setting up credentials.

## 4. Free vs. Paid Components

| Component | Cost Model | Details |
| :--- | :--- | :--- |
| **Workflow File (.json)** | **100% Free** | Open-source under the MIT License. |
| **n8n Engine** | **Free / Variable** | Free if self-hosted (Community edition); paid if you use n8n Cloud. |
| **CEKIVA Account & API Access** | **No separate fee** | No monthly subscription, no per-account fee, no API surcharge, no developer-tier lock. This is not the same as free: publishing is paid, see the next row. |
| **Social Publishing (CEKIVA)** | **Paid: pay-as-you-go credits** | **CEKIVA is not free.** Every published post costs credits (one dispatch per network), billed per post and network; CEKIVA shows the price before you publish and it can change, so check cekiva.com. New accounts can post free during their first 7 days. |
| **AI Generation Engine** | **Variable** | Google Gemini tokens (`gemini-2.5-flash` by default; change it in the first Code node). A post uses a very small request; check Google's current terms for free-tier limits. |
| **Third-Party Services** | **Free / Variable** | None. No image host or other service is required. |

*No prices are quoted here because they can change: see cekiva.com. X (Twitter) is not supported: CEKIVA retired that channel.*

## 5. What you'll get (End-to-End Delivery)
- **Input Data:** A form with *Post Title*, *Keywords / Topic*, and optionally a *Link*, a public *Image URL* and a *Client workspace ID*.
- **Automated Processing:** Validates the form, asks Gemini for per-network copy within each network's limit, appends the link where it fits, and dispatches one request per network.
- **Final Output:** Up to five live posts and a result page listing every network (✅, ❌ with the CEKIVA error code, ⏭️ skipped).
- **Your Remaining Task:** Fill in the form and read the result page. Instagram needs an image URL, otherwise it is skipped.

## 6. A Real-World Example
- **Step 1: Input:**  
  *Post Title:* `Our spring collection is live` · *Keywords / Topic:* `spring, new arrivals, free shipping` · *Link:* `https://shop.example/spring` · *Image URL:* `https://shop.example/img/spring.jpg`
- **Step 2: Processing:**  
  **Prepare Gemini request** builds the prompt, **Gemini: write the copy** returns JSON for the five networks, **Build CEKIVA posts** shortens and links them, and five `POST /v1/posts` requests go out with `publish_now: true`.
- **Step 3: Delivered Result:**  
  Five posts go live: a warm two-sentence Facebook post, an Instagram caption with emoji and hashtags over the image, a professional LinkedIn paragraph, and short Threads and Bluesky posts (Bluesky stays under 300 characters). The result page shows `✅ facebook_page … ✅ bluesky`. *(Illustrative; Gemini writes new text on every run.)*
