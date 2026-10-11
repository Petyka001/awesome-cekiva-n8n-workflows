# 02 · CEKIVA core publisher

A small, production-minded n8n workflow: **webhook in, posts out.** It validates the payload, publishes it through the [CEKIVA](https://cekiva.com/?utm_source=github&utm_medium=template_repo&utm_campaign=n8n_starter) API and answers with a uniform result. Put any trigger in front of it (a form, a schedule, a Google Sheet, another workflow, an AI agent).

```
Webhook → Config → Validate & Prepare → (workspace lookup) → POST /v1/posts → Format Result → Respond
```

## Set it up

1. **Credentials** (*Credentials → Create → Header Auth*), two of them:

   | Credential name | Header name | Header value |
   |---|---|---|
   | `CEKIVA API key` | `Authorization` | `Bearer tplk_...` (create the key in CEKIVA under *Settings → API keys*) |
   | `CEKIVA webhook secret` | `X-Webhook-Secret` | a long random string; every caller must send it |

2. Import `workflow.json`. Open the nodes that show a red credential warning (*Webhook*, *Resolve workspace*, *Dispatch: POST /posts*) and select the credentials.
3. In the **Config** node, `cekiva_base_url` defaults to `https://cekiva.com/api/v1`. Leave it.
4. **Activate** the workflow.

> The webhook is protected by the secret on purpose: an open webhook would let anyone spend your credits.

## Call it

```bash
curl -X POST https://YOUR-N8N/webhook/cekiva-publish \
  -H "X-Webhook-Secret: YOUR_SECRET" \
  -H "Content-Type: application/json" \
  -d '{
    "text": "Our spring collection is live!",
    "platforms": ["linkedin", "instagram"],
    "media_urls": ["https://example.com/spring.jpg"],
    "scheduled_at": "2027-03-01T09:00:00Z"
  }'
```

## Inputs

| Field | Required | Meaning |
|---|---|---|
| `text` | yes (may be empty only when `media_urls` is given) | The post text. |
| `platforms` | yes | Array or comma-separated string of `linkedin`, `tiktok`, `youtube`, `pinterest`, `bluesky`, `discord`, `facebook_page`, `instagram`, `threads`, `facebook_reels`. `facebook` is accepted for `facebook_page`. X (Twitter) is rejected: CEKIVA retired it. |
| `media_urls` | no | Public `http(s)` image or video URLs. One URL is sent as `image_url`, several as a carousel. |
| `workspace_id` | no | UUID of one of your client workspaces (listed by `GET /v1/clients`). Omit it to use your primary workspace. |
| `scheduled_at` | no | ISO 8601 timestamp **with a time zone**, in the future. Omit it to publish right away. |
| `require_approval` | no | `true` to hold the post until your client approves it on their approval link. |
| `ai_generated` | no | `true` sets TikTok's "AI-generated" label. It is never set automatically: it is always your call. |

## Answers

Success (HTTP 200):

```json
{ "status": "SUCCESS", "post_ids": ["…"], "posts": [{ "id": "…", "platform": "linkedin", "status": "scheduled", "scheduled_for": "…", "credits_charged": 10 }], "total_credits_charged": 10, "workspace": null }
```

Error (HTTP 4xx/5xx):

```json
{ "status": "ERROR", "http_status": 402, "error": { "kind": "INSUFFICIENT_CREDITS", "code": "INSUFFICIENT_CREDITS", "stage": "dispatch", "upstream_status": 402, "message": "Not enough credits! …", "hint": "Top up the credits of the account that pays for this workspace …" } }
```

`kind` is the coarse class (`VALIDATION`, `AUTH`, `INSUFFICIENT_CREDITS`, `FORBIDDEN`, `NOT_FOUND`, `TRIAL_LIMIT`, `SERVER_ERROR`, `NETWORK`); `code` is CEKIVA's specific error code. Both are stable `UPPERCASE` identifiers. `message` and `hint` are English text from CEKIVA.

## Notes

- A repeated call creates a second post (no idempotency key).
- Instagram and Pinterest need an image, TikTok and YouTube a video; CEKIVA answers `IMAGE_REQUIRED` / `VIDEO_REQUIRED` otherwise.
- Posting costs credits when it is scheduled; the workspace owner pays. See [cekiva.com](https://cekiva.com/?utm_source=github&utm_medium=template_repo&utm_campaign=n8n_starter) for current prices.
- To use another trigger (Google Sheets, a schedule, another workflow), replace the *Webhook* node: *Validate & Prepare* also reads the same fields straight from the incoming items.
