# Project Research Summary

**Project:** deep-research-agents
**Domain:** Claude Code plugin — multi-agent academic research assistant for agentic code generation
**Researched:** 2026-03-15
**Confidence:** HIGH

## Executive Summary

The deep-research-agents project is a Claude Code plugin that implements a multi-agent research pipeline for an MSE capstone focused on LLM-based agentic code generation. The consensus from all four research dimensions is clear: build this as a plugin-first, skill-orchestrated, subagent-executed system using Claude Code's native primitives (skills, agents, hooks, MCP servers). No external orchestration frameworks are warranted — LangChain, AutoGen, and CrewAI all conflict with or duplicate what the native Claude Code plugin system already provides at this scale. The recommended architecture uses five phase skills (topic, lit-review, experiment, implement, paper), four specialist subagents (Literature Analyst, Experiment Designer, System Architect, Writing Advisor), and a filesystem-based artifact store (`.research/`) as the single source of truth for pipeline state.

The recommended approach prioritizes incremental delivery over architectural completeness. Research strongly advises building and validating one skill end-to-end before expanding. The most urgent feature is topic shortlist generation with feasibility scoring — the capstone timeline constraint ("lock topic in weeks") makes this the first thing that must exist and work. MCP integration (Semantic Scholar + Brave Search) is the critical infrastructure dependency that feeds the entire literature discovery pipeline. Citation hallucination is the highest-consequence technical risk: 18-55% hallucination rates under controlled conditions, and NeurIPS 2025 accepted papers containing verified AI-hallucinated references establishes the real-world severity.

The primary architectural risks are the "Bag-of-Agents" failure (four disconnected skills with no orchestration layer causing artifact divergence) and agent context rot across long research sessions (pipeline state lost as conversation compacts). Both are preventable through disciplined design: a `research_state.md` file loaded at every agent invocation, `context: fork` isolation per phase, and explicit artifact handoff chains rather than routing content through conversation history. SWE-bench Verified's saturation (~77% solved, 10.6% data leakage) must be embedded as domain knowledge in the Experiment Designer agent to prevent the capstone from targeting a benchmark where a marginal improvement carries no academic weight.

---

## Key Findings

### Recommended Stack

The entire plugin is built on Claude Code native primitives with no external Python services or orchestration frameworks. The plugin manifest lives in `.claude-plugin/plugin.json`; skills, agents, and hooks live at the plugin root. MCP servers (Semantic Scholar via `akapet00/semantic-scholar-mcp` + Brave Search via `@modelcontextprotocol/server-brave-search`) bundle the external API access; Python is only invoked in hook scripts and utility scripts, not as a standalone service. All subagents use `model: sonnet` except the Writing Advisor (Haiku sufficient for critique tasks). The Jan 2026 skills/commands merge means new development should use `skills/<name>/SKILL.md` format exclusively — the old `.claude/commands/` format is legacy.

**Core technologies:**
- Claude Code Plugin System (2.x): Core runtime — the only supported distribution format; provides skills, agents, hooks, MCP natively
- `skills/` + `SKILL.md` format: Phase entry points — `context: fork` isolates each phase; `disable-model-invocation: true` prevents autonomous pipeline advancement
- `agents/*.md` with YAML frontmatter: Specialist role definitions — four separate files enforcing role separation; `memory: project` for cross-session knowledge accumulation
- Semantic Scholar MCP (`akapet00/semantic-scholar-mcp`): Structured paper metadata, citation graphs, field-of-study data — far superior to unstructured web search for literature analysis
- Brave Search MCP (`@modelcontextprotocol/server-brave-search`): Web search for preprints and recent blog posts — recommended over Google CSE (sunsetting Jan 2027)
- `arxiv` PyPI (v2.x) + `semanticscholar` PyPI: Python clients for programmatic paper retrieval in hook and utility scripts
- `.research/` artifact directory: Filesystem-as-state — markdown files per pipeline stage that persist across sessions, compaction events, and restarts
- `CLAUDE.md` (under 200 lines): Ambient domain knowledge loaded into every agent context; survey references, benchmark landscape, open problems

**Critical version requirement:** Claude Code 1.0.33+ for plugin system. Current releases are 2.x.

**Note on Semantic Scholar MCP:** Multiple community implementations exist. `akapet00` has confirmed Claude Code integration but is not an official Anthropic-maintained package. The `semanticscholar` PyPI client is a viable fallback for programmatic access.

