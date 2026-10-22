---
name: product-hero-image
description: >
  Create a product hero or packshot-style still image via NexLLM MCP.
  Use for "product photo", "hero image", "packshot", "marketing still",
  "white background product shot", or "generate an image on NexLLM".
---

# Product hero image

Outcome skill: one strong still image for product / marketing use. Text and image only — no video.

## Prerequisites

- NexLLM MCP connected (`https://mcp.nexllm.ai/mcp`)
- Wallet balance for `generate_image`
- See `nexllm-mcp` and https://docs.nexllm.ai/integrations/mcp-integration

## Tools

- `list_models` — required before choosing an image model
- `model_pricing` — optional; may return empty for some image models
- `generate_image` — required to produce the still

## Inputs to collect

- Product description (what it is, materials, notable details)
- Background / scene (e.g. clean white, soft gradient, lifestyle)
- Aspect or size preference if the user has one (`size` / `quality` are free-form strings and model-dependent — no fixed enum)
- Brand constraints (no extra logos, limited on-image text, color palette)
- Reference notes the user already provided (do not invent brand claims)

## Models

1. Call `list_models` and pick a live **image** model available to this key.
2. **Do not rely on `category` / `capability` filters** — they may be no-ops. Choose by **id / name heuristics** from the returned list (e.g. ids containing `image`, `gpt-image`, `seedream`, `dall-e`, `flux`, etc. when present).
3. Example families that *may* appear (always verify live — never hardcode as the only choice): gateway image IDs such as `nexllm/gpt-image-2` or other image-capable IDs returned by discovery. **Suggested default after discovery** when present: `byteplus/Dola-Seedream-5.0-flash`. Never use mock IDs (`mock-nexllm-*`).
4. Optionally call `model_pricing` on the chosen ID. If `data` is **empty**, warn that there is no quote, then continue only if the user accepts wallet spend without a price.
5. If no image model is available to the key, say so and stop. Do not fall back to a fake ID.

## `generate_image` result shape

The tool returns MCP **text** content containing JSON — **not** image content blocks:

```json
{
  "model": "<id>",
  "created": 0,
  "data": [{ "url": "https://..." }]
}
```

or `data: [{ "b64_json": "..." }]`.

- Parse the JSON from the text payload.
- Prefer `url`: fetch or present the URL to the user.
- If only `b64_json`, decode/present carefully; do not assume every environment returns a real PNG.
- Do **not** wait for MCP image blocks.

Default `response_format` is typically `url` when the server supports it; pass `b64_json` only if the user/tool path needs inline data.

## Workflow

1. Collect enough product and style detail for a concrete prompt.
2. Discover an image model with `list_models` (local heuristics, not category filters).
3. Optionally price with `model_pricing`; warn and continue if empty.
4. Call `generate_image` with:
   - the exact model `id` from discovery
   - a specific prompt (subject, lighting, background, composition, constraints)
   - `size` / `quality` only when the user asked and the model accepts those free-form strings
5. Parse JSON-in-text; return the image via `url` or `b64_json`, plus the model id used.
6. Offer one revision pass if the user wants a tweak — re-run `generate_image` with an updated prompt; re-check `list_models` only if the previous model failed or disappeared.

## Errors

See `nexllm-mcp` for dual surfaces:

- HTTP `401` / `403` (auth, scope, connection) → reconnect OAuth / check scopes `models:read`, `pricing:read`, `inference:execute`
- Tool codes: `INSUFFICIENT_CREDITS`, `GATEWAY_KEY_INVALID`, `GATEWAY_UNAVAILABLE`, `GATEWAY_LIVE_MISCONFIGURED`, `INVALID_TOOL_INPUT`, `TOOL_EXECUTION_FAILED`

## Related skills

- `product-photography` — broader studio / lifestyle / hero product stills
- `photoreal-stills` — realism-first look
- `composite-scene` — prompt-only product + subject + backdrop
- `text-in-image` — when exact on-image copy is required
- `nexllm-mcp` — foundation tools and errors

## Refuse

- Video, motion, voiceover, 3D mesh, or audio — explain those MCP tools are not available.
- Hardcoding a single model forever without `list_models`.
- Claiming an MCP image content block was returned when only JSON-in-text arrived.

## Quality bar

- Prompt is specific enough to avoid generic stock junk.
- Model id was live for the connected key.
- Result was parsed from JSON (`url` or `b64_json`).
- On-image text is avoided unless the user explicitly wants a label.
