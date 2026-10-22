---
name: nexllm-models
description: >
  Discover and compare NexLLM models by capability or price. Use for
  "what models do I have", "list models on NexLLM", "cheapest chat model",
  "image models available", "price this model", or "model_pricing".
---

# NexLLM models

Discover live model IDs and pricing for the connected NexLLM key. Do not generate text or images unless the user explicitly asks after the comparison.

## Prerequisites

NexLLM MCP connected (`https://mcp.nexllm.ai/mcp`). See skill `nexllm-mcp` and https://docs.nexllm.ai/integrations/mcp-integration.

## Tools

- `list_models` — required first
- `model_pricing` — for shortlisted IDs when cost matters

Do not call `generate_text` or `generate_image` in this skill unless the user clearly asks to run a generation after seeing options.

## Inputs to collect

- Capability interest: chat / text, image, or “anything”
- Budget preference (cheapest, balanced, quality) if stated
- Any hard constraints (must support vision, long context, etc.) if stated

## Workflow

1. Call `list_models`. **Do not rely on `category` / `capability` filters** — they may be no-ops. List everything returned and filter in the answer by **id / name heuristics** (chat vs image-looking ids).
2. Never hardcode a catalog. Model IDs differ by channel group and key restrictions. Never use mock IDs (`mock-nexllm-*`).
3. Shortlist 2–5 IDs that match the user’s need.
4. For those IDs, call `model_pricing` when the user asked about cost or “cheapest”.
5. Pricing shape when present: `{object:"list", data:[{model, currency, input_price_per_million, output_price_per_million}, ...]}`. Some image / unpriceable rows return **empty `data`** — say “no quote from `model_pricing`” rather than inventing a price; generation can still proceed later with a user warning.
6. Summarize side by side: id, likely use (text vs image from heuristics), and price notes from `model_pricing` (or “unquoted”).
7. Stop. Offer to draft text (`draft-with-nexllm`) or make an image (`product-hero-image`, `product-photography`, `photoreal-stills`, `text-in-image`, `logo-concepts`, `game-assets-2d`, `composite-scene`) only if the user wants a next step.

## Errors

Auth failures (`401` / `403` scope/connection) vs tool codes — see `nexllm-mcp`. Scopes: `models:read`, `pricing:read` (and `inference:execute` only if the user then asks to generate).

## Refuse

- Video, audio, music, or 3D model “skills” — say MCP does not expose those tools yet.
- Inventing IDs that did not appear in `list_models`.
- Inventing prices when `model_pricing` returned empty.

## Quality bar

- Every ID in the answer came from `list_models` (or the user supplied it and you verified with discovery/pricing).
- Pricing claims come from `model_pricing`, not guesses; empty list is reported honestly.
