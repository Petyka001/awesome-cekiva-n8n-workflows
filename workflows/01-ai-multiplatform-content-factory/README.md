# 01 · AI multi-platform content factory

Type a topic into a form; AI researches it, writes **a separate post for Instagram, Facebook and LinkedIn**, asks you to approve by e-mail, creates an image, and publishes everywhere through **one [CEKIVA](https://cekiva.com/?utm_source=github&utm_medium=template_repo&utm_campaign=n8n_starter) API call per network**. A summary arrives by e-mail and (optionally) Telegram.

```
Form → AI research + copy (GPT-4o, SerpAPI) → approval e-mail → image (OpenAI) → ImgBB → Build posts → CEKIVA POST /v1/posts → results → e-mail / Telegram
```

## What you need

| What | Used for | n8n credential |
|---|---|---|
| **CEKIVA API key** | publishing | *Header Auth*, name `CEKIVA API key` |
| OpenAI API key | the AI agents and the image | OpenAI |
| SerpAPI key | the research tool | SerpApi |
| Gmail account | approval and results e-mails | Gmail OAuth2 |
| imgbb.com API key | hosting the image (CEKIVA needs a public image URL) | *Query Auth*, name `ImgBB API key` |
| Telegram bot (optional) | the results summary in a chat; if you do not use it, delete the *Prepare Results Message* and *Telegram Results* nodes | Telegram |

## Set it up

1. **CEKIVA credential.** In CEKIVA open *Settings → API keys* and create a key (it starts with `tplk_` and is shown once). In n8n: *Credentials → Create → Header Auth*:
   - **Name:** `CEKIVA API key`
   - **Header name:** `Authorization`
   - **Header value:** `Bearer tplk_...`

   Select it in the node **CEKIVA: POST /v1/posts**. In CEKIVA, connect your Instagram, Facebook Page and LinkedIn once (there is no per-account fee).
2. **ImgBB credential.** *Credentials → Create → Query Auth*: **name** `key`, **value** your imgbb.com API key. Select it in **Save Image to imgbb.com3**.
3. **The other credentials:** OpenAI, SerpAPI, Gmail OAuth2 (and Telegram if you want the chat summary) in their nodes.
4. **Where the messages go.** Type your own e-mail address into the three Gmail nodes (*Gmail User for Approval*, *Gmail Results*, and the disabled *Approve Final Post Content*): the field says `REPLACE_WITH_YOUR_EMAIL`. Put your chat id into *Telegram Results* (`REPLACE_WITH_YOUR_TELEGRAM_CHAT_ID`). Until you do, those nodes stop with a clear error and nothing is sent to anyone else.
5. **Options** at the top of the **Build CEKIVA posts** node:

   | Option | Meaning |
   |---|---|
   | `CLIENT_ID` | One of your CEKIVA client workspaces (`GET https://cekiva.com/api/v1/clients`). Empty = your primary workspace. The form can also pass a `client_id`. |
   | `PUBLISH_NOW` | `true` (default) publishes right after approval; `false` uses `SCHEDULED_AT`. |
   | `SCHEDULED_AT` | ISO 8601 with a time zone, e.g. `2027-03-01T09:00:00Z`. |

6. Open the **Submit Social Post Details** form and use it.

## The form

| Field | Required | |
|---|---|---|
| Topic | yes | The main subject of the post. |
| Keywords or Hashtags | no | Words the AI should work in. |
| Link | no | A link to include. |

## What happens at publishing

Each network gets the text the AI wrote for it, in its own request: **Instagram** the caption, **Facebook** the post plus its call to action, **LinkedIn** the post, call to action and hashtags. The generated image is uploaded to ImgBB and its link is sent along. Each request answers separately, so a problem on one network never blocks the others. The results e-mail lists a `Success` or `Error` row per platform, with CEKIVA's error code and message.

## Good to know

- **Instagram needs an image.** If the image step fails, the Instagram row reports `IMAGE_REQUIRED`; Facebook and LinkedIn still go out as text posts.
- **X (Twitter) is not included**: CEKIVA retired that channel, so the original's X post was removed. (The AI may still write an X text; it is simply not sent.)
- Posting costs CEKIVA credits when published, and the AI/image steps use your own OpenAI and SerpAPI quota. A repeated call creates a second post.
- Do not publish your own credentials with the exported workflow. This file contains none, only credential names.

## Credit and license

This workflow is **derived from a community template**: [Automated AI Powered Social Media Content Factory](https://n8n.io/workflows/3066) on n8n.io (the form is branded *workflows.diy*). The AI research and copywriting logic, the approval flow, the image step and the results summaries are the original author's work; only the publishing part was replaced by a CEKIVA API call, together with the setup notes and the changes described above. **The MIT license of this repository covers the changes made here. The original parts remain subject to the original author's terms**; please credit the original template when you share it, and see its page on n8n.io. If you are the original author and want this changed or removed, please open an issue.
