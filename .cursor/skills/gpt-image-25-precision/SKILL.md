---
name: gpt-image-25-precision
description: >
  Produce high-precision, detail-heavy stills via NexLLM MCP generate_image,
  preferring GPT Image 2.5 models after discovery. Use for "high precision
  image", "detailed still", "gpt-image-2.5", "fine detail render", or
  "precise product still".
---

# GPT Image 2.5 precision

Outcome skill: stills where **fine detail** matters — materials, small geometry, edges, and restrained clutter. Prompt-only. No video, no reference-image upload, no mask.

## Prerequisites

- NexLLM MCP connected (`https://mcp.nexllm.ai/mcp`)
- Wallet balance for `generate_image`
- See `nexllm-mcp` and https://docs.nexllm.ai/integrations/mcp-integration

## Tools

- `list_models` — required before choosing an image model
- `model_pricing` — optional; may return empty for some image models
- `generate_image` — required to produce the still

## Inputs to collect

- Subject and the details that must read clearly (stitching, type, parts, texture)
- Composition (tight crop vs full object, background simplicity)
- Lighting that reveals form without melting fine edges
- What to avoid (extra objects, warped geometry, plastic sheen, invented labels)
- Aspect or `size` / `quality` if stated (free-form; model-dependent)
- Exact on-image copy only if required — otherwise keep lettering out of the prompt

## Models

1. Call `list_models` and pick a live **image** model available to this key.
2. **Do not rely on `category` / `capability` filters** — they may be no-ops. Choose by **id / name heuristics** from the returned list.
3. **Suggested defaults after discovery**, only when that exact id is in the live list:
   1. `nexllm/gpt-image-2.5-flare` (prefer this when both 2.5 ids are present, unless the user names the other)
   2. `nexllm/gpt-image-2.5-sunburst`
   3. Fallback: `nexllm/gpt-image-2`
   4. Fallback: `byteplus/Dola-Seedream-5.0-flash`
4. If none of those ids are listed, use another live image id and say the GPT Image 2.5 preference was unavailable.
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

1. Collect the subject and which details must stay sharp.
2. Discover an image model with `list_models` (prefer `nexllm/gpt-image-2.5-flare` or `nexllm/gpt-image-2.5-sunburst` when listed).
3. Optionally price with `model_pricing`.
4. Call `generate_image` with a prompt that:
   - names the specific details to resolve
   - asks for clean edges, believable materials, and a simple background unless a scene was requested
   - bans watermarks, extra logos, and invented small text
5. Parse JSON-in-text; return `url` or `b64_json` plus model id.
6. One revision pass is another `generate_image` with tighter detail language (`image-edit-with-nexllm`). Do not claim an edit or upload API.

## Related skills

- `image-edit-with-nexllm` — prompt-only revise (new generation, not a mask)
- `photoreal-stills` — camera-like realism when “photo” matters more than precision branding
- `product-hero-image` / `product-photography` — ecommerce framing
- `text-in-image` — when exact lettering is the goal
- `nexllm-mcp` — foundation tools and errors

## Errors

See `nexllm-mcp` (HTTP `401`/`403` vs tool codes such as `INSUFFICIENT_CREDITS`, `GATEWAY_KEY_INVALID`, `GATEWAY_UNAVAILABLE`).

## Refuse

- Video, audio, 3D, or voiceover.
- Claiming pixel-perfect CAD, vector, or print-ready files from this raster still.
- Hardcoding `nexllm/gpt-image-2.5-flare` or `nexllm/gpt-image-2.5-sunburst` when `list_models` did not return them.
- Reference-image upload or mask edit.

## Quality bar

- Prompt names the details that must stay legible.
- Model id came from live discovery, with 2.5 used only if listed.
- Result parsed from JSON-in-text.
- User was not promised perfect small type unless `text-in-image` was used and copy was quoted.
