# 04 · WordPress Blog to Social → CEKIVA

> Publish a WordPress post and an AI writes a version of it for Facebook, Instagram, LinkedIn, Threads and Bluesky; CEKIVA publishes each with the featured image and a link back.

---

## 1. Who is this workflow for?
- **Target Audience:** WordPress site owners, bloggers and content marketers who promote every article on social media.
- **Experience Level:** Beginner to intermediate (a webhook from WordPress and three credentials).

## 2. What problem does it solve?
- **The Manual Pain:** After every article someone copies the headline into five networks, rewrites it to fit each one, attaches the image and pastes the link. Automating it with native nodes brings Meta and LinkedIn tokens that expire and a Merge node per branch.
- **The Automated Solution:** A protected webhook receives the post id, the workflow reads the article, Gemini writes five versions, and **one `POST /v1/posts` per network** publishes them with the featured image. A failing network never stops the others.

## 3. What do you need? (Prerequisites)
- **Core Platform:** n8n instance (self-hosted or Cloud)
- **Distribution API:** CEKIVA account and API key (`Authorization: Bearer <API_KEY>`, created under *Settings → API keys*). Connect your social accounts once in CEKIVA.
- **External Services & Credentials:** WordPress site with an application password (to read the post); OpenRouter API key (the AI model); A way to call a webhook when a post is published (a webhook plugin, or a few lines of code on the `publish_post` hook).
- **Setup guide:** credentials, the webhook call and client routing are in [SETUP.md](SETUP.md).
- **Technical Skills:** Basic understanding of importing n8n JSON and setting up credentials.

## 4. Free vs. Paid Components

| Component | Cost Model | Details |
| :--- | :--- | :--- |
| **Workflow File (.json)** | **100% Free** | Open-source under the MIT License. |
| **n8n Engine** | **Free / Variable** | Free if self-hosted (Community edition); paid if you use n8n Cloud. |
| **CEKIVA Account & API Access** | **€0 / Free** | €0 account fee, €0 API access fee: no monthly subscription, no per-account fee, no developer-tier lock. |
| **Social Publishing** | **Pay-as-you-go** | Credits are consumed only for what you publish (one dispatch per network). The price per post is shown in CEKIVA and set there; this README quotes no figure because it can change. |
| **AI Generation Engine** | **Variable** | OpenRouter model tokens (`google/gemini-3.5-flash` by default), billed by OpenRouter at its rates. |
| **Third-Party Services** | **Free / Variable** | None. The featured image comes from your own WordPress site; no image host is needed. |

*X (Twitter) is not supported: CEKIVA retired that channel.*

## 5. What you'll get (End-to-End Delivery)
- **Input Data:** A WordPress post id sent to the webhook, for example `{"post_id": 4821}` (optionally a CEKIVA `client_id`).
- **Automated Processing:** Reads the post (title, content, link, featured image), AI writes the copy for five networks within their length limits, appends the link where it fits, and dispatches.
- **Final Output:** Up to five live posts with the featured image; If the post has no featured image, Instagram returns a clear `IMAGE_REQUIRED` row and the other networks still post.
- **Your Remaining Task:** Publish the article in WordPress. Optionally review the first runs' wording.

## 6. A Real-World Example
- **Step 1: Input:**  
  WordPress fires `POST /webhook/cekiva-wordpress` with `{"post_id": 4821}` for the article *Five spring pruning mistakes*.
- **Step 2: Processing:**  
  **Get Post** loads article 4821, **Featured image** fetches its image URL, the AI chain returns five platform versions, and **Build CEKIVA posts** creates five requests (`facebook_page`, `instagram`, `linkedin`, `threads`, `bluesky`) with `publish_now: true`.
- **Step 3: Delivered Result:**  
  Five posts go live, each with the article's image and, where the length limit allows, a link to the article; the Bluesky and Threads versions are short, the LinkedIn one longer. The result rows show `Success` per network. *(Illustrative wording.)*
