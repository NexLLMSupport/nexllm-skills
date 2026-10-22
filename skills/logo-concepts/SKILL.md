---
name: logo-concepts
description: >
  Generate logo and icon concept stills via NexLLM MCP generate_image.
  Use for "logo ideas", "icon concepts", "brand mark sketch", "app icon
  mock", or "logo exploration". Not true SVG or vector delivery.
---

# Logo concepts

Outcome skill: **raster concept stills** for marks, icons, and simple lockups. Useful for exploration — **not** production SVG, font files, or brand-guideline packages.

## Prerequisites

- NexLLM MCP connected (`https://mcp.nexllm.ai/mcp`)
- Wallet balance for `generate_image`
- See `nexllm-mcp` and https://docs.nexllm.ai/integrations/mcp-integration

## Tools

- `list_models` — required before choosing an image model
- `model_pricing` — optional; may return empty for some image models
- `generate_image` — required to produce the still

## Inputs to collect

- Brand or project name (and whether wordmark text should appear)
- Industry / audience
- Style keywords (geometric, wordmark, monogram, mascot, flat, line)
- Color preferences and backgrounds (often plain white or transparent-*looking* — still a raster PNG/JPG via URL)
- Must-avoid motifs (competitor looks, cliché symbols) if stated
- How many concepts (`n` if the model accepts it)
- Aspect or `size` / `quality` if stated

## Models

1. Call `list_models` and pick a live **image** model available to this key.
2. Filter by id/name heuristics; do not trust category filters alone.
3. **Suggested default after discovery** when present: `byteplus/Dola-Seedream-5.0-flash`.
4. Optionally call `model_pricing`; warn if empty.
5. Stop if no image model is available.

## `generate_image` result shape

JSON-in-text with `data[].url` or `data[].b64_json`. Arguments: `model`, `prompt`, optional `n` / `size` / `quality` / `response_format`. No SVG tool, no vector export, no reference logo upload.

## Workflow

1. Collect name, style, and constraints; confirm the user accepts **raster concepts**, not true SVG.
2. Discover an image model with `list_models` (prefer Seedream flash when listed).
3. Optionally price with `model_pricing`.
4. Call `generate_image` with a prompt for a clean mark: simple shapes, high contrast, centered, minimal clutter; include wordmark text only if requested (quote exact spelling).
5. Parse JSON-in-text; return image(s) and model id.
6. Offer another round of concepts with varied style keywords — still raster only.

## Related skills

- `text-in-image` — when poster/packaging copy is the focus
- `game-assets-2d` — game/UI stickers and sprites (also raster)
- `nexllm-mcp` — foundation tools and errors

## Errors

See `nexllm-mcp` for auth vs tool error codes.

## Refuse

- Claiming **true SVG**, EPS, font files, or infinite-scale vectors from this MCP tool.
- Video, audio, 3D, or LoRA training.
- Copying a third-party trademark the user did not own / clear.
- Hardcoding a model without `list_models`.

## Quality bar

- User was told output is concept **raster**, not production SVG.
- Prompt keeps the mark simple and usable as a starting point.
- Model id came from live discovery; result parsed from JSON-in-text.
