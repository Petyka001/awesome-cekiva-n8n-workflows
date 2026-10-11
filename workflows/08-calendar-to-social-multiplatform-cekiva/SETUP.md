# 08 · Content calendar (Google Sheets) → AI copy → CEKIVA

Keep your content plan in a Google Sheet. Every hour the workflow takes the rows marked **READY**, has **Gemini write a separate post for each network**, and **CEKIVA publishes or schedules them** on Facebook Page, Instagram, LinkedIn, Threads and Bluesky. The sheet row is updated with what happened (`PUBLISHED`, `QUEUED`, `PARTIAL` or `ERROR`).

```
Every hour / Run now → Get READY rows → Validate rows → Lock row (PROCESSING) → Gemini: write the copy → Build CEKIVA posts
  → one POST /v1/posts per network → Collect results → Write result (back into the sheet)
```

- **No Merge nodes, no Graph API tokens.** The usual way to post to several networks in n8n is a native node per network, an image step per network and a chain of Merge nodes to bring the results together (five of them in the community template this idea comes from). Facebook and Instagram nodes also depend on Meta access tokens that expire. Here there is one HTTP request per network to CEKIVA, which holds the connections and refreshes them for you.
- **Scheduling is built in.** Put a future time in `Publish At` and CEKIVA schedules the post for that moment; leave it empty (or in the past) and it goes out now.
- **Pay-as-you-go.** CEKIVA charges credits per post, with no monthly plan and no per-account fee; prices are shown in CEKIVA. The Gemini calls are billed by Google (small text requests).
- **X (Twitter) is excluded on purpose.** CEKIVA retired that channel, so there is no X node, no X copy and no `twitter` platform. A row that asks for it is marked `ERROR` with the reason.

## About this template

Written from scratch and released under the repository's [MIT licence](../../LICENSE). The idea and the overall shape (a scheduled run that turns calendar rows into per-network posts) follow the community template [n8n.io/workflows/4637](https://n8n.io/workflows/4637). The copy of that template we worked from carried only its node names and connections, without its prompts or settings, so every prompt, setting and line of code here is new, and it uses Google Sheets for the calendar (the community template used Notion for its brand brief).

## Set it up

1. **CEKIVA.** Connect your networks and create an API key (*Settings → API keys*, starts with `tplk_`).
2. **Three credentials in n8n** (*Credentials → Create*):

   | Credential name | Type | Value |
   |---|---|---|
   | `Google Sheets` | Google Sheets OAuth2 | your Google account |
   | `Gemini API key` | Header Auth | header `x-goog-api-key`, value = your key from [Google AI Studio](https://aistudio.google.com/apikey) |
   | `CEKIVA API key` | Header Auth | header `Authorization`, value `Bearer tplk_...` |

3. **The calendar.** Create a Google Sheet; the **first tab**, first row, holds exactly these headers:

   | Column | You fill it? | Meaning |
   |---|---|---|
   | `Status` | yes | Set to `READY` when the row should go out. The workflow only touches `READY` rows. |
   | `Topic` | yes | What the post is about (required). |
   | `Brief` | no | Extra detail for the AI: key points, offer, tone. |
   | `Link` | no | A page to send readers to. Added to the copy where it fits (not on Instagram). |
   | `Image URL` | no | A **public** image link. **Instagram needs it**: without one, Instagram is skipped for that row. |
   | `Publish At` | no | ISO 8601 **with a time zone**, e.g. `2026-12-01T09:00:00Z`. Future = scheduled (`QUEUED`); empty or past = publish now. |
   | `Client ID` | no | A CEKIVA client workspace (UUID from `GET https://cekiva.com/api/v1/clients`). Empty = your own workspace. |
   | `Platforms` | no | Comma list of `facebook_page`, `instagram`, `linkedin`, `threads`, `bluesky` (`facebook` and `ig` work too). Empty = all five. |
   | `Result`, `Post IDs`, `Updated` | filled by the workflow | What happened, the CEKIVA post ids, and when. |

4. Import `workflow.json`. In **Get READY rows**, **Lock row** and **Write result** select the `Google Sheets` credential and your document (replace `REPLACE_WITH_YOUR_SHEET_ID`; it is the long id in the sheet's URL). Select `Gemini API key` in **Gemini: write the copy** and `CEKIVA API key` in **CEKIVA: POST /v1/posts**.
5. Add a row with `Status = READY`, run **Run now (test)** once, and check the sheet. Then **activate** the workflow.

## What the statuses mean

| Status | Meaning |
|---|---|
| `READY` | You set it. Waiting for the next run. |
| `PROCESSING` | The workflow took the row. It is set **before anything is sent**, so a row can never be sent twice, even if the run stops half way. A row stuck here was interrupted: check CEKIVA, then set it back to `READY` if nothing went out. |
| `PUBLISHED` | CEKIVA accepted every network and is publishing now. |
| `QUEUED` | CEKIVA accepted every network for the `Publish At` time. |
| `PARTIAL` | Some networks were accepted, some were not (see `Result`). The accepted ones are **not** repeated; fix the failed ones by hand or in a new row. |
| `ERROR` | Nothing was sent: an invalid row (the reason is in `Result`), a Gemini problem, or every network was refused (for example no credits). |

`PUBLISHED` means CEKIVA took the post; a network can still refuse it a moment later. CEKIVA shows the final state of each post.

## Options

At the top of **Validate rows**: `MAX_ROWS_PER_RUN` (default 3, a cap on the credits and AI calls one run can spend; further READY rows wait for the next run), `MODEL` (`gemini-2.5-flash`; change it if Google retires it), `LANGUAGE` and `BRAND_VOICE`. The prompt itself is in the same node. The run interval is in **Every hour**.

## Good to know

- One network failing never stops the others: each network is its own request.
- Posting costs credits from the CEKIVA wallet of the workspace owner; if the wallet runs out, the row becomes `ERROR` or `PARTIAL` with the `INSUFFICIENT_CREDITS` code.
- The AI is told not to invent facts, but read what it writes: you are responsible for what you publish. Use `Publish At` in the future to get time to review (and delete a queued post in CEKIVA).
- A row you set back to `READY` after it was published is published again: the API has no idempotency key.
