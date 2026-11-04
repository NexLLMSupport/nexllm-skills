# nexllm-skills

Thin **Agent Skills** pack for [NexLLM](https://www.nexllm.ai) — outcome recipes that call the hosted MCP server for **text and image** only.

Skills are Markdown folders (`SKILL.md`). They do not replace the API. Connect MCP first, then drop these folders where your agent loads skills.

**Discovery paths:** Cloud Agents / project checkouts load skills from `.cursor/skills/` (and `.agents/skills/`). The Cursor plugin pack continues to use `skills/` plus `.cursor-plugin/plugin.json`. Keep both trees in sync for this pack.

| | |
| --- | --- |
| MCP URL | `https://mcp.nexllm.ai/mcp` |
| MCP docs | https://docs.nexllm.ai/integrations/mcp-integration |
| Scope | `list_models`, `model_pricing`, `generate_text`, `generate_image` |
| Version | **v0.1.4** (this tag) |
| Not included | Video, audio, 3D, upscale/edit APIs, reference-image upload, mask/inpaint, LoRA training, ComfyUI, marketplace plugins |

## Skills in this pack

| Folder | Name | Outcome |
| --- | --- | --- |
| `skills/nexllm-mcp/` | `nexllm-mcp` | Foundation: tools, discover → price → generate, refuse video/3D/audio |
| `skills/nexllm-models/` | `nexllm-models` | Discover / compare models and pricing |
| `skills/draft-with-nexllm/` | `draft-with-nexllm` | Draft or rewrite text via `generate_text` |
| `skills/product-hero-image/` | `product-hero-image` | Product / hero still via `generate_image` |
| `skills/product-photography/` | `product-photography` | Studio, lifestyle, and hero product stills |
| `skills/photoreal-stills/` | `photoreal-stills` | Realistic, camera-like still imagery |
| `skills/text-in-image/` | `text-in-image` | Posters / packaging / UI with exact on-image copy |
| `skills/logo-concepts/` | `logo-concepts` | Logo / icon **concept rasters** (not true SVG) |
| `skills/game-assets-2d/` | `game-assets-2d` | Sprites, stickers, and 2D asset stills |
| `skills/composite-scene/` | `composite-scene` | Prompt-only product + subject + backdrop (no mask API) |
| `skills/image-edit-with-nexllm/` | `image-edit-with-nexllm` | Prompt-only revise: a new `generate_image` with before/after instructions (no mask or upload) |
| `skills/gpt-image-25-precision/` | `gpt-image-25-precision` | Detail-heavy stills; prefer GPT Image 2.5 flare/sunburst after discovery |
| `skills/nano-banana-design/` | `nano-banana-design` | Design / concept stills; prefer Gemini 3 Pro image after discovery |
| `skills/fast-stickers-gemini/` | `fast-stickers-gemini` | Fast sticker / icon / emoji stills; prefer Gemini flash image models |
| `skills/seedream-pro-layouts/` | `seedream-pro-layouts` | Layout / multi-element / poster-like stills; prefer Seedream 5.0 pro, then lite, then flash |

Every generate skill **requires `list_models` before hardcoding an ID**. A suggested default is used only when that exact id is in the live list for the connected key. Image skills from `product-hero-image` through `composite-scene` still suggest `byteplus/Dola-Seedream-5.0-flash` when present. The last five rows name their own preferred ids inside each `SKILL.md` and fall back only to another id `list_models` actually returned.

Tag **v0.1.4** does not include those five skills. Plugin version stays **0.1.4**. Do not treat this tree as a `v0.1.5` release.

## Quick install

1. Connect NexLLM MCP in your agent (OAuth key-paste on `connect.nexllm.ai`). See [MCP integration](https://docs.nexllm.ai/integrations/mcp-integration).
2. Clone this pack at tag **`v0.1.4`**, then copy skill folders into your agent’s skills directory (keep each folder’s `SKILL.md`).

```bash
git clone https://github.com/NexLLMSupport/nexllm-skills.git
cd nexllm-skills
git checkout v0.1.4
```

To smoke the five skills that are **not** in tag `v0.1.4`, stay on the default branch instead of checking out the tag, then copy those folders with the rest of `skills/`:

```bash
mkdir -p ~/.cursor/skills
cp -R skills/image-edit-with-nexllm skills/gpt-image-25-precision \
  skills/nano-banana-design skills/fast-stickers-gemini \
  skills/seedream-pro-layouts ~/.cursor/skills/
```

### Claude Code

```bash
# from the clone above
mkdir -p .claude/skills
cp -R skills/nexllm-mcp skills/nexllm-models skills/draft-with-nexllm \
  skills/product-hero-image skills/product-photography skills/photoreal-stills \
  skills/text-in-image skills/logo-concepts skills/game-assets-2d \
  skills/composite-scene .claude/skills/
```

User-scoped (every project):

```bash
mkdir -p ~/.claude/skills
cp -R skills/* ~/.claude/skills/
```

## Install in Cursor

Still requires NexLLM MCP connected at `https://mcp.nexllm.ai/mcp` (see [MCP integration](https://docs.nexllm.ai/integrations/mcp-integration)).

**Preferred (Agents / web):** Cursor **Customize** → import plugin/skills **From GitHub**:

`https://github.com/NexLLMSupport/nexllm-skills`

**Desktop optional:** copy each folder under `skills/` into `~/.cursor/skills/<name>/`, then **Settings → Agents → Sync Skills for Cloud Agents**. For Cloud Agents that clone this repo, skills under `.cursor/skills/` are discovered from the project tree.

```bash
mkdir -p ~/.cursor/skills
cp -R skills/nexllm-mcp skills/nexllm-models skills/draft-with-nexllm \
  skills/product-hero-image skills/product-photography skills/photoreal-stills \
  skills/text-in-image skills/logo-concepts skills/game-assets-2d \
  skills/composite-scene ~/.cursor/skills/
```

### Claude (Desktop / claude.ai)

Connect the [NexLLM MCP](https://docs.nexllm.ai/integrations/mcp-integration) custom connector, then upload or attach these skill folders under **Customize → Skills** (wording may vary). There is no marketplace listing for this pack yet — use the files from this repo.

More detail: [docs/install.md](docs/install.md).

## Try it

With MCP connected and skills installed:

- “What chat models do I have on NexLLM right now?”
- “Draft a short changelog from these bullets with NexLLM.”
- “Make a clean white-background hero image for this product.”
- “Studio packshot and a lifestyle shot of this bottle.”
- “Poster with the exact headline: Launch Week.”
- “Logo concept marks for a coffee brand (raster is fine).”
- “Revise that still: same bottle, warmer light, no extra text.” (new generation, not a mask edit)
- “High-detail still of the clasp and stitching.”
- “Design concept still for a calm skincare brand.”
- “Fast emoji-style sticker of a cat.”
- “Poster-like layout: headline band, three products, quiet footer.”

If you ask for video, voiceover, 3D, upscale pipelines, mask/inpaint, reference-image upload, or LoRA training, the skills tell the agent to refuse — those tools are not on the MCP server.

## Layout

```text
nexllm-skills/
  README.md
  LICENSE
  .cursor-plugin/plugin.json
  docs/install.md
  skills/                    # plugin pack (marketplace / From GitHub plugin)
    nexllm-mcp/SKILL.md
    nexllm-models/SKILL.md
    draft-with-nexllm/SKILL.md
    product-hero-image/SKILL.md
    product-photography/SKILL.md
    photoreal-stills/SKILL.md
    text-in-image/SKILL.md
    logo-concepts/SKILL.md
    game-assets-2d/SKILL.md
    composite-scene/SKILL.md
    image-edit-with-nexllm/SKILL.md
    gpt-image-25-precision/SKILL.md
    nano-banana-design/SKILL.md
    fast-stickers-gemini/SKILL.md
    seedream-pro-layouts/SKILL.md
  .cursor/skills/            # Cloud Agents / project discovery
    (same folders as skills/)
```

## Naming note

Folder names stay aligned with the Mintlify skills docs (`nexllm-mcp`, `nexllm-models`, `draft-with-nexllm`, `product-hero-image`, plus the v0.1.4 image outcomes and the later image outcomes on this tree: `image-edit-with-nexllm`, `gpt-image-25-precision`, `nano-banana-design`, `fast-stickers-gemini`, `seedream-pro-layouts`).

MCP Server’s discovery sketch used shorter aliases (`nexllm-pick-model`, `nexllm-write`, `nexllm-draw`) — same outcomes, different names. **Do not rename** this pack to those aliases for v0.1; if you need mental mapping:

| This pack | Server sketch alias |
| --- | --- |
| `nexllm-models` | ~ `nexllm-pick-model` |
| `draft-with-nexllm` | ~ `nexllm-write` |
| `product-hero-image` | ~ `nexllm-draw` |
| `nexllm-mcp` | foundation (shared) |

## License

MIT — see [LICENSE](LICENSE).
