---
name: product-photography
description: >
  Turn a product description into studio, lifestyle, and hero stills via
  NexLLM MCP generate_image. Use for "product photography", "studio packshot",
  "lifestyle product shot", "ecommerce photo", "catalog still", or
  "product on white".
---

# Product photography

Outcome skill: studio, lifestyle, and hero stills for a product. Text and image only — no video, no reference-image upload, no masking API.

## Prerequisites

- NexLLM MCP connected (`https://mcp.nexllm.ai/mcp`)
- Wallet balance for `generate_image`
- See `nexllm-mcp` and https://docs.nexllm.ai/integrations/mcp-integration

## Tools

- `list_models` — required before choosing an image model
- `model_pricing` — optional; may return empty for some image models
- `generate_image` — required to produce the still

## Inputs to collect

- Product description (what it is, materials, colors, notable details)
- Shot type: studio packshot, lifestyle scene, or marketing hero
- Background / surface (clean white, soft gradient, marble, outdoor, etc.)
- Lighting preference (softbox, daylight, dramatic rim) if stated
- Aspect or `size` / `quality` preference if the user has one (free-form strings; model-dependent — no fixed enum)
- Brand constraints (no extra logos, limited on-image text, color palette)
- Any reference notes the user already typed (do not invent brand claims; this tool has **no image input**)

## Models

1. Call `list_models` and pick a live **image** model available to this key.
2. **Do not rely on `category` / `capability` filters** — they may be no-ops. Choose by **id / name heuristics** from the returned list (e.g. ids containing `image`, `gpt-image`, `seedream`, `dall-e`, `flux`, etc. when present).
3. **Suggested default after discovery** when present in the list: `byteplus/Dola-Seedream-5.0-flash`. Always verify it appears for this key — never skip `list_models`.
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
- Arguments supported: `model`, `prompt`, optional `n` / `size` / `quality` / `response_format`. **No reference image, mask, or edit input.**

Default `response_format` is typically `url` when the server supports it; pass `b64_json` only if the user/tool path needs inline data.

## Workflow

1. Collect product, shot type, and style detail for a concrete prompt.
2. Discover an image model with `list_models` (prefer Seedream flash when listed).
3. Optionally price with `model_pricing`; warn and continue if empty.
4. Call `generate_image` with:
   - the exact model `id` from discovery
   - a specific prompt (subject, materials, lighting, background, composition, constraints)
   - `size` / `quality` / `n` only when the user asked and the model accepts those free-form strings
5. Parse JSON-in-text; return the image via `url` or `b64_json`, plus the model id used.
6. Offer one revision pass if the user wants a tweak — re-run `generate_image` with an updated prompt; re-check `list_models` only if the previous model failed or disappeared.

## Related skills

- `product-hero-image` — shorter hero/packshot-focused recipe
- `photoreal-stills` — realistic non-stock look
- `composite-scene` — prompt-only product + subject + backdrop merge
- `nexllm-mcp` — foundation tools and errors

## Errors

See `nexllm-mcp` for dual surfaces:

- HTTP `401` / `403` (auth, scope, connection) → reconnect OAuth / check scopes `models:read`, `pricing:read`, `inference:execute`
- Tool codes: `INSUFFICIENT_CREDITS`, `GATEWAY_KEY_INVALID`, `GATEWAY_UNAVAILABLE`, `GATEWAY_LIVE_MISCONFIGURED`, `INVALID_TOOL_INPUT`, `TOOL_EXECUTION_FAILED`

## Refuse

- Video, motion, voiceover, 3D mesh, or audio — explain those MCP tools are not available.
- Claiming you uploaded or masked a real product photo — this path is **prompt-only**.
- Hardcoding a single model forever without `list_models`.
- Claiming an MCP image content block was returned when only JSON-in-text arrived.

## Quality bar

- Prompt is specific enough for a usable ecommerce / marketing still.
- Model id was live for the connected key.
- Result was parsed from JSON (`url` or `b64_json`).
- On-image text is avoided unless the user explicitly wants a label.
