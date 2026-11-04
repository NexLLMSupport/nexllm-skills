---
name: seedream-pro-layouts
description: >
  Compose layout, multi-element, and poster-like stills via NexLLM MCP
  generate_image, preferring Seedream 5.0 pro after discovery. Use for
  "poster layout", "multi-element still", "seedream pro", "layout
  composition", or "poster-like image".
---

# Seedream pro layouts

Outcome skill: **one still** with several elements arranged on purpose — poster-like frames, multi-item layouts, simple grids. Prompt-only. Not a layout app, not editable layers, and not a mask composite of user files.

## Prerequisites

- NexLLM MCP connected (`https://mcp.nexllm.ai/mcp`)
- Wallet balance for `generate_image`
- See `nexllm-mcp` and https://docs.nexllm.ai/integrations/mcp-integration

## Tools

- `list_models` — required before choosing an image model
- `model_pricing` — optional; may return empty for some image models
- `generate_image` — required to produce the still

## Inputs to collect

- Canvas purpose (poster, social card, menu-like board, product lineup)
- **Each element** and where it sits (top band, left column, center hero, footer)
- Hierarchy (what is largest, what is secondary)
- Palette, spacing, and background
- Exact strings that must appear, quoted; otherwise ban extra text
- Aspect or `size` / `quality` if stated (free-form; model-dependent)

## Models

1. Call `list_models` and pick a live **image** model available to this key.
2. **Do not rely on `category` / `capability` filters** — they may be no-ops. Choose by **id / name heuristics** from the returned list.
3. **Suggested defaults after discovery**, in order, only when that exact id is in the live list:
   1. `byteplus/Dola-Seedream-5.0-pro`
   2. A Seedream 5.0 **lite** id from the same list. The expected form is `byteplus/Dola-Seedream-5.0-lite` — use the **exact** string returned. Do not invent a suffix if lite is absent.
   3. `byteplus/Dola-Seedream-5.0-flash`
4. If none of those are listed, pick another live image id and say the Seedream pro preference was unavailable.
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

or `data: [{ "b64_json": "..." }]`. Parse JSON; prefer `url`. Arguments: `model`, `prompt`, optional `n` / `size` / `quality` / `response_format`. No reference image, layer, or mask input.

## Workflow

1. Lock the element list and the layout in words. If exact copy is the main goal, prefer `text-in-image` (or quote every string here and keep the layout simple).
2. Discover an image model with `list_models` (prefer `byteplus/Dola-Seedream-5.0-pro` when listed, then lite, then flash).
3. Optionally price with `model_pricing`.
4. Call `generate_image` with a prompt that:
   - names each region and what sits in it
   - states scale order and margins
   - puts required words in quotes and forbids extra slogans
   - asks for a poster-like or multi-element still, not a collage of unrelated scenes
5. Parse JSON-in-text; return `url` or `b64_json` plus model id.
6. Revise by restating the layout with fewer elements if the frame is crowded — another `generate_image` call, not an edit API. Do not claim artboards or layers.

## Related skills

- `text-in-image` — exact on-image copy is the primary goal
- `composite-scene` — one scene that merges product, subject, and backdrop
- `product-hero-image` — single-product hero instead of a multi-element layout
- `nexllm-mcp` — foundation tools and errors

## Errors

See `nexllm-mcp` (HTTP `401`/`403` vs tool codes such as `INSUFFICIENT_CREDITS`, `GATEWAY_KEY_INVALID`, `GATEWAY_UNAVAILABLE`).

## Refuse

- Video, audio, 3D, or voiceover.
- Claiming Figma, layered PSD, or editable text boxes.
- Masked composites from uploaded cutouts.
- Hardcoding `byteplus/Dola-Seedream-5.0-pro` (or lite) when `list_models` did not return that id.

## Quality bar

- Prompt places each requested element in a named region.
- Model id came from live discovery; pro was used only if listed.
- Result parsed from JSON-in-text.
- Required words were quoted; extra text was discouraged.
