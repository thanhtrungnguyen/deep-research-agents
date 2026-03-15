# Technology Stack

**Project:** deep-research-agents
**Researched:** 2026-03-15
**Research Mode:** Ecosystem — Claude Code plugin with multi-agent research capabilities

---

## Recommended Stack

### Plugin Architecture Layer

| Technology | Version | Purpose | Why |
|------------|---------|---------|-----|
| Claude Code Plugin System | 2.x | Core runtime | The only supported distribution format; provides skills, agents, hooks, MCP integration natively |
| `.claude-plugin/plugin.json` | schema v2 | Plugin manifest | Required metadata: name, version, description, author, component paths |
| `agents/` directory | — | Subagent definitions (`.md` with YAML frontmatter) | Each research role (Lit Analyst, Experiment Designer, etc.) maps to one agent file |
| `skills/` directory | — | User- and model-invocable skills (`SKILL.md` format) | GSD pipeline stages map to skills; `context: fork` for isolated execution |
| `hooks/hooks.json` | — | Event-driven automation | PostToolUse for artifact formatting; Stop hooks for output validation |
| `.mcp.json` | — | MCP server declarations | Bundles Semantic Scholar and web search servers with the plugin |
| `CLAUDE.md` | — | Persistent project context | Domain knowledge, research conventions, benchmark reference data |
| `.claude/rules/` | — | Modular rule files scoped by path | Separate rules for each pipeline phase; path-scoped to keep context lean |

**Confidence: HIGH** — sourced directly from official Claude Code documentation at code.claude.com/docs.

---

### MCP Servers for Research Capabilities

| MCP Server | Source | Purpose | When to Use |
|------------|--------|---------|-------------|
| Semantic Scholar MCP | `akapet00/semantic-scholar-mcp` (pip/uv) | Search papers, get citations, retrieve author details | Literature discovery and citation network traversal |
| Brave Search MCP | `@modelcontextprotocol/server-brave-search` (npx) | Web search for recent blog posts, benchmarks, preprints | Supplementing arXiv when papers aren't indexed yet |
| arXiv Python wrapper | `arxiv` PyPI package (v2.x, Mar 2026 release) | Direct arXiv API access from hook scripts | Fetching paper abstracts in batch for seeding domain knowledge |

**Confidence: HIGH for Brave Search and arXiv (official/PyPI). MEDIUM for specific Semantic Scholar MCP implementation** — multiple community implementations exist (akapet00, FujishigeTemma, JackKuo666, zongmin-yu); all provide equivalent search/citation tools but akapet00 has confirmed Claude Code integration demo.

**Why Brave Search over Google CSE:** Google CSE is sunetting January 2027; Brave has no discontinuation concerns, has a free tier, and is now the recommended alternative per the Claude Code MCP ecosystem.

**Why Semantic Scholar over raw arXiv API:** Semantic Scholar provides citation counts, author networks, and field-of-study metadata that arXiv does not expose, enabling gap analysis and influence scoring critical for research topic selection.

---

### Python Tooling (Hook Scripts and Utilities)

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| `arxiv` | 2.x (PyPI, Mar 2026) | arXiv API wrapper | Batch paper retrieval in hook scripts or standalone utilities |
| `semanticscholar` | latest (PyPI) | Unofficial Semantic Scholar Python client | Programmatic paper lookup by DOI, arXiv ID, or S2 Paper ID |
| `jq` | system binary | JSON parsing in bash hook scripts | Extracting fields from hook stdin JSON in shell hooks |
| `uv` | latest | Python package runner for MCP servers | Running MCP servers with `uv run --with` without pre-installing globally |

**Confidence: HIGH** — all are well-established packages with active 2025-2026 release history.

**Note on Python vs Node:** Hook scripts are shell scripts that call Python via `python3` or `uv run`. No standalone Python service is needed — the plugin is pure Claude Code primitives (agents, skills, hooks, MCP). Python is only invoked in hook scripts for artifact processing.

---

### Subagent Model Selection

