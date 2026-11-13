---
name: nexllm-mcp
description: >
  Use NexLLM over MCP for model discovery, pricing, text generation,
  image generation, and short video. Trigger when the user mentions
  NexLLM, mcp.nexllm.ai, connect.nexllm.ai, wallet billing, list_models,
  model_pricing, list_capabilities, generate_text, generate_image, or
  generate_video.
---

# NexLLM MCP

Foundation skill for every NexLLM outcome. Skills are recipes; NexLLM MCP is what runs the work.

## Prerequisites

- NexLLM MCP connected at `https://mcp.nexllm.ai/mcp`
- OAuth completed with an API-key paste on `connect.nexllm.ai`
- Wallet balance if calling `generate_text`, `generate_image`, or `generate_video`

Docs: https://docs.nexllm.ai/integrations/mcp-integration

## Tools (only these)

| Tool | Role |
| --- | --- |
| `list_models` | Discover live model IDs for the connected key |
| `model_pricing` | Confirm cost before a paid run (when available) |
| `list_capabilities` | Which families this server supports. `video-generation` is supported. Audio and other gaps stay unsupported |
| `model_metadata` | Metadata for a catalog model. Use only fields the tool returns |
| `generate_text` | Chat / completion through the gateway |
| `generate_image` | Image generation through the gateway |
| `generate_video` | Short MP4. One call creates the task, polls, and fetches the file. See `generate-video` |

Do **not** invent other MCP tool names. Do **not** call audio, music, TTS, 3D, or ComfyUI tools — they are not on this server. Video is `generate_video` only. Do not send video catalog ids to `generate_text` or `generate_image`.

## Response shapes (important)

### `generate_image`

Returns MCP **text** content whose body is JSON, **not** image content blocks:

```json
{
  "model": "<id>",
  "created": 0,
  "data": [{ "url": "https://..." }]
}
```

or `{ "b64_json": "..." }` inside `data[]`.

- **Parse the JSON** from the text content.
- If `url` is present, the agent **may fetch** the URL to show or save the image.
- If only `b64_json` is present, decode/present carefully; do not assume it is always a real PNG in every environment.
- **Do not** expect MCP `image` content blocks from this server.

### `generate_text`

- **Single-shot only** — no stream parameter.
- Result is JSON-in-text (parse JSON from the text content): typically `{model, text, usage, id}` (field names may vary slightly; read what the tool returns).
- Do not wait for token streaming from MCP.

### `generate_video`

- **One call** creates the task, polls until `completed` or `failed` (or until the poll timeout), and fetches the file. Do not call `POST /v1/videos` or `POST /v1/video/generations` yourself.
- Result is JSON-in-text. The MP4 is **not** inlined as base64. There is **no** upstream video URL. Retrieve JSON has **no** usage or USD — do not invent a billed amount.
- On success the JSON includes `download_ref` (`method: GET`, `path` under `/generated-videos/...`, `expires_at`, `content_type: video/mp4`). That path is a short-lived file on `https://mcp.nexllm.ai`, valid for **15 minutes**. GET it with the **same Bearer** as the MCP session (the OAuth access token, not a NexLLM API key on this host). Do not use `HEAD`. After expiry the server returns HTTP 404.
- Omitted `seconds` send **5**, never `-1`. Pass `-1` only when the user explicitly wants a Dreamina model to choose the duration.
- Omitted `size` sends **480p**. Default and recommend `480p` unless the user asks for `720p` or `1080p`.
- `seconds: 2` is legal for Seedance 1.0 pro and pro-fast (allowed 2–12; cheapest legal duration is 2). The tool rejects `seconds: 2` for every Dreamina id **locally, before any gateway call**, so that shape is not billed. Dreamina 2.0 / fast / mini allow 4–15 or `-1` (cheapest legal 4). Dreamina 2.5 allows 4–30 or `-1` (cheapest legal 4).
- Catalog ids and the download steps are in skill `generate-video`. Require `list_models` before hardcoding an id. BytePlus is the NexLLM channel / LLM partner whose published duration policy covers this route.

