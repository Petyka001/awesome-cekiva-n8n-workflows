# 05 · Clone viral TikToks with AI avatars → CEKIVA

Send a TikTok link to your Telegram bot. The workflow downloads and analyses the video, asks Perplexity for a fresh idea on the same topic, has an **AI avatar** speak the rewritten script, adds the overlay text, and then **CEKIVA publishes the finished video to TikTok, Instagram, YouTube, Facebook, LinkedIn and Threads**.

```
Telegram (TikTok link) → Only my chat → download + thumbnail + audio → GPT analysis → Perplexity idea → GPT rewrite
  → avatar video (Captions.ai) → overlay text (JSON2Video) → preview in Telegram
  → download final video → CEKIVA: upload video → CEKIVA: POST /v1/posts (one per network) → results in Telegram
```

> **No monthly subscription for the social distribution.** CEKIVA is pay-as-you-go: you pay credits per post, with no per-account fee and no plan. The prices are shown in CEKIVA. The other services in this workflow (OpenAI, Perplexity, Captions.ai, JSON2Video, RapidAPI, Cloudinary) are billed by those vendors.
>
> **X (Twitter) is excluded on purpose:** CEKIVA retired that channel, so the workflow has no X node and no `twitter` platform.

## What changed from the community template

Derived from the community template *Clone a viral TikTok and auto-post it to 9 platforms using Perplexity & Blotato* ([n8n.io/workflows/4110](https://n8n.io/workflows/4110); see the original page for its author and terms). Original: 41 nodes, this one: 39.

| Original | Now |
|---|---|
| Blotato: a media upload, an "Assign Social Media IDs" node with account ids, and 9 platform nodes (11 nodes) | **CEKIVA:** one video upload and one `POST /v1/posts` request per network (accounts are the ones you connected in CEKIVA) |
| Bluesky, Pinterest and X | gone: Pinterest needs an image, Bluesky was dropped from this template, X is retired |
| Anyone who found your bot could start it | **`Only my chat`**: only your Telegram chat id, and only a TikTok link, starts a run. Everything else is ignored |
| The final video was only sent to Telegram | the result of every network comes back to the same chat |
| Perplexity `sonar-reasoning` | `sonar-reasoning-pro` (the older model is being retired; this was not tested against a live key) |
| Google Sheet ids, links and a private image link inside the file | removed; you point the three Google Sheets nodes at your own copy |

Kept as they were: the Telegram trigger, the TikTok download (RapidAPI), Cloudinary, GPT-4o analysis and rewrite, Perplexity, Captions.ai avatars, JSON2Video overlay and the Google Sheets log.

## Set it up

1. **CEKIVA.** Connect your TikTok, Instagram, YouTube, Facebook Page, LinkedIn and Threads accounts, then create an API key (*Settings → API keys*, starts with `tplk_`).
2. **n8n credential** (*Credentials → Create → Header Auth*): name `CEKIVA API key`, header name `Authorization`, value `Bearer tplk_...`. Select it in **CEKIVA: upload video** and **CEKIVA: POST /v1/posts**.
3. **Telegram.** Create a bot with [@BotFather](https://t.me/BotFather), add it as the Telegram credential (trigger, *Send Video URL*, *Send Final Video Preview*, *Telegram: publishing results*). Find your chat id by messaging [@userinfobot](https://t.me/userinfobot), then put it into **Only my chat** in place of `REPLACE_WITH_YOUR_TELEGRAM_CHAT_ID`. **Until you do, the workflow ignores every message.**
4. **The services of the first three steps**, as in the original:
   - OpenAI (GPT-4o, audio transcription): the OpenAI credential on the `GPT` nodes.
   - RapidAPI: replace `YOUR_API_KEY` in **Download TikTok Video (RapidAPI)**.
   - Cloudinary: **Upload Thumbnail to Cloudinary**, your cloud name in the URL (`YOUR_ID`) and a Basic Auth credential, plus an unsigned upload preset called `n8n_clone`.
   - Perplexity: replace `YOUR_TOKEN_HERE` in **Suggest Similar Idea (Perplexity)**.
   - Captions.ai: replace `YOUR_API_KEY` in the three Captions nodes.
   - JSON2Video: a Custom Auth credential for **Add Overlay Text with JSON2Video** and **Fetch Final Video from JSON2Video**.
   - Google Sheets: a credential, and in the three Sheets nodes your own copy of the log sheet (`REPLACE_WITH_YOUR_SHEET_ID`; the columns are the French ones of the original template).
5. **Options**, at the top of **Build CEKIVA posts**: `PLATFORMS` (remove those you did not connect), `CLIENT_ID` (a CEKIVA client workspace; empty = your primary workspace), `PUBLISH_NOW` / `SCHEDULED_AT`, `AI_LABEL`.
6. Send your bot a TikTok link.

## Inputs and outputs

Input: a Telegram message containing only a TikTok URL (`tiktok.com`, `vm.tiktok.com`, ...), from your chat.

Output: the avatar video URL and a preview in Telegram, then a summary such as

```
CEKIVA publishing:
✅ tiktok
✅ instagram
❌ youtube: INSUFFICIENT_CREDITS …
```

The full rows (`platform`, `result`, `post_id`, `post_status`, `credits_charged`, `error_code`, `error_message`, `hint`) are the output of **Label CEKIVA results**. Branch on `error_code`, not on the text.

## Good to know

- **It publishes right away, with no approval step** (as the original did). Watch the first runs, or set `PUBLISH_NOW` to `false` and a `SCHEDULED_AT` time to get a window to delete a post in CEKIVA.
- **TikTok needs the video hosted by CEKIVA.** That is why the finished video is downloaded and uploaded with `POST /v1/upload/video` before posting; the same hosted copy is used for every network. CEKIVA deletes it once the posts have been published. The upload needs the workflow's **Name the file** step: TikTok reads the type from the `.mp4` name.
- **The "AI-generated" label is your decision.** An avatar video is AI-generated content; set `AI_LABEL` to `true` to have TikTok mark it as such. It is `false` by default, because CEKIVA never labels a video on its own.
- **Copying other people's ideas.** The workflow rewrites the topic and script rather than reposting the video, but you are responsible for what you publish; check each platform's rules on AI avatars and reused content.
- **Vendor APIs change.** In October 2026 the Captions.ai address used by the original (`api.captions.ai`) did not answer like a working API for us, and we could not test it with a key. If the avatar step fails, swap in the avatar service you use; everything from **Download final video** onwards only needs a public video URL in the sheet's `URL de la vidéo` column.
- A repeated run makes a new video and new posts: the API has no idempotency key.
