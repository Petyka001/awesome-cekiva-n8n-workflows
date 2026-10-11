# 06 · Form + Gemini AI social generator → CEKIVA

Fill in a short form (title, keywords, an optional link and image). **Google Gemini writes a separate post for Facebook, Instagram, LinkedIn, Threads and Bluesky**, and **CEKIVA publishes each one**. Seven nodes, three credentials, nothing else to host.

```
Form → Prepare Gemini request → Gemini: write the copy → Build CEKIVA posts → one POST /v1/posts per network → Show the result (on the form's ending page)
```

**Free and open source.** This workflow is written from scratch and released under the repository's [MIT licence](../../LICENSE): use it, change it, sell your own version. It is not derived from anyone else's template.

- **No per-account surcharge.** Connect as many profiles as you like in CEKIVA; you pay credits per post, with no plan and no monthly fee. Prices are shown in CEKIVA.
- **No image host.** Give the form a public image link; CEKIVA fetches it itself. No ImgBB, Cloudinary or upload step.
- **No X (Twitter).** CEKIVA retired that channel, so there is no X node, no X copy and no `twitter` platform.

## Set it up

1. **CEKIVA.** Connect your Facebook Page, Instagram, LinkedIn, Threads and Bluesky accounts, then create an API key (*Settings → API keys*, starts with `tplk_`).
2. **Three credentials in n8n** (*Credentials → Create*):

   | Credential name | Type | Value |
   |---|---|---|
   | `CEKIVA API key` | Header Auth | header `Authorization`, value `Bearer tplk_...` |
   | `Gemini API key` | Header Auth | header `x-goog-api-key`, value = your key from [Google AI Studio](https://aistudio.google.com/apikey) |
   | `Form password` | Basic Auth | a user name and a password of your choice |

3. Import `workflow.json` and select the credentials: `Form password` in **Post details**, `Gemini API key` in **Gemini: write the copy**, `CEKIVA API key` in **CEKIVA: POST /v1/posts**.
4. **Activate** the workflow and open the production URL of **Post details** (the form). Your browser asks for the form password.

> The form is protected by a password on purpose: without it, anyone who finds the URL could publish on your accounts and spend your credits.

## The form

| Field | Required | Meaning |
|---|---|---|
| Post Title | yes | What the post is about, in a few words. |
| Keywords / Topic | yes | Keywords or a one-line brief for Gemini. |
| Link | no | A page to send readers to (`http(s)://...`). Added to the copy where it fits the network's length limit; Instagram captions do not make links clickable, so none is added there. |
| Image URL | no | A **public** image link. **Instagram needs it**: without it Instagram is skipped (and the result page says so); the other networks post as text. |
| Client workspace ID | no | The id of one of your CEKIVA client workspaces (a UUID, listed by `GET https://cekiva.com/api/v1/clients`). Empty = your own workspace. |

## Options

At the top of **Prepare Gemini request**:

- `MODEL`: `gemini-2.5-flash` by default. Any Gemini text model that supports `generateContent` works; Google retires old models from time to time, so change it here if it ever stops answering.
- `LANGUAGE` and `BRAND_VOICE`: the language of the posts and your voice in one line.

At the top of **Build CEKIVA posts**: `PUBLISH_NOW` / `SCHEDULED_AT` (ISO 8601 with a time zone) and `APPEND_LINK`.

## Output

The form's ending page lists what happened:

```
✅ facebook_page (scheduled)
✅ linkedin (scheduled)
❌ threads: INSUFFICIENT_CREDITS …
⏭️ instagram skipped: Instagram needs an image: fill in Image URL
```

The full rows (`platform`, `result`, `post_id`, `post_status`, `credits_charged`, `error_code`, `error_message`, `hint`) are the output of **Label CEKIVA results**. One network failing never stops the others: each network is its own request. Branch on `error_code`, not on the text.

## Good to know

- **It publishes right away** (`publish_now`). Set `PUBLISH_NOW` to `false` and a `SCHEDULED_AT` time if you want a window to review.
- Gemini writes from your title, keywords and link only. It is told not to invent facts, but read the result: you are responsible for what you publish.
- The Gemini call is billed by Google (it is a very small text request); posting is billed in CEKIVA credits.
- A repeated form submission posts again: the API has no idempotency key.
- The prompt is in **Prepare Gemini request**, where you can adapt the rules per network.