### `list_models`

- Prefer this over memorized IDs.
- Documented `category` / `capability` filters may be **no-ops** until the catalog exposes those fields.
- **Do not rely on filters.** Load the list and pick by **id / name heuristics** (e.g. strings that look like chat vs image vs video models) from what was actually returned.
- Never hardcode mock IDs (`mock-nexllm-*` and similar are not live production IDs).

### `list_capabilities`

- Use this to confirm what this server supports.
- `video-generation` is supported (`generate_video`).
- Audio and other gaps stay unsupported. Do not invent tools for those gaps.

### `model_pricing`

- Returns a list shaped like `{object:"list", data:[{model, currency, input_price_per_million, output_price_per_million}, ...]}` when rows exist (USD per 1M tokens when applicable).
- Some models (especially **image**, **video**, or unpriceable `billing_expr` rows) may yield an **empty** `data` list — that is not always a hard failure, and it is not a price of zero.
- If pricing is empty: **warn the user**, then it is still OK to call `generate_text`, `generate_image`, or `generate_video` if they approve spend without a quote. Do not invent a billed USD amount. `generate_video` will not fill one in.

### `model_metadata`

- Live discovery tool. Read the fields it returns. Do not invent metadata it did not return.

## Auth and errors (two layers)

Handle **both** surfaces:

1. **HTTP / OAuth layer**
   - `401` + `WWW-Authenticate` — reconnect / re-authorize MCP (paste key again on `connect.nexllm.ai`).
   - `403` with `insufficient_scope`, `connection_required`, or `credential_unavailable` — fix connection/scopes; these are not the same as tool `ErrorCode` values.
2. **Tool / gateway layer** (after a connected call)
   - `INSUFFICIENT_CREDITS` — top up wallet and retry.
   - `GATEWAY_KEY_INVALID` — rotate/reconnect with a valid API key.
   - `GATEWAY_UNAVAILABLE` / `GATEWAY_LIVE_MISCONFIGURED` — transient or host config; retry later or report ops.
   - `INVALID_TOOL_INPUT` — fix arguments and retry.
   - `TOOL_EXECUTION_FAILED` — report failure; do not invent success.

**Scopes** the connection should hold for full use: `models:read`, `pricing:read`, `inference:execute`.

## Workflow

1. Prefer `list_models`; filter locally by id/name — do not trust category/capability filters alone. `list_capabilities` can confirm `video-generation` versus unsupported audio.
2. Call `model_pricing` when the user cares about cost; if empty for that model, warn and ask before generating. Do not invent a USD amount.
3. Pass the **exact** model `id` from discovery into `generate_text`, `generate_image`, or `generate_video`. Video catalog ids go only to `generate_video`.
4. Parse JSON-in-text results; for images, use `data[].url` or `data[].b64_json` as above. For video, use `download_ref` (same-auth GET, 15 minutes) and do not expect base64 or an upstream URL.
5. If MCP is not connected, say so and point at https://docs.nexllm.ai/integrations/mcp-integration — do not pretend a call succeeded.
6. If the user asks for a short video, use `generate_video` (skill `generate-video`): `list_models` first, default **480p**, family duration rules in that skill. If they ask for **audio, music, voiceover, or 3D**, refuse clearly: those MCP tools are not available.

## Related skills

- `nexllm-models` — discover and compare models / pricing
- `generate-video` — short text-to-video MP4 via `generate_video`
- `draft-with-nexllm` — text drafts via `generate_text`
- `product-hero-image` — product / hero stills via `generate_image`
- `product-photography` — studio / lifestyle / hero product stills
- `photoreal-stills` — realistic non-stock stills
- `text-in-image` — posters / packaging / UI with exact copy
- `logo-concepts` — logo / icon concept rasters (not true SVG)
- `game-assets-2d` — sprites, stickers, 2D asset stills
- `composite-scene` — prompt-only product + subject + backdrop
- `seedream-pro-layouts` — layout / multi-element / poster-like stills; prefer Seedream pro
