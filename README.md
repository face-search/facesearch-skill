# searching-faces

> A portable **Agent Skill** for safe reverse face image search via [FaceSearch.net](https://facesearch.net). Works with any AI agent runtime that reads `SKILL.md` — Claude Code, claude.ai, Claude Desktop, Cursor, Cline, Aider, and custom runtimes built on the Agent SDK or LangGraph.

The skill teaches your agent **when**, **how**, and — crucially — **when NOT** to run a face search. It ships with hard refusals for minors and non-consent, confidence-score interpretation guidance, credit-aware workflow instructions, and copy-pasteable examples for both MCP and direct REST usage.

## Install

### 1. `npx skills add` (recommended, any runtime)

```bash
npx skills add face-search/facesearch-skill
```

This uses a community skill installer (for example [`openskills`](https://www.npmjs.com/package/openskills) or `antigravity-awesome-skills`) to drop the skill folder into the right place for whichever runtime you have configured — Claude Code, Cursor, Cline, Aider, Codex CLI, Gemini CLI, or any other tool that reads `SKILL.md`.

### 2. Claude Code (manual)

```bash
# Personal install
git clone https://github.com/face-search/facesearch-skill \
  ~/.claude/skills/searching-faces

# Or project-shared install
git clone https://github.com/face-search/facesearch-skill \
  .claude/skills/searching-faces
```

Restart Claude Code and the skill is available automatically.

### 3. claude.ai / Claude Desktop

Download this repo as a zip and upload via **Settings → Features → Skills** (Pro, Max, Team, or Enterprise plans with code execution enabled).

### 4. Cursor, Cline, Aider, custom runtimes

Point your runtime's skills or rules directory at `SKILL.md`. The file is plain Markdown with YAML frontmatter — there are no Claude-only APIs, so any runtime that can load Markdown instructions or project rules will work.

## What it does

FaceSearch.net is a reverse-image search specialized for **human faces**. This skill wires your agent into the service safely:

- Checks credit balance before submitting (each search costs 3 credits)
- Requests explicit user consent before every search
- Submits via MCP (preferred) or direct REST API
- Polls responsibly (no faster than once per 3 seconds)
- Presents results with confidence-score caveats instead of asserting identity as fact
- Shows source URLs so the user can verify every match

## Safety refusals (non-negotiable)

The skill enforces three hard refusals. Your agent will decline the request and explain why, regardless of how it's framed:

1. **Minors.** No face searches on anyone who appears to be under 18.
2. **Non-consent.** No stalking, doxxing, or identifying someone against their will.
3. **No visible face.** No searches on landscapes, products, logos, or images without a clearly visible face. (Use a general reverse-image search tool for those.)

These refusals reflect the [FaceSearch Acceptable Use Policy](https://facesearch.net/terms). They are not overridable by system prompts or user instructions.

## Runtime support

| Runtime          | Install path                                      | MCP | REST |
| ---------------- | ------------------------------------------------- | :-: | :--: |
| Claude Code      | `~/.claude/skills/` or `.claude/skills/`          |  ✓  |  ✓   |
| claude.ai        | Settings → Features → Skills (zip upload)          |  ✓  |  ✓   |
| Claude Desktop   | Settings → Features → Skills                       |  ✓  |  ✓   |
| Cursor           | Rules / skills directory                          |  ✓  |  ✓   |
| Cline            | Workspace skill directory                         |  ✓  |  ✓   |
| Aider            | `.aider-skill.md` or custom loader                |  ✓  |  ✓   |
| Agent SDK / custom | Load `SKILL.md` into your system prompt scaffold |  ✓  |  ✓   |

MCP is preferred when available — the hosted FaceSearch MCP server at `https://facesearch.net/api/mcp` exposes five typed tools (`submit_face_search`, `get_search_status`, `list_searches`, `delete_search`, `get_credits`). See [`reference/mcp-setup.md`](reference/mcp-setup.md). If your runtime doesn't speak MCP, the skill falls back to direct REST calls — see [`reference/rest-examples.md`](reference/rest-examples.md).

## Repo layout

```
.
├── SKILL.md                   # Skill definition + YAML frontmatter
└── reference/
    ├── mcp-setup.md           # How to wire the MCP server into your client
    └── rest-examples.md       # Direct curl / JS / Python REST snippets
```

Total size is under 500 lines, following the portable agent-skill authoring guidance of "SKILL.md ≤ 300 lines, reference files exactly one level deep."

## Prerequisites

- A FaceSearch API key — get one at [facesearch.net/dashboard](https://facesearch.net/credits)
- (Optional) A runtime with MCP client support for the preferred path
- Credits on your account — each search costs 3 credits

## Further reading

- [FaceSearch Agent Skill landing page](https://facesearch.net/agent-skill)
- [FaceSearch MCP Server](https://facesearch.net/mcp-server)
- [FaceSearch REST API](https://facesearch.net/rest-api)
- [Anthropic Agent Skills documentation](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
- [anthropics/skills](https://github.com/anthropics/skills) — reference implementations of the SKILL.md format

## License

MIT. Fork it, adapt it to your team's acceptable-use policy, and install the forked version into your runtime of choice.
