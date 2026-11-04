---
name: nano-banana-design
description: >
  Create design and concept stills via NexLLM MCP generate_image, preferring
  Gemini image models (Nano Banana family) after discovery. Use for "nano
  banana", "design concept still", "concept art still", "gemini image
  design", or "brand concept visual".
---

# Nano Banana design

Outcome skill: **design and concept stills** — mood frames, brand visuals, product-direction boards as a single raster. “Nano Banana” is a nickname for Gemini image models. There is **no** model id named `nano-banana`. Prompt-only; no vector export and no reference-image upload.

## Prerequisites

- NexLLM MCP connected (`https://mcp.nexllm.ai/mcp`)
- Wallet balance for `generate_image`
- See `nexllm-mcp` and https://docs.nexllm.ai/integrations/mcp-integration

## Tools

- `list_models` — required before choosing an image model
- `model_pricing` — optional; may return empty for some image models
- `generate_image` — required to produce the still

## Inputs to collect

- What the concept is for (brand, product, campaign, UI direction, key art)
- Style keywords (editorial, geometric, soft gradient, tactile, futuristic)
- Palette and what to keep out (competitor looks, clutter, fake logos)
- Whether any exact words must appear (if yes, quote them; otherwise ban extra text)
- How many directions (`n` if the model accepts it, or sequential prompts)
- Aspect or `size` / `quality` if stated

## Models

1. Call `list_models` and pick a live **image** model available to this key.
2. **Do not rely on `category` / `capability` filters** — they may be no-ops. Choose by **id / name heuristics** from the returned list.
3. **Suggested defaults after discovery**, in order. Use an id only when `list_models` returned it. Match on the name if the gateway prefix differs, and pass the **exact** id string from the list:
   1. `nexllm/gemini-3-pro-image`
   2. `gemini-3.1-flash-image` (expected form `nexllm/gemini-3.1-flash-image`)
   3. `gemini-3.1-flash-lite-image` (expected form `nexllm/gemini-3.1-flash-lite-image`)
4. If none of those names are listed, pick another live image id and say the Gemini / Nano Banana preference was unavailable. `byteplus/Dola-Seedream-5.0-flash` is acceptable when it is actually present.
5. Optionally call `model_pricing`; if empty, warn and continue only with user OK.
6. If no image model is available, say so and stop. Never use mock IDs (`mock-nexllm-*`).

## `generate_image` result shape

MCP **text** content with JSON — not image blocks:

```json
{
  "model": "<id>",
  "created": 0,
  "data": [{ "url": "https://..." }]
}
```

or `data: [{ "b64_json": "..." }]`. Parse JSON; prefer `url`. Arguments: `model`, `prompt`, optional `n` / `size` / `quality` / `response_format`. No reference image input. No SVG tool.

## Workflow

1. Collect the concept, style, and constraints. Tell the user delivery is a **raster concept still**, not a design-system file.
2. Discover an image model with `list_models` (prefer `nexllm/gemini-3-pro-image` when that id is listed).
3. Optionally price with `model_pricing`.
4. Call `generate_image` with a prompt that states subject, layout, palette, and “design concept / art direction still,” and that bans extra slogans unless the user supplied exact words.
5. Parse JSON-in-text; return `url` or `b64_json` plus model id.
6. Offer another direction with different style keywords — still one generated still per call.

## Related skills

- `fast-stickers-gemini` — faster sticker / icon / emoji stills on Gemini flash when listed
- `logo-concepts` — mark exploration (also raster, not SVG)
- `text-in-image` — when exact on-image copy is the point
- `game-assets-2d` — sprite / game UI stills
- `nexllm-mcp` — foundation tools and errors

## Errors

See `nexllm-mcp` (HTTP `401`/`403` vs tool codes such as `INSUFFICIENT_CREDITS`, `GATEWAY_KEY_INVALID`, `GATEWAY_UNAVAILABLE`).

## Refuse

- Video, audio, 3D, or voiceover.
- Claiming Figma, SVG, or editable design-file output.
- Calling a model id `nano-banana` or any Gemini id that `list_models` did not return.
- Hardcoding a model without `list_models`.

## Quality bar

- Prompt reads as a design concept, with palette and composition named.
- Preferred model was `nexllm/gemini-3-pro-image` only if discovery listed it.
- Result parsed from JSON-in-text.
- User knows the file is a raster starting point.