### Expected Features

Research surveyed Elicit, SciSpace, Consensus, Agent Laboratory (NeurIPS 2025), AI-Researcher (NeurIPS 2025 Spotlight), Paperguide, Anara, and Web of Science Research Assistant to establish the feature landscape.

**Must have (table stakes):**
- Paper search via arXiv and Semantic Scholar — every academic AI tool has this; users have been trained to expect it
- PDF/paper ingestion via `@` file reference — Claude Code built-in capability; users will attempt this from day one
- Structured per-stage artifact files — all serious research tools (Agent Laboratory, AI-Researcher) produce files, not just conversation
- Multi-stage research pipeline with explicit phase gates — research-trained users expect distinct phases; freeform chat feels unprofessional
- Literature summary generation with grounded citations — table stakes; must include arXiv IDs; citation hallucination at 18-55% rates in unconstrained models is an existential risk
- Research gap identification — SciSpace, AnswerThis, Fynman all have dedicated gap-finder features; users will compare
- Slash-command invocation per stage — Claude Code plugin convention; one command per phase
- Progress state across sessions via artifact files — research spans days/weeks; files are the state, not conversation

**Should have (competitive differentiators):**
- Domain-seeded knowledge (agentic code generation landscape: surveys 2508.00083, 2508.11126, 2509.06216 embedded in CLAUDE.md) — generic tools don't know SWE-bench saturation levels; this is the moat
- Feasibility scoring for research topics (Research Gap Score + one-person-feasibility check) — only Aveksana and Fynman do this; highly valuable for capstone selection under timeline constraints
- Benchmark-aligned experiment design (SWE-bench Verified, SWE-EVO, FeatureBench, HumanEval) — no tool found specifically targeting SWE-bench-family experiment templates
- Four specialized subagents with distinct roles — multi-role specialization is rare in Claude Code plugins; most are single-agent
- Research question refinement loop (broad → specific → evaluable) — tools like Elicit answer questions but don't sharpen the question itself
- Tight timeline prioritization embedded in feasibility scoring — "lockable in 2 weeks" ranked above "most impactful"

**Defer to v2+:**
- Prototype scaffolding bridge (System Architect agent) — valuable but requires topic + experiment plan to exist first; can be added after core pipeline validates
- Self-referential case study prompting — low-complexity angle but Writing Advisor only needed in final phase; add as a prompt tweak post-MVP
- Research question refinement loop (if approximated adequately by Literature Analyst in early iterations)

**Anti-features (explicitly out of scope):**
- Citation management / BibTeX formatting — Zotero does this better; surface DOI/arXiv ID in copyable block for easy export
- Automated benchmark execution — Docker, test runners, compute; way outside plugin scope
- Web scraping / full-text PDF download from publisher sites — copyright risk, rate limiting, legal liability
- Real-time paper alerts / scheduled tracking — requires background process outside Claude Code's model
- General-purpose research assistant for arbitrary domains — dilutes domain-seeded advantage

### Architecture Approach

The architecture follows a plugin-first, skill-orchestrated, subagent-executed pattern where the plugin registers static scaffolding, skills act as phase entry points with `context: fork` isolation, and four specialist subagents write structured output to a shared `.research/` artifact store. Information flows strictly forward through the pipeline (each phase reads the previous artifact, never modifies it) via file path references — never routing full content through a central orchestrator. CLAUDE.md provides ambient domain knowledge to every agent automatically. Hooks enforce quality gates at phase transitions (validating artifact existence and citation completeness before marking a phase complete). The user is an explicit checkpoint between every phase via `disable-model-invocation: true` on all skills.

**Major components:**
1. Plugin Shell (`.claude-plugin/plugin.json` + `CLAUDE.md`) — registers all components; seeds domain knowledge into every agent context
2. Phase Skills (`skills/gsd-*/SKILL.md`) — five `context: fork` entry points, one per pipeline phase; each reads the prior artifact and spawns the appropriate specialist
3. Specialist Subagents (`agents/*.md`) — four role-focused agents with `memory: project`; write directly to artifact files; no direct peer-to-peer communication
4. Artifact Store (`.research/*.md`) — filesystem state that survives session compaction, restarts, and multi-day gaps; five artifacts map to five pipeline stages
5. MCP Servers (`.mcp.json`) — Semantic Scholar and Brave Search; provide structured paper metadata that WebSearch alone cannot deliver
6. Hooks (`hooks/hooks.json` + `scripts/`) — SubagentStop and TaskCompleted handlers validate artifact quality before advancing; use `${CLAUDE_PLUGIN_ROOT}` in all paths

