# 07 · Publish an Image or Video to Multiple Social Networks → CEKIVA

> A small form: tick the networks, write a caption, upload an image or a video; CEKIVA hosts the file and publishes it to every network you ticked, with no recurring plan.

---

## 1. Who is this workflow for?
- **Target Audience:** Freelancers and agencies who post client creative by hand, and anyone who wants an upload-and-publish form.
- **Experience Level:** Beginner friendly.

## 2. What problem does it solve?
- **The Manual Pain:** Posting one photo or clip to several networks means opening each app or keeping a per-seat publishing API and its profiles in sync, with a separate call for photos and for videos.
- **The Automated Solution:** A password-protected form that sends the file to CEKIVA (`POST /v1/upload/image` or `/v1/upload/video`) and makes **one `POST /v1/posts` per ticked network**. Several networks per submission, one uniform result page, no monthly plan.

## 3. What do you need? (Prerequisites)
- **Core Platform:** n8n instance (self-hosted or Cloud)
- **Distribution API:** CEKIVA account and API key (`Authorization: Bearer <API_KEY>`, created under *Settings → API keys*). Connect your social accounts once in CEKIVA.
- **External Services & Credentials:** A form password of your choice (Basic Auth on the form).
- **Setup guide:** the form fields, media limits and options are in [SETUP.md](SETUP.md).
- **Technical Skills:** Basic understanding of importing n8n JSON and setting up credentials.

## 4. Free vs. Paid Components

| Component | Cost Model | Details |
| :--- | :--- | :--- |
| **Workflow File (.json)** | **100% Free** | Open-source under the MIT License. |
| **n8n Engine** | **Free / Variable** | Free if self-hosted (Community edition); paid if you use n8n Cloud. |
| **CEKIVA Account & API Access** | **€0 / Free** | €0 account fee, €0 API access fee: no monthly subscription, no per-account fee, no developer-tier lock. |
| **Social Publishing** | **Pay-as-you-go** | Credits are consumed only for what you publish (one dispatch per network). The price per post is shown in CEKIVA and set there; this README quotes no figure because it can change. Uploads are free. |
| **AI Generation Engine** | **Variable** | None. This workflow uses no AI. |
| **Third-Party Services** | **Free / Variable** | None. CEKIVA hosts the uploaded file until the posts are published, then deletes it. |

*X (Twitter) is not supported: CEKIVA retired that channel.*

## 5. What you'll get (End-to-End Delivery)
- **Input Data:** A form with *Networks* (tick boxes), *Caption*, *Upload* (JPG, PNG, WebP, GIF, MP4, MOV or WebM) and an optional *Client workspace ID*.
- **Automated Processing:** Detects image or video from the file type, names it for CEKIVA, uploads it, skips networks that cannot take it (TikTok and YouTube need a video), and dispatches one request per network.
- **Final Output:** Live posts on every ticked network and a result page listing each outcome (✅, ❌ with the error code, ⏭️ skipped).
- **Your Remaining Task:** Pick the networks, write the caption and attach the file.

## 6. A Real-World Example
- **Step 1: Input:**  
  *Networks:* `Instagram, LinkedIn, TikTok (video only)` · *Caption:* `Behind the scenes at the studio` · *Upload:* `studio.mp4`
- **Step 2: Processing:**  
  The file is recognised as a video and uploaded with `POST /v1/upload/video`; three `POST /v1/posts` requests follow with the returned CEKIVA-hosted URL.
- **Step 3: Delivered Result:**  
  The clip appears on Instagram, LinkedIn and TikTok with the same caption. Had you uploaded a photo instead, TikTok would be reported as `⏭️ tiktok skipped: tiktok needs a video, you uploaded an image` and the other two would still post.
