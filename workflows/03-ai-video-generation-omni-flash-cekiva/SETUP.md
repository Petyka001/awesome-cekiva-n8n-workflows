# 03 · AI video generation (Gemini Omni 1.1 Flash) → CEKIVA

Once a day an AI invents a short, shareable video idea, writes the video prompt, **CEKIVA generates the video** with Gemini Omni 1.1 Flash, and CEKIVA **publishes it** to TikTok, YouTube, Instagram and Facebook Reels. Two API calls do the work that used to need a video vendor, a file host and one HTTP node per network.

```
Schedule → AI idea → AI video prompt → CEKIVA: start video → (wait 30 s → status → ready?) × n → CEKIVA: POST /v1/posts → results
```

## What changed from the community template

This workflow is derived from the community template *"Automate video creation with Veo3 and auto-post to Instagram, TikTok via Blotato"* (n8n.io; see the original page for its author and terms). Original: 29 nodes, this one: 22.

| Original | Now |
|---|---|
| Video from `fal-ai/veo3` on fal.ai (needs a fal.ai key; Veo 3.1 preview models are shut down by Google on 2026-10-22) | `POST /v1/generate-video` on CEKIVA: **Gemini Omni 1.1 Flash**, no fal.ai or Google key |
| One fixed 5-minute wait, then one fetch | A 30-second poll loop on the job status; it stops with a clear error if the job fails or takes longer than about 15 minutes |
| Blotato: media upload + 9 platform nodes with account ids you had to paste | one `POST /v1/posts` per platform; accounts are the ones you connected in CEKIVA |
| Google Sheets as the idea/video log (3 nodes + a sheet to prepare) | removed; a small *Idea fields* code node passes the idea on. Add a Sheets node after *Label CEKIVA results* if you want a log |
| X (Twitter) | gone: CEKIVA retired that channel |
| OpenAI (GPT-4.1) writes the idea and the prompt | unchanged: it needs your OpenAI credential |

## Set it up

1. In CEKIVA: connect your accounts and create an API key (*Settings → API keys*, starts with `tplk_`).
2. In n8n create **one** credential used three times: *Credentials → Create → Header Auth*, name `CEKIVA API key`, header name `Authorization`, value `Bearer tplk_...`.
3. Import `workflow.json`. Select `CEKIVA API key` in **CEKIVA: start video**, **CEKIVA: video status** and **CEKIVA: POST /v1/posts**, and your OpenAI credential in the two *LLM* nodes.
4. Adjust, if you like:
   - the video: the body of **CEKIVA: start video** (`quality`: `720p`, `1080p` or `4k`; `duration_seconds`: `4`, `6` or `8`; `aspect_ratio`: `9:16` or `16:9`);
   - the publishing: the `CONFIG` block at the top of **Build CEKIVA posts** (`PLATFORMS`, `CLIENT_ID` for a client workspace, `PUBLISH_NOW` / `SCHEDULED_AT`);
   - the idea: the topic in **AI Agent: Generate Video Concept** (the original asks for a yeti vlogger).
5. Run it once by hand before activating the schedule.

Remove the platforms you have not connected from `PLATFORMS`: a platform CEKIVA cannot post to comes back as a row with an error code, the other platforms are not affected.

## Inputs and outputs

There is no external input: the schedule trigger starts it. The output is one row per platform from **Label CEKIVA results**:

```json
{ "platform": "tiktok", "result": "Success", "http_status": 201, "post_id": "…", "post_status": "scheduled", "credits_charged": 0, "error_code": null, "error_message": null, "hint": null }
```

or, on failure, `"result": "Error"` with CEKIVA's `error_code` (e.g. `INSUFFICIENT_CREDITS`), `error_message` and `hint`. Branch on the code, not on the text.

## Good to know

- **Credits.** Generating the video and every post cost credits from your CEKIVA wallet. The price is shown in CEKIVA and set there, not in this workflow. A video that fails is refunded by CEKIVA; if the clip comes back shorter than ordered, the missing seconds are refunded.
- **Videos stay on CEKIVA.** TikTok can only fetch a video that CEKIVA hosts, which a generated video is. This is why the video is not downloaded and re-uploaded anywhere.
- **The "AI-generated" label is your decision.** `AI_LABEL` in *Build CEKIVA posts* is `false` by default; set it to `true` to have TikTok mark the video as AI-generated. A generated video that you publish as if it were real footage is your responsibility.
- Video generation always runs in your primary CEKIVA workspace; `CLIENT_ID` only moves the *posts* to a client workspace.
- A repeated run makes a new video and new posts: the API has no idempotency key.
- Based on community work; the original author's credit and terms are on its n8n.io page.