**Build order:** Plugin shell → artifact schemas → agent role definitions → phase skills → helper scripts/hooks. Each layer depends on the previous; this order prevents circular "can't test the skill because the agent doesn't exist" problems.

### Critical Pitfalls

Research identified 5 critical pitfalls and 9 moderate/minor pitfalls across 14 total. The top 5 are:

1. **Misplaced plugin directory structure** — Putting `skills/`, `agents/`, or `hooks/` inside `.claude-plugin/` instead of at the plugin root causes silent failure: plugin installs but nothing registers, no error thrown. Prevention: verify structure against official docs before writing any functional code; run `/help` after loading with `--plugin-dir` and confirm skills appear.

2. **Citation hallucination in Literature Analyst output** — LLMs hallucinate 18-55% of citations under controlled conditions. NeurIPS 2025 accepted papers contained 100+ verified AI-hallucinated references. Prevention: every citation must include a DOI or arXiv ID; tag unverified citations as `[UNVERIFIED]`; add a hook that detects citation patterns without accompanying IDs and blocks artifact completion.

3. **Agent context rot across long research sessions** — At 70%+ context window fill, models lose fidelity on earlier decisions; at 90%+, responses become erratic. Prevention: maintain a persistent `research_state.md` (under 500 lines) loaded at the start of every agent invocation; design this state schema in Phase 1 before any agent is built.

4. **Bag-of-Agents anti-pattern (no orchestration layer)** — Four disconnected agents with no sequencing or artifact dependency enforcement produce locally reasonable but globally inconsistent outputs. MAST taxonomy data shows 17x higher error propagation vs. orchestrated pipelines. Prevention: build the orchestration pipeline skill before connecting agents; enforce artifact precondition checks between phases.

5. **SWE-bench Verified overconfidence** — SWE-bench Verified is ~77% saturated, has 10.6% data leakage to open-source models, and 29.6% of "solved" patches contain behavioral discrepancies. A capstone showing marginal SWE-bench improvement carries no academic weight. Prevention: embed benchmark saturation data in Experiment Designer's SKILL.md; require pairing with harder benchmarks (SWE-EVO ~21%, FeatureBench, SWE-bench Pro); frame contributions against long-horizon or near-miss sub-tasks.

---

## Implications for Roadmap

### Phase 1: Plugin Foundation and State Schema

**Rationale:** Everything depends on the plugin being installable and the state schema being defined. CLAUDE.md domain knowledge must exist before any agent runs. The directory structure pitfall (Pitfall 1) and context bloat pitfall (Pitfall 7) must be caught here, before any functional code exists. The `research_state.md` schema (Pitfall 3) must be designed now — retrofitting state management after agents are built is expensive.

**Delivers:** Installable plugin shell; CLAUDE.md with agentic code generation landscape (surveys, benchmark map, open problems); `.research/` directory with artifact template schemas; `research_state.md` format defined; MCP server declarations in `.mcp.json`; plugin scope set to project (not user-global)

**Addresses:** Plugin shell (table stakes infrastructure); CLAUDE.md under 200 lines (Pitfall 7 prevention); `${CLAUDE_PLUGIN_ROOT}` in all paths (prevents post-install breakage)

**Avoids:** Pitfall 1 (misplaced structure), Pitfall 3 (context rot — state schema designed upfront), Pitfall 7 (CLAUDE.md bloat), Pitfall 13 (wrong install scope)

---

### Phase 2: Literature Search and Topic Selection Agents

**Rationale:** Topic selection is the highest-urgency feature given the "weeks to lock topic" constraint. Literature search feeds topic scoring. These two capabilities must exist and work before the capstone student can make an informed topic decision. This is also where the citation hallucination risk (Pitfall 2) is highest and must be built with verification gates from day one — not retrofitted. The `gsd-topic` and `gsd-lit-review` skills, plus the Literature Analyst agent, are the core MVP.

