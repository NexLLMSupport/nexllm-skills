# Install nexllm-skills

Copy these skill folders into wherever your agent already loads Agent Skills. Connect [NexLLM MCP](https://docs.nexllm.ai/integrations/mcp-integration) first (`https://mcp.nexllm.ai/mcp`); skills without MCP can only describe steps, not run them.

## Get the pack

```bash
git clone https://github.com/NexLLMSupport/nexllm-skills.git
cd nexllm-skills
git checkout v0.1.3
```

Then use the paths below (`/path/to/nexllm-skills` = this clone).

## Before you start

1. NexLLM account + API key at [www.nexllm.ai](https://www.nexllm.ai)
2. MCP connected with OAuth key-paste on `connect.nexllm.ai`
3. Confirm a trivial ask works (for example, “list a few models on NexLLM” → agent calls `list_models`)
4. Wallet balance if you will run `generate_text` or `generate_image`

## Claude Code

**Project-scoped**

```bash
cd /path/to/your/project
mkdir -p .claude/skills
cp -R /path/to/nexllm-skills/skills/nexllm-mcp \
      /path/to/nexllm-skills/skills/nexllm-models \
      /path/to/nexllm-skills/skills/draft-with-nexllm \
      /path/to/nexllm-skills/skills/product-hero-image \
      .claude/skills/
```

**User-scoped**

```bash
mkdir -p ~/.claude/skills
cp -R /path/to/nexllm-skills/skills/* ~/.claude/skills/
```

Start a session and ask an outcome in plain language. Matchers use each skill’s `name` + `description` frontmatter.

## Cursor

**Project-scoped**

```bash
mkdir -p .cursor/skills
cp -R /path/to/nexllm-skills/skills/* .cursor/skills/
```

**User-scoped**

```bash
mkdir -p ~/.cursor/skills
cp -R /path/to/nexllm-skills/skills/* ~/.cursor/skills/
```

Ask in chat, or use Cursor’s skill picker (`/` / Skills UI, depending on build) and select by skill name.

Ensure MCP is configured to `https://mcp.nexllm.ai/mcp` ([MCP docs](https://docs.nexllm.ai/integrations/mcp-integration)).

## Claude (Desktop / claude.ai)

1. Add the NexLLM custom connector — see [MCP integration](https://docs.nexllm.ai/integrations/mcp-integration) (Claude tab).
2. Open **Customize → Skills** (label may vary by client version).
3. Upload or attach each folder under `skills/` (each must contain `SKILL.md`).
4. Ask for the outcome in chat.

There is no official Claude marketplace /plugin entry for this pack yet. Use the files from this repository.

## Other Agent Skills clients

Many clients look under `.agents/skills/` or `~/.agents/skills/`. Confirm your agent’s docs, then:

```bash
mkdir -p .agents/skills
cp -R /path/to/nexllm-skills/skills/* .agents/skills/
```

Pair recipes with MCP tools — skills alone do not talk to NexLLM.

## Verify

| Ask | Expected skill | Expected tools |
| --- | --- | --- |
| “Cheapest chat models on NexLLM?” | `nexllm-models` | `list_models`, maybe `model_pricing` |
| “Draft a polite reply from these notes on NexLLM” | `draft-with-nexllm` | `list_models` → `generate_text` |
| “White-background packshot of this bottle” | `product-hero-image` | `list_models` → `generate_image` |
| “Make a vertical launch video with VO” | `nexllm-mcp` (refuse) | no fake tool calls |

## Updating

Replace the skill folders with a newer copy of this pack. After MCP reconnect or key rotation, re-test `list_models` once.

## Related

- Pack overview: [../README.md](../README.md)
- MCP connect: https://docs.nexllm.ai/integrations/mcp-integration
