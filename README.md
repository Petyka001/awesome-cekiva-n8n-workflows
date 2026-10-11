# awesome-cekiva-n8n-workflows

Open-source (MIT) [n8n](https://n8n.io) workflows that publish to **nine social networks with one HTTP request**, through the [CEKIVA](https://cekiva.com/?utm_source=github&utm_medium=template_repo&utm_campaign=n8n_starter) API. The workflow files are free; **CEKIVA itself is not free**: it is pay-as-you-go, you buy credits and spend them on every published post. There is no monthly subscription.

## What is CEKIVA?

[CEKIVA](https://cekiva.com/?utm_source=github&utm_medium=template_repo&utm_campaign=n8n_starter) is **publishing as a service**. You connect your social accounts once, then post from anything that can send an HTTP request: n8n, Make, a script, an AI agent.

- **Pay as you go.** You buy credits and spend them when a post is published; new accounts can post free during their first 7 days. There is no monthly fee and no lock-in. Current prices are on [cekiva.com](https://cekiva.com/?utm_source=github&utm_medium=template_repo&utm_campaign=n8n_starter).
- **No per-account surcharge.** Connect as many profiles as you like.
- **Nine networks, one endpoint.** LinkedIn, TikTok, YouTube, Pinterest, Bluesky, Discord, Facebook Pages, Instagram and Threads (plus Facebook Reels for video).
- **Client workspaces.** Separate accounts per client, roles and an internal approval flow, for agencies and freelancers.
- **A full REST API at no extra cost**, with English, machine-readable errors (stable `UPPERCASE` codes). See the [API documentation](https://cekiva.com/developers).

X (Twitter) is not supported: CEKIVA retired that channel.

## Workflows

| # | Workflow | What it does |
|---|----------|--------------|
| [01](workflows/01-ai-multiplatform-content-factory) | **AI multi-platform content factory** | A form asks for a topic; AI researches it and writes a separate post for Instagram, Facebook and LinkedIn; you approve by e-mail; an image is generated; CEKIVA publishes everywhere. |
| [02](workflows/02-cekiva-core-publisher) | **CEKIVA core publisher** | A protected webhook that validates a payload (`text`, `platforms`, `media_urls`, `workspace_id`, `scheduled_at`), publishes it through CEKIVA and answers `SUCCESS` or a clear `ERROR`. Plug it behind any trigger. |
| [03](workflows/03-ai-video-generation-omni-flash-cekiva) | **AI video generation (Gemini Omni 1.1 Flash)** | Every day AI invents a video idea and a prompt, CEKIVA generates the video with Gemini Omni 1.1 Flash and publishes it to TikTok, YouTube, Instagram and Facebook Reels. No video vendor, file host or per-network nodes. |
| [04](workflows/04-wordpress-blog-to-social-cekiva) | **WordPress blog to social** | A webhook receives a WordPress post id; AI writes a version for Facebook, Instagram, LinkedIn, Threads and Bluesky; CEKIVA publishes each with the featured image and a link. X (Twitter) is not included. |
| [05](workflows/05-clone-viral-tiktoks-ai-avatar-cekiva) | **Clone viral TikToks with AI avatars** | Send a TikTok link to your Telegram bot; AI analyses it, Perplexity suggests a fresh idea, an AI avatar speaks the new script, and CEKIVA uploads the video and publishes it to TikTok, Instagram, YouTube, Facebook, LinkedIn and Threads. X (Twitter) is not included. |
| [06](workflows/06-gemini-ai-social-content-generator-cekiva) | **Form + Gemini AI social generator** | A form (title, keywords, optional link and image); Gemini writes a separate post for Facebook, Instagram, LinkedIn, Threads and Bluesky; CEKIVA publishes each. Written from scratch, MIT, password-protected form. X (Twitter) is not included. |
| [07](workflows/07-publish-image-video-to-multiple-socials-cekiva) | **Publish an image or video to multiple networks** | A password-protected form: tick the networks, write a caption, upload an image or a video; CEKIVA hosts the file and publishes it to Facebook Page, Instagram, LinkedIn, Threads, Bluesky, TikTok and YouTube. No monthly plan. X (Twitter) is not included. |
| [08](workflows/08-calendar-to-social-multiplatform-cekiva) | **Content calendar → AI copy → CEKIVA** | Plan posts in a Google Sheet; every hour the READY rows get AI copy per network from Gemini and CEKIVA publishes or schedules them to Facebook Page, Instagram, LinkedIn, Threads and Bluesky, then writes the result back to the row. No Merge nodes, no expiring Graph API tokens. X (Twitter) is not included. |
| [09](workflows/09-content-publishing-factory-prompt-composition-cekiva) | **Content publishing factory with prompt composition** | Chat with a router agent; the system prompt and output schema live in Google Docs; AI writes the post and an image; you approve by e-mail; CEKIVA hosts the image and publishes to Instagram, Facebook Page, LinkedIn, Threads or Bluesky. One dispatch instead of Meta/X token trees and merge switches. X (Twitter) is not included. |

## Quick start (every workflow)

1. **Create a CEKIVA account** and connect your networks. Posting is free during your first 7 days; after that you pay credits per post, see the site for the current price.
2. **Create an API key**: in CEKIVA open *Settings → API keys*. The key starts with `tplk_` and is shown once.
3. **Add the credential in n8n**: *Credentials → Create → Header Auth*
   - **Name:** `CEKIVA API key`
   - **Header name:** `Authorization`
   - **Header value:** `Bearer tplk_...` (your key)
4. **Import** the workflow's `workflow.json` (*Workflows → Import from file*), select the credential in the CEKIVA node(s), and follow that workflow's `SETUP.md` (each workflow's `README.md` explains who it is for, what it costs and shows an example).

> Keep the key secret and limit the credential to `cekiva.com` (*Allowed domains*) if your n8n offers it.

## The API in 20 seconds

```bash
curl -X POST https://cekiva.com/api/v1/posts \
  -H "Authorization: Bearer tplk_YOUR_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "platforms": ["linkedin", "bluesky"],
    "content": "Hello from the CEKIVA API",
    "scheduled_for": "2027-01-15T09:00:00Z"
  }'
```

Add `"publish_now": true` to publish right away (`scheduled_for` is still required, any time works). Platform ids: `linkedin`, `tiktok`, `youtube`, `pinterest`, `bluesky`, `discord`, `facebook_page`, `instagram`, `threads`, `facebook_reels`. Instagram and Pinterest need an image URL (`image_url`), TikTok, YouTube and Reels need a video URL.

Errors come back as `{"detail": "...", "status_code": 402, "error": {"code": "INSUFFICIENT_CREDITS", "message": "...", "hint": "..."}}`: branch on `error.code`, not on the text.

## Good to know

- A repeated call creates a second post: the API has no idempotency key. Do not retry a request that may have succeeded.
- A post that is waiting for a client's approval cannot be deleted through the API.
- Posting costs credits; the workspace owner pays for their members.

## Contributing

Issues and pull requests are welcome: new workflows, fixes, better docs. Please never commit API keys, tokens or personal addresses; use credentials and placeholders.

## License

[MIT](LICENSE). Workflows 01, 03, 04, 05, 07 and 09 are derived from community templates; see their READMEs for the credit and what that means.
