---
name: draft-with-nexllm
description: >
  Draft or rewrite text via NexLLM MCP generate_text. Use for "draft with
  NexLLM", "write a changelog", "rewrite this email", "summarize with
  NexLLM", "blog outline", or "generate text on NexLLM".
---

# Draft with NexLLM

Outcome skill: produce or rewrite text through the NexLLM gateway. Text only — not images or video.

## Prerequisites

- NexLLM MCP connected (`https://mcp.nexllm.ai/mcp`)
- Wallet balance for `generate_text`
- See `nexllm-mcp` and https://docs.nexllm.ai/integrations/mcp-integration

## Tools

- `list_models` — required before choosing a chat / text model
- `model_pricing` — optional, when the user cares about cost
- `generate_text` — required to produce the draft

## Inputs to collect

- Goal (draft, rewrite, summarize, outline, translate, etc.)
- Audience and tone
- Length or format constraints (bullets, markdown sections, max words)
- Source material the user pasted (do not invent quotes or facts)
- Any must-include / must-avoid phrases

## Models

1. Call `list_models` and pick a live **chat / text** model available to this key.
2. **Do not rely on category/capability filters** (may be no-ops). Pick by id/name heuristics from the returned list.
3. Prefer a model that matches the task (general chat for drafts; stronger models only if the user asks for quality over cost).
4. Optionally call `model_pricing` on the chosen ID; if empty, warn and continue only with user OK.
5. If no suitable text model is available, say so and stop. Never hardcode mock IDs.

## `generate_text` result shape

- **Single-shot only** — no stream parameter on this MCP tool.
- Returns JSON-in-text (parse the text content), typically including `model`, `text`, `usage`, and `id`.
- Present the `text` field to the user; do not wait for streaming tokens from MCP.

## Workflow

1. Clarify goal, tone, and constraints if missing.
2. Discover a text model with `list_models` — do not hardcode an ID.
3. Optionally price with `model_pricing`.
4. Call `generate_text` with:
   - the exact model `id` from discovery
   - a clear system/user prompt that includes the user’s material and constraints
5. Parse JSON-in-text; return the draft and the model id used.
6. On revision requests, call `generate_text` again with the prior draft and the change request; keep the same model unless it failed or the user asks to switch (then re-run `list_models` if needed).

## Errors

See `nexllm-mcp`: HTTP auth (`401`/`403`) vs tool codes (`INSUFFICIENT_CREDITS`, `GATEWAY_KEY_INVALID`, etc.). Needs scopes `models:read`, `pricing:read`, `inference:execute`.

## Refuse

- Image generation — point the user at `product-hero-image`.
- Video, audio, music, or 3D — those MCP tools are not available.
- Presenting memorized model IDs as if they were verified for this key.
- Claiming streaming output from MCP `generate_text`.

## Quality bar

- Output matches the requested format.
- Facts stay within user-provided source material unless the user asked for open-ended creative writing.
- Model id came from `list_models` for this session’s key.
- Result parsed from JSON-in-text, not assumed as raw streamed tokens.
