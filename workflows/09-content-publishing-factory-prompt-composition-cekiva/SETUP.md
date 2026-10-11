# 09 · Content publishing factory with prompt composition → CEKIVA

A chat-driven social media factory. You write *"a post about our spring launch for Instagram"*; a **router agent** picks the right network, a **content agent** writes the post from a system prompt and an output schema that you keep in two **Google Docs**, an image is generated, everything is archived in Google Drive and sent to you for **approval by e-mail**. When you approve, **CEKIVA publishes it**.

```
Chat → Router agent → (tool per network) → Compose prompt + schema from Google Docs → Content agent (+ web search) → image
   → Google Drive archive + Telegram + report e-mail → approval e-mail (Gmail send-and-wait)
   → approved? → download the image from Drive → CEKIVA: upload image → CEKIVA: POST /v1/posts → result in Telegram
```

**One dispatch instead of a token tree.** The community template this comes from published through a native node (or a hand-built Graph API call) per network, with Meta access tokens that expire, an image host (ImgBB), a Switch to route the network and Merge nodes to join the branches. Here everything after the approval is **two requests to CEKIVA**: it hosts the image, and publishes the post to the network the router chose. Connect your accounts once in CEKIVA; there is nothing to refresh. Pay-as-you-go credits, no monthly plan, prices shown in CEKIVA.

> **X (Twitter) is excluded on purpose:** CEKIVA retired that channel. The X tool, the X prompt/schema sections and the X publishing node are gone, and a stray `xtwitter` route is refused with a clear message. The YouTube Shorts tool is gone too (this workflow publishes image posts).
>
> **Licence.** The repository is [MIT](../../LICENSE); everything added here is MIT. The workflow is derived from the community template [n8n.io/workflows/3135](https://n8n.io/workflows/3135): see the original page for its author and terms.

## What changed from the community template

Original: 100 nodes, this one: 88 (47 working nodes and 41 notes).

| Original | Now |
|---|---|
| Tools for X, Instagram, Facebook, LinkedIn, Threads, YouTube Shorts | Tools for **Instagram, Facebook, LinkedIn, Threads and Bluesky** (X and YouTube Shorts removed, Bluesky added) |
| Publishing: a Switch, native X / Facebook / LinkedIn nodes, an Instagram Graph API container call, "implement here" placeholders for Threads and YouTube Shorts, four response nodes | **Removed.** One `Build CEKIVA post` Code node and one `POST /v1/posts` request |
| Image hosting on ImgBB (needs an API key) and a 3-way Merge | **Removed.** The image stays in Google Drive; after approval it is downloaded and uploaded to CEKIVA (`POST /v1/upload/image`) |
| Merge of the image and post contents before routing | **Removed** (no routing any more). The Merge that joins the prompt and the schema stays: it is part of the prompt composition |
| Approval, report and status e-mails sent to an address read from an n8n environment variable (`$env`, blocked by default on n8n 2.x) | Plain `REPLACE_WITH_YOUR_EMAIL` / `REPLACE_WITH_YOUR_TELEGRAM_CHAT_ID` placeholders; a forgotten one fails loudly instead of mailing a stranger |
| Prompt and schema examples naming another company and an X section | New generic examples in [`docs/`](docs) for the five networks |

Kept as they were: the chat trigger and memory, the router agent, the **prompt and schema composition from Google Docs** (the XML-tagged documents are parsed per network), the content agent with its web search tool, the image generation (pollinations.ai), the Google Drive archive, the Telegram status messages (optional), the HTML report and the **double-approval e-mail** that gates the publishing.

## Set it up

1. **CEKIVA.** Connect Instagram, Facebook Page, LinkedIn, Threads and Bluesky, then create an API key (*Settings → API keys*, starts with `tplk_`).
2. **Credentials in n8n:**

   | Credential | Used by |
   |---|---|
   | `CEKIVA API key` (Header Auth: header `Authorization`, value `Bearer tplk_...`) | **CEKIVA: upload image**, **CEKIVA: POST /v1/posts** |
   | OpenAI | the agents (router, content creator, e-mail composers) |
   | SerpAPI | the content agent's web search tool |
   | Google Docs, Google Drive, Gmail (OAuth2) | prompt and schema documents, the archive, the approval and report e-mails |
   | Telegram (optional) | the status messages |

3. **Two Google Docs.** Create one Doc with the contents of [`docs/system-prompt.txt`](docs/system-prompt.txt) (the system prompt and one section per network) and another with [`docs/schema.txt`](docs/schema.txt) (the JSON schema of each network's output). Put their ids (or URLs) into **Social Media System Prompt** and **Social Media Schema**. Edit the prompt to your brand; every network tag (`<instagram>`, `<facebook>`, `<linkedin>`, `<threads>`, `<bluesky>`) must exist in **both** documents, and the field names in the schema are the ones **Build CEKIVA post** reads (`caption`, `post`, `text_post`, `call_to_action`, and `hashtags` in `<common>`).
4. **Placeholders to replace:** `REPLACE_WITH_YOUR_EMAIL` (**Gmail User for Approval**, **Gmail**), `REPLACE_WITH_YOUR_TELEGRAM_CHAT_ID` (the Telegram nodes) and `REPLACE_WITH_YOUR_DRIVE_FOLDER_ID` (the two Google Drive save nodes).
5. **Options**, at the top of **Build CEKIVA post**: `CLIENT_ID` (a CEKIVA client workspace, a UUID from `GET https://cekiva.com/api/v1/clients`; empty = your own workspace) and `PUBLISH_NOW` / `SCHEDULED_AT`.
6. Open the chat of **When chat message received** and write, for example: *"Create an Instagram post about our spring collection launch, with a link to example.com"*.

## What happens, step by step

1. The **router agent** calls the tool of the network you asked for (one network per request).
2. The tool re-enters this workflow, which downloads your two Google Docs, picks the network's section and schema, and composes the final prompt.
3. The **content agent** researches the topic on the web and answers in the network's JSON schema.
4. An image is generated from the schema's `image_suggestion`; image, post and report are archived in Google Drive and mailed to you.
5. The **approval e-mail** asks you to approve or decline (it waits 45 minutes). Declining ends the run; nothing is uploaded or posted.
6. On approval the image is downloaded from Drive, uploaded to CEKIVA, and **one post** is created for the chosen network. The result (with CEKIVA's error code if something is wrong) arrives in Telegram.

## Good to know

- **Instagram needs the image**, which the flow always has. The image is uploaded to CEKIVA only after you approve, so a declined post leaves nothing behind.
- Posting costs credits from the CEKIVA wallet of the workspace owner; image upload is free. OpenAI, SerpAPI and pollinations are billed by those services.
- A repeated approval would post again: the API has no idempotency key.
- The approval step is what keeps an AI from publishing on its own. If you remove it, review the outputs of the first runs carefully.
- This template was checked with unit tests of its Code nodes and of the prompt composition against the documents in `docs/`; it was not run end to end on a live n8n with all the services connected, so test it with a draft before relying on it.
