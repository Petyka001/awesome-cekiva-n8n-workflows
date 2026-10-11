# 05 · Clone Viral TikToks with AI Avatars → CEKIVA

> Send a TikTok link to your Telegram bot: the workflow analyses it, Perplexity suggests a fresh idea, an AI avatar speaks the new script, and CEKIVA uploads and publishes the video to TikTok, Instagram, YouTube, Facebook, LinkedIn and Threads.

---

## 1. Who is this workflow for?
- **Target Audience:** TikTok-first creators and social media managers who build avatar or faceless video channels.
- **Experience Level:** Advanced n8n user (many third-party services and a long pipeline).

## 2. What problem does it solve?
- **The Manual Pain:** This pipeline needs a video-download service, an image host, GPT analysis, Perplexity, an avatar service, a text-overlay renderer, a Google Sheet and a publishing service with its own account ids, plus one node per network. Every extra key is another thing that expires.
- **The Automated Solution:** The publishing half collapses into CEKIVA: **`POST /v1/upload/video`** hosts the finished video (TikTok only fetches CEKIVA-hosted files) and **one `POST /v1/posts` per network** publishes it. The Blotato account ids and the nine per-network nodes are gone, and only your own Telegram chat can start a run.

## 3. What do you need? (Prerequisites)
- **Core Platform:** n8n instance (self-hosted or Cloud)
- **Distribution API:** CEKIVA account and API key (`Authorization: Bearer <API_KEY>`, created under *Settings → API keys*). Connect your social accounts once in CEKIVA.
- **External Services & Credentials:** Telegram bot (trigger and previews); OpenAI API key (GPT-4o analysis and rewrite, transcription); Perplexity API key; RapidAPI key for a TikTok download API; Cloudinary account (thumbnail hosting); Captions.ai API key (the avatar step); JSON2Video API key (overlay text); Google Sheets (the log).
- **Known risk:** the Captions.ai address used by the original template (`api.captions.ai`) did not answer like a working API when we checked. If the avatar step fails, replace it with the avatar service you use: everything from **Download final video** onwards only needs a public video URL.
- **Setup guide:** the `YOUR_...` placeholders and options are in [SETUP.md](SETUP.md).
- **Technical Skills:** Basic understanding of importing n8n JSON and setting up credentials.

## 4. Free vs. Paid Components

| Component | Cost Model | Details |
| :--- | :--- | :--- |
| **Workflow File (.json)** | **100% Free** | Open-source under the MIT License. |
| **n8n Engine** | **Free / Variable** | Free if self-hosted (Community edition); paid if you use n8n Cloud. |
| **CEKIVA Account & API Access** | **€0 / Free** | €0 account fee, €0 API access fee: no monthly subscription, no per-account fee, no developer-tier lock. |
| **Social Publishing** | **Pay-as-you-go** | Credits are consumed only for what you publish (one dispatch per network; video upload is free). The price per post is shown in CEKIVA and set there; this README quotes no figure because it can change. |
| **AI Generation Engine** | **Variable** | OpenAI and Perplexity tokens, plus Captions.ai and JSON2Video renders: all billed by those vendors. RapidAPI and Cloudinary have their own plans. |
| **Third-Party Services** | **Free / Variable** | Several paid or metered services as listed above. CEKIVA replaces only the publishing and the video hosting. |

*X (Twitter) is not supported: CEKIVA retired that channel.*

## 5. What you'll get (End-to-End Delivery)
- **Input Data:** A Telegram message containing a TikTok URL, sent from your own chat.
- **Automated Processing:** Download and analyse the video, idea by Perplexity, script rewrite, AI avatar video, overlay text, then upload to CEKIVA and publish.
- **Final Output:** The new video live on up to six networks, a preview in Telegram and a summary of every network's result.
- **Your Remaining Task:** Send the link, look at the preview, and decide on the *AI-generated* label (off by default). You are responsible for what you publish and for the platforms' rules on AI avatars and reused content.

## 6. A Real-World Example
- **Step 1: Input:**  
  Telegram message from your chat: `https://www.tiktok.com/@somecreator/video/7300000000000000000`
- **Step 2: Processing:**  
  The workflow rewrites the idea, renders the avatar video, then **Download final video → CEKIVA: upload video** returns a hosted `.mp4` URL and six requests `POST /v1/posts` run (`tiktok`, `instagram`, `youtube`, `facebook_page`, `linkedin`, `threads`).
- **Step 3: Delivered Result:**  
  Your Telegram chat receives `✅ tiktok ✅ instagram ✅ youtube ❌ linkedin: INSUFFICIENT_CREDITS …` style lines; the successful networks show the avatar video with the AI-written caption. *(Illustrative.)*
