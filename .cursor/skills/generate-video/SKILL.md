---
name: generate-video
description: >
  Generate a short text-to-video MP4 via NexLLM MCP generate_video.
  Use for "generate a video", "text to video", "Seedance", "Dreamina",
  or "NexLLM video clip". One tool call creates, polls, and fetches the file.
---

# Generate video

Outcome skill: one short text-to-video MP4 on NexLLM. Call the live MCP tool `generate_video` at `https://mcp.nexllm.ai/mcp`. That single call creates the task, polls until it finishes, and fetches the file. Do not call the HTTP Videos API yourself.

Spend is billed to the wallet on the connected key. There is no MCP-specific surcharge. The tool result has no usage or USD fields — do not invent a billed amount.

## Prerequisites

- NexLLM MCP connected (`https://mcp.nexllm.ai/mcp`)
- OAuth completed with an API-key paste on `connect.nexllm.ai`
- Wallet balance for a billed video job
- Scope `inference:execute` (the same scope as `generate_text` and `generate_image`)
- See `nexllm-mcp` and https://docs.nexllm.ai/integrations/mcp-integration

Confirm the tool is available: `list_capabilities` includes `video-generation`, or `tools/list` includes `generate_video`. Audio stays unsupported.

## Tools

- `list_models` — required before choosing a catalog id
- `list_capabilities` — optional; `video-generation` is supported, audio is not
- `model_pricing` — optional; video rows may be empty. Empty is not a price of zero
- `generate_video` — required. The tool validates `seconds`, creates the task, polls, and GETs content

These catalog ids are not chat or image models. Do not send them to `generate_text` or `generate_image`. Do not `POST /v1/chat/completions`. Do not use the legacy path `POST /v1/video/generations`. Do not send a NexLLM key to a BytePlus host.

BytePlus is the NexLLM channel / LLM partner whose published duration policy covers this route.

## Inputs to collect

- Scene / action (subject, motion, setting)
- Duration, if the user named one (see family rules below)
- Aspect if stated (`16:9`, or `adaptive`). When they did not name one, pass `16:9`
- Watermark (default: request `watermark: false`)
- Resolution: **default and recommend `480p`**. Use `720p` or `1080p` only when the user clearly asks and accepts the higher reservation
- Family preference if they named one (Seedance 1.0 vs Dreamina)

If the user does not specify duration, omit `seconds` or pass **5**. The tool sends **5** when `seconds` is omitted. Never send `-1` unless the user explicitly wants a Dreamina model to choose the duration. If they ask for the cheapest or shortest clip, use the family minimum below instead of 5.

## Models

1. Call `list_models` and pick a live **video** id for this key.
2. **Do not rely on `category` / `capability` filters** — they may be no-ops. Choose by **id / name heuristics** (`Seedance`, `Dreamina`, `byteplus/`) from the returned list.
3. **Suggested defaults after discovery**, only when that exact id is in the live list (cheapest legal 480p clip first):
   1. `byteplus/ByteDance-Seedance-1.0-pro-fast` (minimum **2** seconds)
   2. `byteplus/ByteDance-Seedance-1.0-pro` (minimum **2** seconds)
   3. `byteplus/Dreamina-Seedance-2.0-mini` (minimum **4** seconds)
   4. `byteplus/Dreamina-Seedance-2.0-fast` (minimum **4** seconds)
   5. `byteplus/Dreamina-Seedance-2.0` (minimum **4** seconds)
   6. `byteplus/Dreamina-Seedance-2.5` (minimum **4** seconds)
4. Never hardcode an id `list_models` did not return. Never use mock IDs (`mock-nexllm-*`).
5. Optionally call `model_pricing`. If `data` is empty, warn that there is no quote and that the video tool result will not include USD, then continue only if the user accepts wallet spend without a price.
6. If no video model is available, say so and stop.

## Duration rules (mandatory)

Use **`seconds`**. Omitted `seconds` send **5**, never `-1`.

| Family | Catalog ids | Allowed `seconds` | Cheapest legal duration |
| --- | --- | --- | --- |
| Seedance 1.0 pro / pro-fast | `byteplus/ByteDance-Seedance-1.0-pro-fast`, `byteplus/ByteDance-Seedance-1.0-pro` | 2–12 | **2** |
| Dreamina 2.0 / fast / mini | `byteplus/Dreamina-Seedance-2.0-mini`, `byteplus/Dreamina-Seedance-2.0-fast`, `byteplus/Dreamina-Seedance-2.0` | 4–15, or `-1` | **4** |
| Dreamina 2.5 | `byteplus/Dreamina-Seedance-2.5` | 4–30, or `-1` | **4** |

These ranges follow the published duration policy of BytePlus, the NexLLM channel / LLM partner for this route.

`seconds: 2` is valid for Seedance 1.0 pro and pro-fast. The tool **rejects** `seconds: 2` for every Dreamina id locally, before any gateway call, so that shape is not billed. Do not retry a rejected Dreamina 2-second call at the same duration.

Do not pass `seconds: -1` on Seedance. Do not pass `seconds: -1` on Dreamina unless the user explicitly wants the model to choose a duration — that choice can be longer and more expensive than the minimum.

## Resolution

Default and recommend **`480p`**. Pass `size: "480p"` unless the user explicitly wants `720p` or `1080p`.

If `size` is omitted, the tool still sends `480p` on `size`, `metadata.resolution`, and `metadata.size`, so the gateway does not reserve up to 1080p. Pass `480p` explicitly when the cheap tier should be obvious in the call.

## `generate_video` arguments

