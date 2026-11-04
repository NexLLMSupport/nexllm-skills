---
name: image-edit-with-nexllm
description: >
  Revise or restyle a still by generating a new image from a before/after
  prompt via NexLLM MCP generate_image. Use for "edit this image with
  NexLLM", "revise the still", "change the background", "image edit prompt",
  or "make another version". Not a mask, inpaint, or upload API.
---

# Image edit with NexLLM

Outcome skill: a **new still** that follows a written before/after brief. The hosted MCP server exposes `list_models`, `model_pricing`, `generate_text`, and `generate_image` only. There is **no** reference-image upload, mask, inpaint, or edit tool. “Edit” here means **revise via a fresh `generate_image` call** whose prompt states what the picture was and what must change.

## Prerequisites

- NexLLM MCP connected (`https://mcp.nexllm.ai/mcp`)
- Wallet balance for `generate_image`
- See `nexllm-mcp` and https://docs.nexllm.ai/integrations/mcp-integration

## Tools

- `list_models` — required before choosing an image model
- `model_pricing` — optional; may return empty for some image models
- `generate_image` — required; this is a **new generation**, not a pixel edit

Do **not** invent `edit_image`, `inpaint`, `upload_image`, or mask tools. Do **not** pass image bytes, image URLs, or masks as `generate_image` arguments.

## Inputs to collect

- A **written description of the current still** (subject, layout, colors, lighting, text). If the user only attaches a file, describe it in words yourself or ask them to. The file is not an API input.
- The **change** in concrete terms (what stays, what changes)
- What must not drift (logo wording, product shape, count of objects)
- Aspect or `size` / `quality` if stated (free-form; model-dependent)
- Confirmation they accept a **new image**, not a masked edit of their pixels

## Models

1. Call `list_models` and pick a live **image** model available to this key.
2. **Do not rely on `category` / `capability` filters** — they may be no-ops. Choose by **id / name heuristics** from the returned list.
3. A catalog string that mentions “edit” does **not** add an edit API. Still call `generate_image` only.
4. **Suggested defaults after discovery**, in order, only when that exact id is in the live list:
   1. `nexllm/gpt-image-2`
   2. `nexllm/gpt-image-2.5-flare`
   3. `nexllm/gpt-image-2.5-sunburst`
5. If none of those ids are listed, use another live image id. `byteplus/Dola-Seedream-5.0-flash` is acceptable when it is actually present.
6. Optionally call `model_pricing`; if empty, warn and continue only with user OK.
7. If no image model is available, say so and stop. Never use mock IDs (`mock-nexllm-*`).

## `generate_image` result shape

MCP **text** content with JSON — not image blocks:

```json
{
  "model": "<id>",
  "created": 0,
  "data": [{ "url": "https://..." }]
}
```

or `data: [{ "b64_json": "..." }]`. Parse JSON; prefer `url`. Arguments: `model`, `prompt`, optional `n` / `size` / `quality` / `response_format`. **No** `image`, `image_url`, `mask`, or edit parameter.

## Workflow

1. Collect the before description and the after change. Say clearly this is a new generation, not a pixel edit of their file.
2. Discover an image model with `list_models` (prefer `nexllm/gpt-image-2`, then the GPT Image 2.5 ids above, when listed).
3. Optionally price with `model_pricing`.
4. Call `generate_image` with one prompt that:
   - states the previous look (“the still shows …”)
   - states the change (“change only …; keep …”)
   - forbids extras they did not ask for (new logos, extra people, watermark)
5. Parse JSON-in-text; return `url` or `b64_json`, the model id, and a one-line reminder that the pixels are newly generated.
6. Further tweaks are another `generate_image` with a tighter before/after prompt. Re-check `list_models` only if the previous model failed or disappeared.

## Related skills

- `gpt-image-25-precision` — detail-first stills on GPT Image 2.5 when listed
- `photoreal-stills` / `product-photography` / `product-hero-image` — first still, then revise here
- `composite-scene` — prompt-only product + subject + backdrop (also no mask)
- `text-in-image` — when the change is exact on-image copy
- `nexllm-mcp` — foundation tools and errors

## Errors

See `nexllm-mcp` (HTTP `401`/`403` vs tool codes such as `INSUFFICIENT_CREDITS`, `GATEWAY_KEY_INVALID`, `GATEWAY_UNAVAILABLE`).

## Refuse

- True pixel edit, inpaint, outpaint, mask, matte, or “upload this photo into the model.” Say those APIs are not on this MCP server, and offer a new still from a written brief only if they want that.
- Video, audio, 3D, or voiceover.
- Inventing tool arguments for reference images.
- Hardcoding a model without `list_models`.

## Quality bar

- The user was told the result is a new generation.
- The prompt states both the previous look and the requested change.
- No image or mask was sent to `generate_image`.
- Model id came from live discovery.
- Result parsed from JSON-in-text.
