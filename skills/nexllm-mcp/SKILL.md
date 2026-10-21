---
name: nexllm-mcp
description: >
  Use NexLLM over MCP for model discovery, pricing, text generation, and
  image generation. Trigger when the user mentions NexLLM, mcp.nexllm.ai,
  connect.nexllm.ai, wallet billing, list_models, model_pricing,
  generate_text, or generate_image.
---

# NexLLM MCP

Foundation skill for every NexLLM outcome. Skills are recipes; NexLLM MCP is what runs the work.

## Prerequisites

- NexLLM MCP connected at `https://mcp.nexllm.ai/mcp`
- OAuth completed with an API-key paste on `connect.nexllm.ai`
- Wallet balance if calling `generate_text` or `generate_image`

Docs: https://docs.nexllm.ai/tools/mcp

## Tools (only these)

| Tool | Role |
| --- | --- |
| `list_models` | Discover live model IDs for the connected key |
| `model_pricing` | Confirm cost before a paid run (when available) |
| `generate_text` | Chat / completion through the gateway |
| `generate_image` | Image generation through the gateway |

Do **not** invent other MCP tool names. Do **not** call video, audio, 3D, TTS, or ComfyUI tools — they are not on this server.

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

### `list_models`

- Prefer this over memorized IDs.
- Documented `category` / `capability` filters may be **no-ops** until the catalog exposes those fields.
- **Do not rely on filters.** Load the list and pick by **id / name heuristics** (e.g. strings that look like chat vs image models) from what was actually returned.
- Never hardcode mock IDs (`mock-nexllm-*` and similar are not live production IDs).

### `model_pricing`

- Returns a list shaped like `{object:"list", data:[{model, currency, input_price_per_million, output_price_per_million}, ...]}` when rows exist (USD per 1M tokens when applicable).
- Some models (especially **image** / unpriceable `billing_expr` rows) may yield an **empty** `data` list — that is not always a hard failure.
- If pricing is empty: **warn the user**, then it is still OK to call `generate_text` / `generate_image` if they approve spend without a quote.

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

1. Prefer `list_models`; filter locally by id/name — do not trust category/capability filters alone.
2. Call `model_pricing` when the user cares about cost; if empty for that model, warn and ask before generating.
3. Pass the **exact** model `id` from discovery into `generate_text` or `generate_image`.
4. Parse JSON-in-text results; for images, use `data[].url` or `data[].b64_json` as above.
5. If MCP is not connected, say so and point at https://docs.nexllm.ai/tools/mcp — do not pretend a call succeeded.
6. If the user asks for **video, audio, music, or 3D**, refuse clearly: those MCP tools are not available yet.

## Related skills

- `nexllm-models` — discover and compare models / pricing
- `draft-with-nexllm` — text drafts via `generate_text`
- `product-hero-image` — product / hero stills via `generate_image`