| Argument | Required | Notes |
| --- | --- | --- |
| `model` | Yes | Exact catalog id from `list_models` |
| `prompt` | Yes | Text description of the clip |
| `seconds` | Recommended | Family rules above. Omitted → the tool sends **5** |
| `size` | Recommended | `480p` (default), `720p`, or `1080p`. Omitted → the tool sends **480p** |
| `ratio` | Optional | For example `16:9`, or `adaptive` |
| `watermark` | Optional | Defaults to `false` |
| `poll_timeout_sec` | Optional | How long the tool polls. Default **180**, maximum **600**. Short 480p clips often finish in under two minutes; Dreamina can be slower |

Example for Seedance 1.0 pro-fast at the 2-second minimum, and only after `list_models` returned that id:

```json
{
  "model": "byteplus/ByteDance-Seedance-1.0-pro-fast",
  "prompt": "A red ball rolls across a table.",
  "seconds": 2,
  "size": "480p",
  "ratio": "16:9",
  "watermark": false
}
```

For a Dreamina id, use the same shape with `"seconds": 4` (or higher, within that family's range).

## Result shape

The tool returns MCP **text** content whose body is JSON. The MP4 is not inlined as base64. There is no upstream video URL. Retrieve JSON has no usage or USD — do not invent a billed amount.

```json
{
  "model": "byteplus/ByteDance-Seedance-1.0-pro-fast",
  "id": "task_...",
  "object": "video",
  "status": "completed",
  "progress": 100,
  "seconds_requested": 2,
  "size": "480p",
  "content_type": "video/mp4",
  "filename": "task_....mp4",
  "byte_length": 612000,
  "download_ref": {
    "method": "GET",
    "path": "/generated-videos/...",
    "expires_at": "2026-11-13T12:00:00.000Z",
    "content_type": "video/mp4"
  }
}
```

`id` is the gateway task id. `download_ref` is a short-lived file on `mcp.nexllm.ai`. It is not a field on the poll JSON and not an upstream provider URL. `byte_length` is whatever the tool returns; do not assume a fixed size.

## Download the MP4

When status is `completed`, fetch the clip with **GET** and the **same Bearer token** the MCP session already uses (the OAuth access token, not a NexLLM API key on this host). The reference is valid for **15 minutes**.

`GET https://mcp.nexllm.ai` + `download_ref.path` (for example `https://mcp.nexllm.ai/generated-videos/{id}`).

A successful download is HTTP 200 and `Content-Type: video/mp4`. After expiry, or if the id is unknown, the server returns HTTP 404. Unauthenticated requests are rejected. Do not use `HEAD`. Save the file before `download_ref.expires_at`.

## Workflow

1. Collect the scene. Pick a legal `seconds` value: the user's duration if it is inside the family range; the family minimum if they want the cheapest or shortest clip; otherwise omit `seconds` or pass 5. Never substitute `-1` for "unspecified".
2. Discover a video model with `list_models` (local heuristics, not category filters). Prefer the cheapest suitable id that is actually listed.
3. Optionally call `model_pricing`. If empty, warn and continue only with the user's OK. Do not invent a USD figure.
4. Call `generate_video` with the exact model id, the prompt, the legal `seconds`, and `size: "480p"` unless the user asked for a higher tier.
5. Parse the JSON text. If status is not `completed`, or the tool returns an error or a poll timeout, report that and stop. Do not GET `download_ref` and do not invent a finished file.
6. On `completed`, GET `download_ref` with the same MCP Bearer before the 15-minute expiry. Return the model id, task id, and the saved MP4. Do not claim an upstream video URL or a billed USD amount.

## Errors

See `nexllm-mcp` for auth vs tool codes (`INSUFFICIENT_CREDITS`, `GATEWAY_KEY_INVALID`, `GATEWAY_UNAVAILABLE`, `GATEWAY_LIVE_MISCONFIGURED`, `INVALID_TOOL_INPUT`, `TOOL_EXECUTION_FAILED`). Scopes: `models:read`, `pricing:read`, `inference:execute`.

- Dreamina `seconds: 2` is rejected locally, before the gateway, and is not billed. Retry only at 4 or higher, and only with user OK, because the new shape is a billed job.
- Poll timeout means the tool waited up to `poll_timeout_sec` (default 180, max 600). Report the timeout; do not claim the clip completed.

## Related skills

- `nexllm-mcp` — foundation tools, including `generate_video` and `list_capabilities`
- `nexllm-models` — discover / compare models and pricing
- Image outcome skills (`product-hero-image`, `photoreal-stills`, and others) — stills via `generate_image`, not video

## Refuse

- Inventing MCP tools beyond the live set. Audio, music, voiceover, 3D, and TTS are not on this server.
- Calling `generate_text` or `generate_image` with these video catalog ids.
- Calling `POST /v1/videos`, polling, or `GET /v1/videos/{id}/content` yourself, or calling `POST /v1/video/generations`. `generate_video` already creates, polls, and fetches content.
- Sending a NexLLM API key to a BytePlus host.
- `seconds: 2` on any Dreamina id.
- `seconds: -1` unless the user explicitly wants a Dreamina model to choose the duration.
- A higher resolution than `480p` unless the user clearly accepts that reservation.
- Hardcoding a model id `list_models` did not return.
- Inventing a billed USD amount. The tool JSON has no price.
- Treating `download_ref` as an upstream video URL, inlining base64, downloading without the same MCP Bearer, using `HEAD`, or fetching after the 15-minute expiry.

## Quality bar

- The path is one `generate_video` call, then a same-auth GET of `download_ref` when the user needs the file.
- `seconds` respects the family range. Omitted duration stays at the tool's 5, not `-1`.
- `size` is `480p` unless the user asked for a higher tier.
- The model id came from `list_models` for this key.
- The result is an MP4 or a clear failure, with no invented price and no invented video URL.
