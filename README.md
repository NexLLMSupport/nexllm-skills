# nexllm-skills

Thin **Agent Skills** pack for [NexLLM](https://www.nexllm.ai) — outcome recipes that call the hosted MCP server for **text and image** only.

Skills are Markdown folders (`SKILL.md`). They do not replace the API. Connect MCP first, then drop these folders where your agent loads skills.

| | |
| --- | --- |
| MCP URL | `https://mcp.nexllm.ai/mcp` |
| MCP docs | https://docs.nexllm.ai/tools/mcp |
| Scope | `list_models`, `model_pricing`, `generate_text`, `generate_image` |
| Not included | Video, audio, 3D, ComfyUI, marketplace plugins |

## Skills in this pack

| Folder | Name | Outcome |
| --- | --- | --- |
| `skills/nexllm-mcp/` | `nexllm-mcp` | Foundation: tools, discover → price → generate, refuse video/3D/audio |
| `skills/nexllm-models/` | `nexllm-models` | Discover / compare models and pricing |
| `skills/draft-with-nexllm/` | `draft-with-nexllm` | Draft or rewrite text via `generate_text` |
| `skills/product-hero-image/` | `product-hero-image` | Product / hero still via `generate_image` |

Every generate skill **requires `list_models` before hardcoding an ID**.

## Quick install

1. Connect NexLLM MCP in your agent (OAuth key-paste on `connect.nexllm.ai`). See [MCP integration](https://docs.nexllm.ai/tools/mcp).
2. Copy skill folders into your agent’s skills directory (keep each folder’s `SKILL.md`).

### Claude Code

```bash
# from a clone or unpack of this repo
mkdir -p .claude/skills
cp -R skills/nexllm-mcp skills/nexllm-models skills/draft-with-nexllm skills/product-hero-image .claude/skills/
```

User-scoped (every project):

```bash
mkdir -p ~/.claude/skills
cp -R skills/* ~/.claude/skills/
```

### Cursor

```bash
mkdir -p .cursor/skills
cp -R skills/nexllm-mcp skills/nexllm-models skills/draft-with-nexllm skills/product-hero-image .cursor/skills/
```

Or `~/.cursor/skills/` for all projects.

### Claude (Desktop / claude.ai)

Connect the [NexLLM MCP](https://docs.nexllm.ai/tools/mcp) custom connector, then upload or attach these skill folders under **Customize → Skills** (wording may vary). There is no marketplace listing for this pack yet — use the files from this repo.

More detail: [docs/install.md](docs/install.md).

## Try it

With MCP connected and skills installed:

- “What chat models do I have on NexLLM right now?”
- “Draft a short changelog from these bullets with NexLLM.”
- “Make a clean white-background hero image for this product.”

If you ask for video, voiceover, or 3D, the foundation skill tells the agent to refuse — those tools are not on the MCP server.

## Layout

```text
nexllm-skills/
  README.md
  LICENSE
  docs/install.md
  skills/
    nexllm-mcp/SKILL.md
    nexllm-models/SKILL.md
    draft-with-nexllm/SKILL.md
    product-hero-image/SKILL.md
```


## Naming note

Folder names stay aligned with the Mintlify skills draft (`nexllm-mcp`, `nexllm-models`, `draft-with-nexllm`, `product-hero-image`).

MCP Server’s discovery sketch used shorter aliases (`nexllm-pick-model`, `nexllm-write`, `nexllm-draw`) — same outcomes, different names. **Do not rename** this pack to those aliases for v0.1; if you need mental mapping:

| This pack | Server sketch alias |
| --- | --- |
| `nexllm-models` | ~ `nexllm-pick-model` |
| `draft-with-nexllm` | ~ `nexllm-write` |
| `product-hero-image` | ~ `nexllm-draw` |
| `nexllm-mcp` | foundation (shared) |

## License

MIT — see [LICENSE](LICENSE).
