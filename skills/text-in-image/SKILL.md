---
name: text-in-image
description: >
  Render posters, packaging mockups, and UI-style stills with exact on-image
  copy via NexLLM MCP generate_image. Use for "poster with text", "packaging
  label", "text in image", "exact copy on image", "title card", or
  "UI mockup image".
---

# Text in image

Outcome skill: stills where **specific copy must appear on the image** (posters, packaging faces, simple UI frames). Prompt-only — no font upload, no layout engine, no SVG export.

## Prerequisites

- NexLLM MCP connected (`https://mcp.nexllm.ai/mcp`)
- Wallet balance for `generate_image`
- See `nexllm-mcp` and https://docs.nexllm.ai/integrations/mcp-integration

## Tools

- `list_models` — required before choosing an image model
- `model_pricing` — optional; may return empty for some image models
- `generate_image` — required to produce the still

## Inputs to collect

- **Exact strings** to render (quote them; do not paraphrase brand claims)
- Hierarchy (headline / subhead / CTA / fine print)
- Layout vibe (poster, packaging face, app screen, social card)
- Color palette and background
- Language / script if not English
- Aspect or `size` / `quality` if stated
- What must **not** appear (extra slogans, fake logos, wrong brand names)

## Models

1. Call `list_models` and pick a live **image** model available to this key.
2. Filter by id/name heuristics; do not trust category filters alone.
3. **Suggested default after discovery** when present: `byteplus/Dola-Seedream-5.0-flash`.
4. Optionally call `model_pricing`; warn if empty.
5. Stop if no image model is available.

## `generate_image` result shape

JSON-in-text with `data[].url` or `data[].b64_json`. Arguments: `model`, `prompt`, optional `n` / `size` / `quality` / `response_format`. No reference image or font file input.

## Workflow

1. Lock the exact copy with the user before generating.
2. Discover an image model with `list_models` (prefer Seedream flash when listed).
3. Optionally price with `model_pricing`.
4. Call `generate_image` with a prompt that:
   - puts each required string in **quotes** inside the prompt
   - states placement (top headline, centered, packaging panel, etc.)
   - asks for sharp, legible lettering and no extra invented text
5. Parse JSON-in-text; return the image and model id.
6. If lettering is wrong, revise by re-quoting the exact strings and simplifying the layout — do not claim a dedicated typesetting API.

## Related skills

- `logo-concepts` — mark / icon concepts (not true SVG)
- `product-photography` — product stills where text is usually avoided
- `nexllm-mcp` — foundation tools and errors

## Errors

See `nexllm-mcp` for auth vs tool error codes. Scopes: `models:read`, `pricing:read`, `inference:execute`.

## Refuse

- Video, audio, 3D.
- Claiming perfect typography, editable text layers, or PDF/SVG vector output.
- Inventing legal disclaimers or trademarked slogans the user did not provide.
- Hardcoding a model without `list_models`.

## Quality bar

- Every on-image string the user required was included verbatim in the prompt.
- Model id was live for this key.
- Result parsed from JSON (`url` or `b64_json`).
- Extra invented text was explicitly discouraged in the prompt.
