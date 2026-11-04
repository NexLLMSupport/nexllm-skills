---
name: fast-stickers-gemini
description: >
  Make fast sticker, icon, and emoji-style stills via NexLLM MCP
  generate_image, preferring Gemini flash image models after discovery. Use
  for "fast sticker", "emoji sticker", "gemini flash sticker", "icon still",
  or "quick sticker sheet".
---

# Fast stickers (Gemini)

Outcome skill: **quick raster stickers, icons, and emoji-style stills**. Prompt-only. No die-cut pipeline, no emoji-font export, no reference sheet upload. Prefer a Gemini **flash** image model when `list_models` lists one, so the run stays on the faster image ids in that family.

## Prerequisites

- NexLLM MCP connected (`https://mcp.nexllm.ai/mcp`)
- Wallet balance for `generate_image`
- See `nexllm-mcp` and https://docs.nexllm.ai/integrations/mcp-integration

## Tools

- `list_models` — required before choosing an image model
- `model_pricing` — optional; may return empty for some image models
- `generate_image` — required to produce the still

## Inputs to collect

- Subject (one character, object, or symbol per still unless they asked for a sheet)
- Style (emoji, die-cut sticker look, flat icon, thick outline)
- Background (plain, sticker-white, simple color) — describe it; transparency is not guaranteed
- Palette and outline weight
- Count (`n` if accepted, or “one sheet of N” inside a single prompt)
- Exact lettering only if needed (otherwise say no text)
- Aspect or `size` / `quality` if stated

## Models

1. Call `list_models` and pick a live **image** model available to this key.
2. **Do not rely on `category` / `capability` filters** — they may be no-ops. Choose by **id / name heuristics** from the returned list.
3. **Suggested defaults after discovery**, in order. Use an id only when `list_models` returned it. Match on the name if the gateway prefix differs, and pass the **exact** id string from the list:
   1. `gemini-3.1-flash-image` (expected form `nexllm/gemini-3.1-flash-image`)
   2. `gemini-3.1-flash-lite-image` (expected form `nexllm/gemini-3.1-flash-lite-image`)
   3. Fallback in the same family: `nexllm/gemini-3-pro-image`
4. If none of those names are listed, pick another live image id and say the Gemini flash preference was unavailable. `byteplus/Dola-Seedream-5.0-flash` is acceptable when it is actually present.
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

or `data: [{ "b64_json": "..." }]`. Parse JSON; prefer `url`. Arguments: `model`, `prompt`, optional `n` / `size` / `quality` / `response_format`. No reference image input.

## Workflow

1. Collect subject, sticker/emoji/icon style, and background.
2. Discover an image model with `list_models` (prefer a Gemini flash image id — `gemini-3.1-flash-image`, then `gemini-3.1-flash-lite-image` — using the exact string returned).
3. Optionally price with `model_pricing`.
4. Call `generate_image` with a prompt for a single readable mark or a simple sheet: bold shapes, limited palette, centered, plain background, “sticker / emoji / icon,” and no extra text unless quoted.
5. Parse JSON-in-text; return `url` or `b64_json` plus model id.
6. Variants are another prompt with the same style block, or `n` when the model accepts it. Do not claim character-ID locking or a cut-path file.

## Related skills

- `game-assets-2d` — broader sprites and game UI stills
- `logo-concepts` — brand marks (raster concepts, not SVG)
- `nano-banana-design` — design / concept stills on Gemini pro image when listed
- `nexllm-mcp` — foundation tools and errors

## Errors

See `nexllm-mcp` (HTTP `401`/`403` vs tool codes such as `INSUFFICIENT_CREDITS`, `GATEWAY_KEY_INVALID`, `GATEWAY_UNAVAILABLE`).

## Refuse

- Video, animated sticker pipelines, or 3D.
- Claiming a true transparent PNG, die-line, or emoji-font file as native tool output.
- Hardcoding a Gemini flash id that was not in `list_models`.
- Reference-image upload.

## Quality bar

- Prompt is sticker / icon / emoji shaped, with a plain background.
- Flash Gemini id was used only if discovery listed it.
- Result parsed from JSON-in-text.
- User understands delivery is a raster URL, not an asset pipeline.
