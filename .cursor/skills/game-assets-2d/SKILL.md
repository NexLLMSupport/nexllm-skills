---
name: game-assets-2d
description: >
  Make 2D game-style sprites, stickers, and consistent asset-set stills via
  NexLLM MCP generate_image. Use for "game sprite", "sticker sheet",
  "2D game asset", "icon set for game", "pixel-ish character", or
  "UI game art".
---

# Game assets 2D

Outcome skill: **raster** 2D game/UI art — sprites, stickers, simple prop sheets. Prompt-only stills; no sprite atlas packing, no animation timeline, no reference sheet upload.

## Prerequisites

- NexLLM MCP connected (`https://mcp.nexllm.ai/mcp`)
- Wallet balance for `generate_image`
- See `nexllm-mcp` and https://docs.nexllm.ai/integrations/mcp-integration

## Tools

- `list_models` — required before choosing an image model
- `model_pricing` — optional; may return empty for some image models
- `generate_image` — required to produce the still

## Inputs to collect

- Asset type (character sprite, prop, sticker, tile-like icon, UI badge)
- Art style (pixel-inspired, flat cel, chibi, painterly — describe in words)
- Consistency notes (palette, outline weight, view: front/side)
- Transparent / plain background preference (describe in prompt; delivery is still raster via URL)
- Count of variants (`n` if supported) or “sheet of N poses” described in one prompt
- Aspect or `size` / `quality` if stated
- Exact on-asset text only if needed (otherwise avoid lettering)

## Models

1. Call `list_models` and pick a live **image** model available to this key.
2. Filter by id/name heuristics; do not trust category filters alone.
3. **Suggested default after discovery** when present: `byteplus/Dola-Seedream-5.0-flash`.
4. Optionally call `model_pricing`; warn if empty.
5. Stop if no image model is available.

## `generate_image` result shape

JSON-in-text with `data[].url` or `data[].b64_json`. Arguments: `model`, `prompt`, optional `n` / `size` / `quality` / `response_format`. No spritesheet compiler, no animation, no reference image input.

## Workflow

1. Collect asset type, style, and consistency constraints.
2. Discover an image model with `list_models` (prefer Seedream flash when listed).
3. Optionally price with `model_pricing`.
4. Call `generate_image` with a prompt that specifies style, view, palette, plain/transparent-looking background, and “game asset / sticker” framing.
5. Parse JSON-in-text; return image(s) and model id.
6. For a “set”, either use `n` when accepted or run sequential prompts with the same style block for consistency — do not claim automatic character-ID locking.

## Related skills

- `logo-concepts` — brand marks (also raster concepts)
- `text-in-image` — when HUD/copy readability is primary
- `nexllm-mcp` — foundation tools and errors

## Errors

See `nexllm-mcp` for auth vs tool error codes.

## Refuse

- Video, animated GIF pipelines, spine/rig export, or 3D meshes.
- Claiming packed texture atlases or engine-ready sprite sheets as a native tool output.
- Hardcoding a model without `list_models`.

## Quality bar

- Style and view are explicit in the prompt.
- Model id came from live discovery.
- Result parsed from JSON-in-text.
- User understands delivery is still image URL(s), not an asset pipeline.