**Delivers:** `skills/gsd-topic/SKILL.md` with feasibility scoring (novelty, benchmark availability, timeline fit); `agents/literature-analyst.md` with `memory: project`, restricted tool scope (Pitfall 10), and citation verification requirements; `skills/gsd-lit-review/SKILL.md` reading from `topic_shortlist.md`; integration with Semantic Scholar MCP and Brave Search MCP; per-paper `@` reference ingestion

**Uses:** Semantic Scholar MCP, Brave Search MCP, `arxiv` PyPI, `semanticscholar` PyPI, `context: fork` with `disable-model-invocation: true`

**Implements:** Artifact Store layer (`.research/topic_shortlist.md`, `.research/literature_review.md`), Literature Analyst agent, first two phase skills

**Avoids:** Pitfall 2 (citation hallucination — verification gates in SKILL.md from first commit), Pitfall 10 (over-broad tool scope), Pitfall 11 (topic without evaluation path — benchmark alignment in feasibility score), Pitfall 12 (open-access search bias — hard-code Semantic Scholar as required target)

---

### Phase 3: Experiment Designer and Orchestration Layer

**Rationale:** The Bag-of-Agents pitfall (Pitfall 4) makes orchestration the most important component after the individual agents exist. The orchestration pipeline skill — which sequences agents, enforces artifact preconditions, and passes structured handoffs — must be built before any two agents are connected. The Experiment Designer is the second most urgent agent: once a topic is locked, the experiment plan determines whether the capstone is evaluable. This phase also addresses the benchmark overconfidence risk (Pitfall 5) by embedding saturation data in the Experiment Designer.

**Delivers:** `agents/experiment-designer.md` with benchmark landscape domain knowledge (SWE-bench saturation, SWE-EVO, FeatureBench); `skills/gsd-experiment/SKILL.md` with artifact precondition gate (requires `topic_shortlist.md`); orchestration pipeline skill sequencing topic → lit-review → experiment with explicit handoffs; `research_state.md` update logic enforced between phases; handoff prompt templates using narrative casting (Pitfall 9 prevention)

**Uses:** `context: fork`, structured artifact handoffs (file paths not content), `research_state.md` as handoff state

**Implements:** Orchestration layer connecting Literature Analyst output to Experiment Designer input

**Avoids:** Pitfall 4 (Bag-of-Agents), Pitfall 5 (SWE-bench overconfidence), Pitfall 8 (vague sub-agent tasking — explicit scope partitioning in orchestrator), Pitfall 9 (inter-agent state hallucination — narrative casting handoffs)

---

### Phase 4: System Architect and Quality Hooks

**Rationale:** The System Architect agent (prototype scaffolding from experiment plan) is a strong differentiator but depends on the experiment plan artifact existing and being trustworthy. Hooks (quality gates at SubagentStop) should be added here once the core pipeline is working, to enforce artifact quality without blocking early development. This phase converts the pipeline from "works when correct" to "verifiably correct."

**Delivers:** `agents/system-architect.md` producing `implementation_notes.md` from `experiment_plan.md`; `hooks/hooks.json` with SubagentStop validation (artifact existence, citation ID checks, minimum paper count); `scripts/validate_artifact.sh` invoked by hooks; `skills/gsd-implement/SKILL.md` gated on `experiment_plan.md` existence

**Uses:** Hook system (`hooks/hooks.json`), `${CLAUDE_PLUGIN_ROOT}` in hook commands, `jq` for JSON parsing in hook scripts

**Implements:** Quality Gates layer (Layer 6 from architecture); System Architect agent (Layer 3, last specialist)

**Avoids:** Pitfall 6 (over-engineering before validating — hooks added after core pipeline validates, not before), Pitfall 10 (tool scope in Architect agent restricted to Read/Write/Bash/Grep)

---

### Phase 5: Writing Advisor and End-to-End Validation

**Rationale:** The Writing Advisor is last because it depends on all upstream artifacts. The paper draft gating risk (Pitfall 14 — generating results sections before data exists) must be built in from the start of this phase. End-to-end validation of the full pipeline (`/gsd-topic` → `/gsd-lit-review` → `/gsd-experiment` → `/gsd-implement` → `/gsd-paper`) is the phase 5 deliverable: a working research pipeline that produces a coherent paper scaffold from a topic input.