| Agent Role | Recommended Model | Rationale |
|------------|------------------|-----------|
| Literature Analyst | `inherit` (Sonnet default) | Requires deep reasoning about paper content and gap identification |
| Experiment Designer | `sonnet` (explicit) | Complex multi-step experimental logic; needs full capability |
| System Architect | `sonnet` (explicit) | Architecture decisions require careful reasoning |
| Writing Advisor | `haiku` | Paper section critique is lower-stakes and faster with Haiku |
| Orchestrator / main thread | `inherit` | Uses whatever model the user invoked Claude Code with |

**Confidence: HIGH** — model alias options (`sonnet`, `opus`, `haiku`, `inherit`) verified in official subagent documentation.

---

## What NOT to Use (Anti-Recommendations)

| Rejected Option | Why Not | What to Use Instead |
|-----------------|---------|---------------------|
| **LangChain / LangGraph** | External Python orchestration framework adds complexity with no benefit inside Claude Code; duplicates what the native agent system provides | Claude Code native agents + skills |
| **AutoGen / CrewAI** | Same problem — external multi-agent framework that conflicts with Claude Code's agent model | Claude Code native agents |
| **Standalone FastAPI/Flask service** | Adds a server process to manage; skills and MCP servers handle all inter-component communication natively | MCP servers for external tools; native skills for internal logic |
| **`.claude/commands/` (old format)** | Commands are legacy; skills merged with commands in Jan 2026. New development should use `skills/` with `SKILL.md` | `skills/<name>/SKILL.md` |
| **`plugin.json` inside `skills/` or `agents/`** | Common structural mistake — only `plugin.json` lives in `.claude-plugin/`; all other directories are at plugin root | Correct directory structure (see below) |
| **General-purpose web scraping (Playwright/Puppeteer)** | Out of scope — paper access via Semantic Scholar/arXiv API is sufficient and doesn't require browser automation | Semantic Scholar MCP + arXiv Python client |
| **Google CSE MCP** | Sunsetting January 2027 | Brave Search MCP |
| **Agent Teams (real-time peer communication)** | Overkill for this use case; subagents returning summaries to main thread is sufficient for sequential pipeline stages | Subagent chaining via main conversation |

---

## Canonical Plugin Directory Structure

```
deep-research-agents/
├── .claude-plugin/
│   └── plugin.json                 # Manifest only — nothing else here
├── agents/
│   ├── literature-analyst.md       # Literature review and gap identification
│   ├── experiment-designer.md      # Benchmark design, methodology
│   ├── system-architect.md         # Implementation guidance
│   └── writing-advisor.md          # Paper structure, section drafting
├── skills/
│   ├── select-topic/
│   │   └── SKILL.md                # GSD Phase 1: topic shortlist generation
│   ├── literature-review/
│   │   └── SKILL.md                # GSD Phase 2: systematic literature analysis
│   ├── design-experiment/
│   │   └── SKILL.md                # GSD Phase 3: experiment plan generation
│   ├── plan-implementation/
│   │   └── SKILL.md                # GSD Phase 4: prototype architecture
│   └── draft-paper/
│       └── SKILL.md                # GSD Phase 5: paper section drafting
├── hooks/
│   └── hooks.json                  # PostToolUse for artifact formatting
├── .mcp.json                       # Semantic Scholar + Brave Search MCP declarations
├── CLAUDE.md                       # Domain knowledge: surveys, benchmarks, open problems
├── .claude/
│   └── rules/
│       ├── research-conventions.md # Research methodology rules (always loaded)
│       ├── artifact-format.md      # Output file formats (always loaded)
│       └── benchmark-context.md   # SWE-bench, HumanEval refs (always loaded)
├── scripts/
│   └── format-artifact.sh          # Hook script for artifact output formatting
└── settings.json                   # (optional) default agent activation
```

---

## Installation

```bash
# Test the plugin locally during development
claude --plugin-dir ./deep-research-agents

# Reload after changes (without restart)
# In Claude Code session:
/reload-plugins

# Verify agents are registered
/agents

# Invoke a skill manually
/deep-research-agents:select-topic
```

**MCP server setup (add to `.mcp.json` in plugin root):**

