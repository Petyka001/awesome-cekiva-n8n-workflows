# 04 · WordPress blog to social, through CEKIVA

Publish a WordPress post, and an AI writes a separate version of it for **Facebook, Instagram, LinkedIn, Threads and Bluesky**; CEKIVA publishes each one with the post's featured image and a link back to the article.

```
WordPress webhook (or manual test) → Post ID → Get Post → Featured image → AI writes 5 versions → one POST /v1/posts per network → results
```

> **X (Twitter) is excluded on purpose.** CEKIVA retired that channel, so the workflow has no X node, no X copy and no `twitter` platform.

## What changed from the community template

Derived from the community template *AI Social Media Publisher from WordPress* ([n8n.io/workflows/3086](https://n8n.io/workflows/3086); see the original page for its author and terms). Original: 20 nodes, this one: 16.

| Original | Now |
|---|---|
| Started by hand; the post id came from a Google Sheet | **A protected webhook** receives the WordPress post id (a manual test trigger is kept). No Google Sheets, no sheet to clone |
| Native X, Facebook, LinkedIn nodes and a two-step Instagram container call, each with its own app and credential | one `POST /v1/posts` per network through CEKIVA; you connect the accounts once in CEKIVA |
| AI-generated images (OpenAI, billed to you, temporary URLs) | the **post's featured image** (free, public URL); no OpenAI image cost |
| Copy for LinkedIn, Instagram, Facebook and X | copy for Facebook, Instagram, LinkedIn, **Threads and Bluesky**, with their length limits in the prompt |
| Free experimental Gemini 2.0 model on OpenRouter (no longer in OpenRouter's catalogue) | `google/gemini-3.5-flash` (change it in *OpenRouter Chat Model* if you prefer another) |
| A prompt that named another company's website and products | a generic prompt |

## Set it up

1. **CEKIVA:** connect your Facebook Page, Instagram, LinkedIn, Threads and Bluesky, and create an API key (*Settings → API keys*, starts with `tplk_`).
2. **n8n credentials** (*Credentials → Create → Header Auth*), two of them:

   | Credential name | Header name | Header value |
   |---|---|---|
   | `CEKIVA API key` | `Authorization` | `Bearer tplk_...` |
   | `CEKIVA webhook secret` | `X-Webhook-Secret` | a long random string |

   Plus your **WordPress** credential (an application password, *Users → Profile → Application Passwords*) and your **OpenRouter** credential.
3. Import `workflow.json` and select the credentials: `CEKIVA API key` in **CEKIVA: POST /v1/posts**, `CEKIVA webhook secret` in **WordPress webhook**, WordPress in **Get Post**, OpenRouter in **OpenRouter Chat Model**.
4. **Activate** the workflow and copy the production URL of **WordPress webhook** (path `cekiva-wordpress`).

> The webhook is protected by the secret on purpose: an open webhook would let anyone spend your credits.

## WordPress webhook intake

When a post is published, WordPress must send a `POST` with the secret header and the post id:

```bash
curl -X POST https://YOUR-N8N/webhook/cekiva-wordpress \
  -H "X-Webhook-Secret: YOUR_SECRET" \
  -H "Content-Type: application/json" \
  -d '{"post_id": 123}'
```

WordPress does not do this by itself. Use a webhook plugin that can set a custom header and a JSON body (for example *WP Webhooks* or *Uncanny Automator*), triggered on "post published", or a few lines in a small plugin that call `wp_remote_post()` on the `publish_post` hook. The body only needs the id; `post_id`, `ID` and `post.ID` are all understood. The webhook answers at once, so WordPress is never kept waiting for the AI.

**Manual test:** open the workflow, type a real post id into **Test Post ID** (`"post_id": "123"`) and run **When clicking 'Test workflow'**.

## Client workspace routing

By default the posts go to your primary CEKIVA workspace. To publish into a client's workspace:

- send `"client_id": "<id>"` in the webhook body (one WordPress site per client, say), or
- set `CLIENT_ID` at the top of **Build CEKIVA posts** for every run.

The ids are listed by `GET https://cekiva.com/api/v1/clients` (each client workspace is a client). The webhook value wins over `CLIENT_ID`. An id that is not yours comes back as an error row, nothing is posted elsewhere.

## Options

At the top of **Build CEKIVA posts**: `PUBLISH_NOW` / `SCHEDULED_AT` (ISO 8601 with a time zone) and `APPEND_LINK`. The article's link is added to the copy for Facebook, LinkedIn, Threads and Bluesky when it still fits the length limit (Threads 500, Bluesky 300); Instagram captions do not make links clickable, so none is added there.

## Output

One row per network from **Label CEKIVA results**:

```json
{ "platform": "linkedin", "result": "Success", "http_status": 201, "post_id": "…", "post_status": "scheduled", "credits_charged": 0, "error_code": null, "error_message": null, "hint": null }
```

or `"result": "Error"` with CEKIVA's `error_code`, `error_message` and `hint`. Branch on the code, not on the text.

## Good to know

- **Instagram needs an image.** A post without a featured image gets an `IMAGE_REQUIRED` row for Instagram; the other networks still post.
- **Credits.** Every post costs credits from the CEKIVA wallet of the workspace owner; the price is shown in CEKIVA and set there, not in this workflow. AI text through OpenRouter is billed by OpenRouter.
- A repeated webhook call publishes the post again: the API has no idempotency key, so make your WordPress trigger fire once per post.
- Only the featured image is used. Images inside the article body are not picked up.