**Delivers:** `agents/writing-advisor.md` with `model: haiku` (appropriate for critique tasks); `skills/gsd-paper/SKILL.md` with hard gate on `experiment_results.md` existence before generating results/conclusion sections; full pipeline end-to-end test on a real research question in the agentic code generation domain; self-referential case study prompting added as Writing Advisor prompt tweak

**Uses:** Haiku model (Writing Advisor), all upstream artifact files, `memory: project` for Writing Advisor to accumulate user style preferences

**Implements:** Final specialist agent; closes the full five-phase pipeline

**Avoids:** Pitfall 14 (premature results generation — hard gate on `experiment_results.md`), Pitfall 6 (over-engineering validated by running the pipeline on a real task before declaring done)

---

### Phase Ordering Rationale

- Foundation before agents: the directory structure and CLAUDE.md domain seeding must exist before any agent can be tested; getting these wrong silently breaks everything
- Artifact schemas before agents: agents cannot produce correct output without agreed schema; defining schemas first allows agents to be tested immediately on first implementation
- Topic selection before literature review: the feature dependency chain is clear (`gap identification → shortlist → topic lock → downstream phases`); implementing in dependency order means each skill can be immediately tested against the prior artifact
- Orchestration layer before connecting agents: the Bag-of-Agents pitfall data (17x error propagation) makes this a hard architectural constraint, not a nice-to-have
- Writing Advisor last: the Writing Advisor's value is in the final phase of the research pipeline; building it first would require mocking all upstream artifacts
- Hooks after core pipeline: quality gates that block completion should not be added before the pipeline produces correct output; adding them too early makes debugging much harder

### Research Flags

Phases likely needing deeper research during planning:

- **Phase 2 (Literature Agent):** Citation verification mechanism design is nuanced — the exact hook implementation for detecting citations without IDs, and the integration between the Semantic Scholar MCP tools and the Literature Analyst system prompt, will need careful specification. Validate the `akapet00` Semantic Scholar MCP tool list against actual use before writing SKILL.md.
- **Phase 3 (Orchestration):** Narrative casting handoff templates (Pitfall 9 prevention) need concrete examples validated against Claude Code's subagent invocation model. The exact mechanism for passing `research_state.md` into a `context: fork` skill context needs verification against current Claude Code docs.

Phases with standard patterns (can skip research-phase):

- **Phase 1 (Plugin Foundation):** Plugin shell structure, CLAUDE.md format, `.mcp.json` syntax, and `plugin.json` schema are all fully documented in official Claude Code docs with HIGH confidence. Standard implementation, no additional research needed.
- **Phase 5 (Writing Advisor):** Haiku model assignment, `disable-model-invocation: true`, and artifact gating patterns are established by phases 1-4. Writing Advisor is a straight application of patterns already built.

---

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | Sourced directly from official Claude Code documentation at code.claude.com. Single exception: Semantic Scholar MCP implementation choice (MEDIUM — multiple community implementations exist; akapet00 confirmed but not Anthropic-maintained) |
| Features | HIGH (table stakes), MEDIUM (differentiators) | Table stakes verified across 8+ tools. Differentiator claims (feasibility scoring, benchmark-aligned experiment design) are inference from absence — no competing tool found with these features, but absence of evidence is not conclusive |
| Architecture | HIGH | Sourced from official Claude Code plugin reference docs plus Anthropic's own multi-agent research system engineering blog. Both first-party HIGH confidence sources |
| Pitfalls | HIGH | MAST taxonomy (peer-reviewed, arXiv 2503.13657), Anthropic engineering blog (first-party), NeurIPS 2025 citation hallucination data (empirically documented). Chroma context rot research and MindStudio skill mistakes guide are MEDIUM confidence community sources that corroborate first-party findings |

**Overall confidence:** HIGH

### Gaps to Address

- **Semantic Scholar MCP tool interface:** The `akapet00` implementation's exact tool names and input schemas should be verified against a live test before the Literature Analyst SKILL.md is written. If the tool interface differs from assumptions, the agent's search instructions will be wrong.
- **`context: fork` state visibility:** Exactly what context a forked skill can read from the parent conversation vs. only from files needs empirical validation. If forked contexts cannot read `research_state.md` by path without explicit instruction, the handoff design requires adjustment.
- **Haiku vs. Sonnet for Writing Advisor:** The STACK.md recommendation assigns Haiku to Writing Advisor for cost reasons. Validate this works for section-level critique quality before committing — if Haiku produces noticeably weaker structural feedback, upgrade to Sonnet.
- **Benchmark feasibility threshold for the capstone timeline:** The capstone timeline constraint is real but the specific thresholds in the feasibility scoring formula (e.g., what makes a topic "lockable in 2 weeks") are not sourced from external research — they are reasonable estimates that should be validated with the capstone advisor.