```json
{
  "mcpServers": {
    "semantic-scholar": {
      "command": "uv",
      "args": ["run", "--with", "git+https://github.com/akapet00/semantic-scholar-mcp", "semantic-scholar-mcp"],
      "env": {
        "SEMANTIC_SCHOLAR_API_KEY": "${SEMANTIC_SCHOLAR_API_KEY}"
      }
    },
    "brave-search": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-brave-search"],
      "env": {
        "BRAVE_API_KEY": "${BRAVE_API_KEY}"
      }
    }
  }
}
```

**Environment variables required:**
- `SEMANTIC_SCHOLAR_API_KEY` — from semanticscholar.org/product/api (optional for low-volume, required for production rates)
- `BRAVE_API_KEY` — from api.search.brave.com (free tier available)

---

## Key Plugin System Facts (Verified Against Official Docs)

**Version requirement:** Claude Code 1.0.33+ for plugin system. Current releases are 2.x.

**Skills vs Commands (Jan 2026 merge):** A file at `.claude/commands/deploy.md` and `skills/deploy/SKILL.md` are now equivalent; `user-invocable: true` is what makes a skill appear in the `/` menu. Use `skills/` for new development.

**Subagents cannot spawn subagents.** Nesting is not supported — the 4 research agents must be invoked from the main conversation thread, not from each other. Chain them sequentially: main → Lit Analyst → main → Experiment Designer → etc.

**`disable-model-invocation: true`** on pipeline skills (like `draft-paper`) prevents Claude from invoking them spontaneously during non-research conversations. Use this for all GSD-stage skills; they should only fire when the user explicitly invokes them.

**`memory: project` on subagents** enables cross-session learning stored in `.claude/agent-memory/<agent-name>/`. Each research agent should use this to accumulate domain knowledge across conversations — highly relevant for building up literature survey state over multiple sessions.

**CLAUDE.md token budget:** Keep under 200 lines for reliable adherence. Split benchmark reference data, survey summaries, and research conventions into `.claude/rules/` files using path-scoped rules where possible.

**Hook input format:** All hook scripts receive JSON on stdin. Always use `jq -r '.tool_input.field'` to extract values — never assume positional arguments.

**`${CLAUDE_PLUGIN_ROOT}`** must be used in all hook commands and MCP server paths. Absolute paths break after marketplace installation due to plugin caching.

---

## Sources

- [Claude Code Plugin Creation Docs](https://code.claude.com/docs/en/plugins) — HIGH confidence, official
- [Claude Code Plugins Reference](https://code.claude.com/docs/en/plugins-reference) — HIGH confidence, official
- [Claude Code Subagents Documentation](https://code.claude.com/docs/en/sub-agents) — HIGH confidence, official
- [Claude Code Skills Documentation](https://code.claude.com/docs/en/skills) — HIGH confidence, official
- [Claude Code Hooks Specification](https://code.claude.com/docs/en/hooks) — HIGH confidence, official
- [Claude Code Memory / CLAUDE.md](https://code.claude.com/docs/en/memory) — HIGH confidence, official
- [Semantic Scholar Academic Graph API](https://www.semanticscholar.org/product/api) — HIGH confidence, official
- [semanticscholar PyPI package](https://pypi.org/project/semanticscholar/) — HIGH confidence, PyPI
- [arxiv PyPI package](https://pypi.org/project/arxiv/) — HIGH confidence, PyPI (v2.x, Mar 2026)
- [akapet00/semantic-scholar-mcp](https://github.com/akapet00/semantic-scholar-mcp) — MEDIUM confidence, community (confirmed Claude Code integration)
- [Brave Search MCP](https://claudelog.com/claude-code-mcps/brave-mcp/) — MEDIUM confidence, community guides
- [Claude Code MCP Docs](https://code.claude.com/docs/en/mcp) — HIGH confidence, official
- [Multi-agent orchestration patterns](https://claudefa.st/blog/guide/agents/sub-agent-best-practices) — MEDIUM confidence, community
- [wshobson/agents — multi-agent orchestration](https://github.com/wshobson/agents) — MEDIUM confidence, reference implementation
- [Deep-Research-skills plugin](https://github.com/Weizhena/Deep-Research-skills) — MEDIUM confidence, community reference
