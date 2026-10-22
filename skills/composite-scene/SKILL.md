---
name: composite-scene
description: >
  Merge a product, a subject, and a backdrop into one still via NexLLM MCP
  generate_image using prompt-only composition. Use for "composite scene",
  "product with model in scene", "merge subject and backdrop", or
  "lifestyle composite still". No masking or layer API.
---

# Composite scene

Outcome skill: one still that **describes** product + subject + backdrop together. This is **prompt-only composition** — there is no mask, matte, or multi-image upload on `generate_image`.

## Prerequisites

- NexLLM MCP connected (`https://mcp.nexllm.ai/mcp`)
- Wallet balance for `generate_image`
- See `nexllm-mcp` and https://docs.nexllm.ai/integrations/mcp-integration

## Tools

- `list_models` — required before choosing an image model
- `model_pricing` — optional; may return empty for some image models
- `generate_image` — required to produce the still

## Inputs to collect

- Product details (size cues, materials, color)
- Subject / person or character description (or “no person”)
- Backdrop / environment
- Interaction (holding, wearing, placed on table, floating hero)
- Lighting that unifies the scene
- Brand constraints (no extra logos; limited text)
- Aspect or `size` / `quality` if stated
- Any typed reference notes (not image files — **no image input**)

## Models

1. Call `list_models` and pick a live **image** model available to this key.
2. Filter by id/name heuristics; do not trust category filters alone.
3. **Suggested default after discovery** when present: `byteplus/Dola-Seedream-5.0-flash`.
4. Optionally call `model_pricing`; warn if empty.
5. Stop if no image model is available.

## `generate_image` result shape

JSON-in-text with `data[].url` or `data[].b64_json`. Arguments: `model`, `prompt`, optional `n` / `size` / `quality` / `response_format`. **No mask, no layers, no multi-image merge tool.**

## Workflow

1. Collect product, subject, and backdrop; tell the user this is a single generated still, not a Photoshop-style composite of their files.
2. Discover an image model with `list_models` (prefer Seedream flash when listed).
3. Optionally price with `model_pricing`.
4. Call `generate_image` with one coherent prompt that names all three elements, shared lighting, camera distance, and contact points (hands, surface, scale).
5. Parse JSON-in-text; return the image and model id.
6. Revise by adjusting scale, contact, or lighting language — do not claim inpainting/outpainting APIs.

## Related skills

- `product-photography` — product-led studio / lifestyle stills
- `photoreal-stills` — realism-first look
- `product-hero-image` — simpler hero/packshot recipe
- `nexllm-mcp` — foundation tools and errors

## Errors

See `nexllm-mcp` for auth vs tool error codes.

## Refuse

- Claiming true masked composites from user-uploaded cutouts (no image/mask input).
- Video, audio, 3D.
- Hardcoding a model without `list_models`.

## Quality bar

- Prompt unifies product, subject, and backdrop with shared light and scale.
- User understands this is generative composition, not file-based masking.
- Model id from live discovery; result parsed from JSON-in-text.