---

## Sources

### Primary (HIGH confidence)
- [Claude Code Plugin Creation Docs](https://code.claude.com/docs/en/plugins) — plugin structure, manifest, component registration
- [Claude Code Plugins Reference](https://code.claude.com/docs/en/plugins-reference) — skill frontmatter, agent frontmatter, hooks schema
- [Claude Code Subagents Documentation](https://code.claude.com/docs/en/sub-agents) — subagent spawning, model options, memory scopes
- [Claude Code Skills Documentation](https://code.claude.com/docs/en/skills) — `context: fork`, `disable-model-invocation`, `argument-hint`
- [Claude Code Hooks Specification](https://code.claude.com/docs/en/hooks) — hook event types, stdin format, exit codes
- [Claude Code MCP Docs](https://code.claude.com/docs/en/mcp) — `.mcp.json` format, `${CLAUDE_PLUGIN_ROOT}` variable
- [Anthropic Engineering: How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) — artifact-based state, narrative casting, telephone game effect
- [Anthropic Engineering: Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — context rot, lost-in-the-middle
- [Why Do Multi-Agent LLM Systems Fail? (MAST Taxonomy) arXiv:2503.13657](https://arxiv.org/abs/2503.13657) — Bag-of-Agents failure mode, 17x error propagation data
- [Are "Solved Issues" in SWE-bench Really Solved Correctly? arXiv:2503.15223](https://arxiv.org/abs/2503.15223) — SWE-bench behavioral discrepancy data
- [Semantic Scholar Academic Graph API](https://www.semanticscholar.org/product/api) — API capabilities and free tier
- [semanticscholar PyPI package](https://pypi.org/project/semanticscholar/) — Python client
- [arxiv PyPI package v2.x](https://pypi.org/project/arxiv/) — arXiv API wrapper
- [NeurIPS 2025 hallucinated citations documentation](https://gptzero.me/news/neurips/) — citation hallucination at scale, empirical data

### Secondary (MEDIUM confidence)
- [akapet00/semantic-scholar-mcp](https://github.com/akapet00/semantic-scholar-mcp) — Semantic Scholar MCP implementation with confirmed Claude Code integration
- [Agent Laboratory: Using LLMs as Research Assistants](https://agentlaboratory.github.io/) — multi-agent pipeline reference, feature comparison
- [AI-Researcher (NeurIPS 2025 Spotlight)](https://github.com/HKUDS/AI-Researcher) — autonomous research workflow stages
- [The SWE-Bench Illusion arXiv:2506.12286](https://arxiv.org/html/2506.12286v3) — SWE-bench saturation and data leakage analysis
- [Context Rot: How Increasing Input Tokens Impacts LLM Performance — Chroma Research](https://research.trychroma.com/context-rot) — context window degradation thresholds
- [Why Your Multi-Agent System is Failing — Towards Data Science](https://towardsdatascience.com/why-your-multi-agent-system-is-failing-escaping-the-17x-error-trap-of-the-bag-of-agents/) — Bag-of-Agents 17x error propagation corroboration
- [How to Use Claude Code Skills Without Making These 3 Common Mistakes — MindStudio](https://www.mindstudio.ai/blog/claude-code-skills-common-mistakes-guide) — tool scope over-provisioning as #1 skill config mistake
- [Elicit](https://elicit.com/), [SciSpace](https://scispace.com/), [Paperguide](https://paperguide.ai/) — feature landscape comparison

### Tertiary (LOW confidence, needs validation)
- [Building Reliable State Handoffs Between AI Agent Sessions — DEV Community](https://dev.to/aureus_c_b3ba7f87cc34d74d49/building-reliable-state-handoffs-between-ai-agent-sessions-1bk3) — handoff design patterns, validate against official docs
- [Multi-agent orchestration patterns](https://claudefa.st/blog/guide/agents/sub-agent-best-practices) — community guide, corroborates official docs but not authoritative

---
*Research completed: 2026-03-15*
*Ready for roadmap: yes*
