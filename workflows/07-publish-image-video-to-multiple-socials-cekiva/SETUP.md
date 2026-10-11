# 07 · Publish an image or a video to multiple social networks → CEKIVA

A small form: pick the networks, write a caption, upload an **image or a video**. The workflow sends the file to CEKIVA and **CEKIVA publishes it to every network you ticked**: Facebook Page, Instagram, LinkedIn, Threads, Bluesky, TikTok and YouTube.

```
Form → Prepare media → image or video? → CEKIVA: upload image | upload video → Build CEKIVA posts → one POST /v1/posts per network → result page
```

**A drop-in replacement for a per-seat publishing API, without the recurring subscription.** This is the "upload a file, tick the networks" form that Upload-Post style workflows offer, rebuilt on CEKIVA: **pay-as-you-go credits only, no monthly plan, no per-profile fee**. Connect your accounts once in CEKIVA; there is no "profile name" to type into the form and no second account to keep in sync. Prices are shown in CEKIVA.

> **X (Twitter) is excluded on purpose.** CEKIVA retired that channel, so the form has no X option and the workflow no X node or `twitter` platform.

## What changed from the community template

Derived from the community template *Publish image/video to multiple social media* ([n8n.io/workflows/3669](https://n8n.io/workflows/3669); see the original page for its author and terms). Original: 14 nodes, this one: 10.

| Original | Now |
|---|---|
| Upload-Post API: one call for photos, one for videos, an `Apikey` header, a "profile" per user | CEKIVA: `POST /v1/upload/image` or `/v1/upload/video`, then one `POST /v1/posts` per network; one API key |
| One network per submission (a dropdown), including X | **Several networks at once** (checkboxes), X removed, Facebook Page, Bluesky and YouTube added |
| An "Account" field with a profile name, plus a Facebook page id | an optional **Client workspace ID** (CEKIVA client workspaces); Pages are connected once in CEKIVA |
| Only `image/jpeg` and `video/mp4` were routed; anything else got no answer | JPG, PNG, WebP, GIF, MP4, MOV, WebM; anything else is refused with a clear message |
| Four "OK/KO" pages and two result/IF nodes | one result page listing every network (✅, ❌ with CEKIVA's error code, ⏭️ skipped) |
| Open form | **Basic Auth on the form**, so a leaked URL cannot spend your credits |
| An advertising image and a referral link in the notes | removed |

## Set it up

1. **CEKIVA.** Connect the networks you use, then create an API key (*Settings → API keys*, starts with `tplk_`).
2. **Two credentials in n8n** (*Credentials → Create*):

   | Credential name | Type | Value |
   |---|---|---|
   | `CEKIVA API key` | Header Auth | header `Authorization`, value `Bearer tplk_...` |
   | `Form password` | Basic Auth | a user name and a password of your choice |

3. Import `workflow.json`. Select `Form password` in **Post details** and `CEKIVA API key` in the three CEKIVA nodes (**CEKIVA: upload image**, **CEKIVA: upload video**, **CEKIVA: POST /v1/posts**).
4. **Activate** the workflow and open the production URL of **Post details**. Your browser asks for the form password.

> The form is protected on purpose: an open form would let anyone who finds the URL publish on your accounts and spend your credits.

## The form

| Field | Required | Meaning |
|---|---|---|
| Networks | yes | Tick one or more: Facebook Page, Instagram, LinkedIn, Threads, Bluesky, TikTok (video only), YouTube (video only). |
| Caption | yes | The text of the post, sent as it is to every network you ticked. |
| Upload | yes | One image (JPG, PNG, WebP, GIF) or one video (MP4, MOV, WebM). |
| Client workspace ID | no | The id of one of your CEKIVA client workspaces (a UUID, listed by `GET https://cekiva.com/api/v1/clients`). Empty = your own workspace. |

## Images and videos

The workflow looks at the file's type and sends it to CEKIVA first, because CEKIVA hosts the file until the posts are published (then deletes it). That hosted link is what every network receives, which is also what TikTok requires: it only fetches videos that CEKIVA hosts.

- **Image** (up to 8 MB): published to Facebook Page, Instagram, LinkedIn, Threads, Bluesky. **TikTok and YouTube are skipped** for an image (they need a video) and the result page says so.
- **Video** (up to 300 MB): can go to every listed network. TikTok and YouTube need one.
- **Instagram always needs media**, which the form guarantees: the upload is required.
- n8n limits the size of an upload through the form too (`N8N_PAYLOAD_SIZE_MAX`, 16 MB by default). For larger videos raise that setting on your n8n.

## Output

The result page lists what happened, for example:

```
✅ facebook_page (scheduled)
✅ instagram (scheduled)
❌ bluesky: INSUFFICIENT_CREDITS …
⏭️ youtube skipped: youtube needs a video, you uploaded an image
```

The full rows (`platform`, `result`, `post_id`, `post_status`, `credits_charged`, `error_code`, `error_message`, `hint`) are the output of **Label CEKIVA results**. One network failing never stops the others, because each network is its own request. Branch on `error_code`, not on the text. If the upload itself fails (file too large or of the wrong type), the form shows an error and the details are in *Executions*; uploads cost no credits.

## Options

At the top of **Build CEKIVA posts**: `PUBLISH_NOW` (default `true`) and `SCHEDULED_AT` (ISO 8601 with a time zone) to publish later instead.

## Good to know

- Posting costs credits from the CEKIVA wallet of the workspace owner; uploads are free.
- A repeated form submission posts again: the API has no idempotency key.
- The same caption goes to every network. For per-network copy written by AI, see workflow 06.
