---
name: photoreal-stills
description: >
  Produce realistic, non-AI-looking still imagery via NexLLM MCP
  generate_image. Use for "photoreal", "realistic photo", "natural looking
  image", "documentary still", "camera-like photo", or "no AI look".
---

# Photoreal stills

Outcome skill: still images that read as real photographs — natural light, believable materials, restrained processing. Text and image only — no video, no reference-image upload.

## Prerequisites

- NexLLM MCP connected (`https://mcp.nexllm.ai/mcp`)
- Wallet balance for `generate_image`
- See `nexllm-mcp` and https://docs.nexllm.ai/integrations/mcp-integration

## Tools

- `list_models` — required before choosing an image model
- `model_pricing` — optional; may return empty for some image models
- `generate_image` — required to produce the still

## Inputs to collect

- Subject and setting (who/what, where, time of day)
- Camera feel if useful (lens length vibe, depth of field, handheld vs tripod) — describe in prompt words only
- Mood / color (neutral, warm daylight, overcast, night)
- What to avoid (plastic skin, warped hands, fake bokeh, illegible text, logos)
- Aspect or `size` / `quality` preference if stated (free-form; model-dependent)
- Any factual details the user supplied (do not invent real people or trademarks)

## Models

1. Call `list_models` and pick a live **image** model available to this key.
2. **Do not rely on `category` / `capability` filters** — they may be no-ops. Choose by **id / name heuristics** from the returned list.
3. **Suggested default after discovery** when present: `byteplus/Dola-Seedream-5.0-flash`. Always verify it appears for this key.
4. Optionally call `model_pricing`; if empty, warn and continue only with user OK.
5. If no image model is available, say so and stop.

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

1. Collect subject, setting, and realism constraints.
2. Discover an image model with `list_models` (prefer Seedream flash when listed).
3. Optionally price with `model_pricing`.
4. Call `generate_image` with a prompt that emphasizes:
   - natural lighting and materials
   - correct anatomy / geometry where relevant
   - “photograph” / documentary cues rather than illustration or CGI
   - explicit bans on watermarks, fake UI chrome, and extra logos unless requested
5. Parse JSON-in-text; return `url` or `b64_json` plus model id.
6. Offer one revision pass with tightened realism constraints if needed.

## Related skills

- `product-photography` / `product-hero-image` — product / ecommerce framing
- `text-in-image` — when legible on-image copy is the goal instead
- `nexllm-mcp` — foundation tools and errors

## Errors

See `nexllm-mcp` (HTTP `401`/`403` vs tool codes such as `INSUFFICIENT_CREDITS`, `GATEWAY_KEY_INVALID`, `GATEWAY_UNAVAILABLE`).

## Refuse

- Video, audio, 3D, or voiceover.
- Claiming photoreal output of a real identifiable private person from a photo upload (no image input exists).
- Hardcoding a model without `list_models`.

## Quality bar

- Prompt pushes camera-like realism, not stylized art, unless the user asked otherwise.
- Model id came from live discovery.
- Result parsed from JSON-in-text.
