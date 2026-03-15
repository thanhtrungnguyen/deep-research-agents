# Architecture Patterns

**Project:** Deep Research Agents (Claude Code Plugin)
**Dimension:** Architecture — multi-agent plugin structure for GSD-style research pipeline
**Researched:** 2026-03-15
**Overall confidence:** HIGH (sourced from official Claude Code documentation at code.claude.com)

---

## Recommended Architecture

A **plugin-first, skill-orchestrated, subagent-executed** architecture where:

- The plugin provides the static scaffolding (commands, agents, hooks, CLAUDE.md)
- Skills act as the entry points for each pipeline phase
- Subagents carry out each specialized role (Literature Analyst, Experiment Designer, System Architect, Writing Advisor)
- Markdown files in `.research/` serve as the shared artifact store (state)

```
User invokes /gsd:topic or /gsd:lit-review
        |
        v
 Skill (SKILL.md, context: fork)
 -- defines the phase task --
        |
        v
 Orchestrator subagent (general-purpose or custom)
 -- decomposes task, delegates, synthesizes --
      /   |   |   \
     v    v   v    v
  Agent  Agent Agent Agent
  (lit)  (exp) (arc) (wri)
 -- each writes to .research/<artifact>.md --
        |
        v
 Artifacts persisted to filesystem
 -- topic_shortlist.md, literature_review.md, etc. --
        |
        v
 Next skill reads previous artifact as input
```

---

## Component Boundaries

### Layer 1: Plugin Shell

**What it is:** The Claude Code plugin directory, installed at project scope.

**Responsibility:** Registers all components with Claude Code; provides CLAUDE.md domain seeding.

**Structure:**
```
deep-research-agents/
├── .claude-plugin/
│   └── plugin.json                  # Plugin manifest
├── .claude/
│   └── CLAUDE.md                    # Domain knowledge: agentic code gen landscape
├── commands/                        # Phase entry points (legacy style, or use skills/)
├── skills/                          # Preferred: SKILL.md-based phase skills
│   ├── gsd-topic/SKILL.md           # Phase 1: Topic selection
│   ├── gsd-lit-review/SKILL.md      # Phase 2: Literature review
│   ├── gsd-experiment/SKILL.md      # Phase 3: Experiment design
│   ├── gsd-implement/SKILL.md       # Phase 4: Implementation guidance
│   └── gsd-paper/SKILL.md           # Phase 5: Paper writing
├── agents/                          # Subagent role definitions
│   ├── literature-analyst.md
│   ├── experiment-designer.md
│   ├── system-architect.md
│   └── writing-advisor.md
├── hooks/
│   └── hooks.json                   # TaskCompleted, SubagentStop handlers
└── scripts/                         # Helper utilities (Python)
    ├── arxiv_search.py
    └── semantic_scholar.py
```

**Communicates with:** Claude Code runtime (automatic discovery). Does not communicate directly with any other component — it registers components that Claude Code then loads.

---

### Layer 2: Phase Skills (Entry Points)

**What they are:** SKILL.md files, one per GSD pipeline phase.

**Responsibility:** Define the task for each pipeline phase. Each skill:
1. Accepts arguments (e.g., research topic, paper URLs)
2. Reads the previous phase's artifact from `.research/`
3. Spawns a subagent via `context: fork` to execute the phase task
4. Returns summary to main conversation

**Key frontmatter pattern:**
```yaml
---
name: gsd-lit-review
description: Run a structured literature review for the research topic. Use after topic selection is complete and topic_shortlist.md exists.
context: fork
agent: literature-analyst
disable-model-invocation: true
argument-hint: "[topic or @paper-reference]"
---
```

**Why `context: fork` with `disable-model-invocation: true`:** Phase skills should only run when explicitly invoked by the user (e.g., `/gsd-lit-review`), not when Claude guesses they might be relevant. The `context: fork` isolates each phase in its own context window, preventing cross-phase context contamination.

**Communicates with:** Subagent layer (spawns a subagent); reads/writes artifact layer.

---

### Layer 3: Subagent Role Definitions

**What they are:** Markdown files in `agents/` with YAML frontmatter, defining the four specialized roles.

**Responsibility:** Each subagent has a focused identity, restricted tools, and a system prompt encoding deep domain expertise. They are invoked by phase skills (via `context: fork` + `agent:` field) or directly by the orchestrator.

**Role definitions:**

#### Literature Analyst
```markdown
---
name: literature-analyst
description: Discovers and analyzes academic papers on LLM-based code generation agents. Invoked during lit review phases.
tools: WebSearch, Read, Write, Grep, Bash
model: sonnet
memory: project
---

You are a senior AI research analyst specializing in LLM-based code generation...
[domain expertise system prompt]
```

| Role | Tools | Model | Memory |
|------|-------|-------|--------|
| Literature Analyst | WebSearch, Read, Write, Bash | sonnet | project |
| Experiment Designer | Read, Write, Bash | sonnet | project |
| System Architect | Read, Write, Bash, Grep | sonnet | project |
| Writing Advisor | Read, Write, Grep | sonnet | project |

**Why `memory: project`:** Each agent accumulates codebase-specific knowledge across sessions (e.g., what papers have been reviewed, what benchmarks are selected). Project scope means the memory is checked into `.claude/agent-memory/` and shared with collaborators.

**Communicates with:** Artifact layer (reads and writes `.research/*.md`). Does NOT communicate directly with other agents in the subagent pattern — they report back to the orchestrating skill.

---

### Layer 4: Artifact Store (State Layer)

**What it is:** A `.research/` directory at project root containing structured Markdown files, one per pipeline stage.

**Responsibility:** Persists pipeline state across sessions and across agent invocations. Each artifact is the single source of truth for its stage.

**Artifact map:**

| Artifact | Produced by | Consumed by | Contents |
|----------|-------------|-------------|----------|
| `topic_shortlist.md` | gsd-topic skill | gsd-lit-review skill | 3-5 candidate topics, feasibility scores, gap analysis |
| `literature_review.md` | literature-analyst | gsd-experiment skill | Paper summaries, citations, identified gaps |
| `experiment_plan.md` | experiment-designer | gsd-implement skill | Benchmark selection, hypothesis, methodology |
| `implementation_notes.md` | system-architect | gsd-paper skill | System design decisions, prototype status |
| `paper_draft.md` | writing-advisor | User | Structured sections: abstract → conclusion |

**Design rationale:** File-based artifact storage is the pattern recommended by Anthropic's own multi-agent research system. Subagents write directly to files; the orchestrator passes lightweight file references rather than routing all content through a central context. This avoids context window exhaustion and the "telephone game" problem (content degrading through repeated summarization).

**Communicates with:** All agent layers read from and write to this store. Skills read the previous stage's artifact as input context.

---

### Layer 5: CLAUDE.md Domain Seeding

**What it is:** The project-level CLAUDE.md file.

**Responsibility:** Seeds every session with deep domain knowledge about agentic code generation so agents start with expert context, not generic knowledge.

**Contents:**
- Key surveys (arxiv 2508.00083, 2508.11126, 2509.06216) as summarized reference
- Benchmark landscape: SWE-bench Verified (~77% saturated), SWE-EVO (~21%), FeatureBench, HumanEval
- Active research gaps: multi-agent collaboration, near-miss syndrome, long-horizon tasks
- Plugin-specific conventions: artifact naming, `.research/` structure, invocation patterns

**Why this layer exists:** Subagents receive only their own system prompt (plus CLAUDE.md). By embedding domain knowledge in CLAUDE.md rather than in each agent's system prompt, all agents benefit without duplicating content.

**Communicates with:** Every subagent automatically (CLAUDE.md is loaded into all subagent contexts).

---

### Layer 6: Hooks (Quality Gates)

**What they are:** `hooks/hooks.json` defining event handlers.

**Responsibility:** Enforce quality gates at phase transitions.

**Key hooks:**

```json
{
  "hooks": {
    "TaskCompleted": [
      {
        "matcher": "literature-analyst",
        "hooks": [
          {
            "type": "prompt",
            "command": "Verify literature_review.md exists and contains at least 10 paper summaries with arXiv IDs. If not, exit 2 to block completion."
          }
        ]
      }
    ],
    "SubagentStop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "${CLAUDE_PLUGIN_ROOT}/scripts/validate_artifact.sh"
          }
        ]
      }
    ]
  }
}
```

**Communicates with:** Subagent layer (fires on SubagentStop); artifact layer (validates file existence/quality).

---

## Data Flow

### Phase Transition Flow

```
User: /gsd-topic "multi-agent self-repair for SWE-bench"
      |
      v
[gsd-topic/SKILL.md loads]
      |
      v
[context: fork → general-purpose agent spawned]
  - Reads CLAUDE.md (domain context)
  - Runs literature gap analysis via WebSearch (arXiv, Semantic Scholar)
  - Evaluates feasibility: timeline, benchmarks, novelty
  - Writes .research/topic_shortlist.md
  - Returns summary to main conversation
      |
      v
[User reviews topic_shortlist.md, selects topic]
      |
      v
User: /gsd-lit-review
      |
      v
[gsd-lit-review/SKILL.md loads]
  - Reads .research/topic_shortlist.md (previous artifact)
  - context: fork → literature-analyst agent spawned
      |
      v
[literature-analyst agent]
  - Reads topic_shortlist.md for selected topic
  - Searches arXiv, Semantic Scholar via WebSearch + Bash (Python scripts)
  - Analyzes @-referenced papers if provided by user
  - Writes .research/literature_review.md
  - SubagentStop hook validates artifact
      |
      v
[Summary returned to main conversation]
User continues: /gsd-experiment
```

### Information Flow Rules

1. **Forward-only**: Artifacts only flow forward in the pipeline. A phase reads from the previous artifact, never modifies it. This preserves auditability.

2. **File references not content**: Agents pass `.research/artifact.md` paths between phases, not the full content. This prevents context window exhaustion when artifacts grow large (literature reviews can reach 10,000+ words).

3. **CLAUDE.md is ambient**: Domain knowledge flows into every agent automatically via CLAUDE.md. Agents do not need to "request" domain context.

4. **User as checkpoint**: The user is the explicit checkpoint between phases. Skills have `disable-model-invocation: true` to prevent Claude from autonomously advancing the pipeline. The user decides when to proceed.

5. **Memory accumulates sideways**: Each agent's `memory: project` store accumulates knowledge independently across sessions. A Literature Analyst remembers which papers it has reviewed; an Experiment Designer remembers which benchmarks were considered. This is orthogonal to the forward artifact flow.

---

## Build Order (Component Dependencies)

Build in this order because each layer depends on the previous:

### Phase 1 — Foundation (Build First)
**Components:** Plugin shell, plugin.json, CLAUDE.md

**Why first:** Everything else depends on the plugin being installable. CLAUDE.md domain knowledge must exist before any agent runs, or agents operate without context.

**Deliverable:** `claude plugin install` succeeds; CLAUDE.md loads domain knowledge.

---

### Phase 2 — Artifact Infrastructure
**Components:** `.research/` directory conventions, artifact schemas (Markdown templates)

**Why second:** Agents cannot write artifacts without agreed schemas. Define the shape of `topic_shortlist.md`, `literature_review.md`, etc. before building agents that produce them.

**Deliverable:** Artifact template files exist; schemas documented in CLAUDE.md.

---

### Phase 3 — Agent Role Definitions
**Components:** `agents/literature-analyst.md`, `agents/experiment-designer.md`, `agents/system-architect.md`, `agents/writing-advisor.md`

**Why third:** Skills reference agents by name (`agent: literature-analyst`). Agents must exist before skills that invoke them can be tested.

**Build order within phase:** Literature Analyst first (used in phase 2 of pipeline, highest priority for capstone timeline); Writing Advisor last.

**Deliverable:** `/agents` interface shows all four roles; each agent can be invoked manually and produces correctly-formatted artifacts.

---

### Phase 4 — Phase Skills (Entry Points)
**Components:** `skills/gsd-topic/`, `skills/gsd-lit-review/`, `skills/gsd-experiment/`, `skills/gsd-implement/`, `skills/gsd-paper/`

**Why fourth:** Skills depend on agents (layer 3) and artifacts (layer 2). Build in pipeline order: topic → lit-review → experiment → implement → paper. Each skill can only be tested end-to-end once the agent it invokes exists.

**Deliverable:** Full pipeline can be invoked: `/gsd-topic` → `/gsd-lit-review` → `/gsd-experiment` → `/gsd-implement` → `/gsd-paper`.

---

### Phase 5 — Helper Tooling
**Components:** `scripts/arxiv_search.py`, `scripts/semantic_scholar.py`; hooks validation scripts

**Why last:** Scripts are invoked by agents (not by skills), and agents are already defined. Hooks reference scripts by path; scripts can be added incrementally.

**Deliverable:** Agents can discover papers programmatically; quality gate hooks fire correctly.

---

## Patterns to Follow

### Pattern 1: Context-Forked Phase Skills

**What:** Each GSD phase is a skill with `context: fork` that spawns an isolated subagent.

**When:** Any pipeline phase that should run in isolation, produce artifacts, and not pollute the main conversation context.

**Why:** Phase tasks (literature review, experiment design) generate large intermediate output. Running them in a forked context keeps the main conversation clean and prevents one phase's context from bleeding into the next.

```yaml
---
name: gsd-lit-review
description: Run structured literature review. Use after topic_shortlist.md exists.
context: fork
agent: literature-analyst
disable-model-invocation: true
argument-hint: "[optional: @paper.pdf to include]"
---

You are running the literature review phase of a capstone research pipeline.

1. Read .research/topic_shortlist.md to identify the selected research topic.
2. Search arXiv and Semantic Scholar for papers published 2023-2026 on this topic.
   Run: Bash(python scripts/arxiv_search.py "$TOPIC")
3. For each relevant paper, write a structured summary:
   - Citation (arXiv ID, authors, year)
   - Key contribution
   - Relevance to selected topic
   - Gap or limitation identified
4. Write all summaries to .research/literature_review.md
5. Append a "Research Gaps" section identifying 3-5 open problems.
```

---

### Pattern 2: Artifact-Chained Phases

**What:** Each skill reads the previous phase's artifact as its primary input.

**When:** All phase transitions.

**Why:** Keeps phases decoupled. A skill does not need to know how the previous phase worked — it only needs the artifact. This also allows the user to manually edit artifacts between phases (e.g., curate the topic shortlist before running lit review).

```yaml
# In gsd-experiment/SKILL.md
1. Read .research/literature_review.md
2. Identify the 3 most compelling research gaps.
3. For each gap, design an experiment targeting one of: SWE-bench Verified, SWE-EVO, FeatureBench, HumanEval.
...
```

---

### Pattern 3: Persistent Agent Memory

**What:** Each agent uses `memory: project` for cross-session knowledge accumulation.

**When:** All four specialist agents.

**Why:** Research pipelines span days or weeks. An agent that remembers which papers it has already reviewed, which benchmarks were previously considered, and which experiment designs were rejected avoids redundant work and builds compounding expertise.

```yaml
# In agents/literature-analyst.md frontmatter
memory: project
```

The agent's `MEMORY.md` (at `.claude/agent-memory/literature-analyst/MEMORY.md`) should contain:
- Papers reviewed with arXiv IDs
- Topics already covered
- Identified gaps per topic
- User preferences for paper depth/style

---

### Pattern 4: MCP Servers for External APIs

**What:** Bundle MCP servers for arXiv and Semantic Scholar access.

**When:** The Literature Analyst and Experiment Designer agents need reliable, structured access to paper metadata.

**Why:** WebSearch can find papers but returns unstructured text. An MCP server wrapping the Semantic Scholar API (free, no auth required) returns structured JSON with citation counts, abstracts, and related paper graphs — far more useful for literature analysis.

```json
// .mcp.json
{
  "mcpServers": {
    "semantic-scholar": {
      "command": "python",
      "args": ["${CLAUDE_PLUGIN_ROOT}/scripts/mcp_semantic_scholar.py"],
      "env": {}
    }
  }
}
```

---

## Anti-Patterns to Avoid

### Anti-Pattern 1: Routing All Content Through the Orchestrator

**What:** Having each specialist agent return its full output to a central orchestrator, which then assembles it.

**Why bad:** With 4 agents each producing thousands of words of research content, the orchestrator's context window quickly saturates. Content also degrades through repeated summarization ("telephone game" effect, flagged by Anthropic's own multi-agent system post-mortem).

**Instead:** Agents write directly to artifact files. The orchestrator passes file paths, not content. The next skill reads the file directly.

---

### Anti-Pattern 2: Continuous Agent Teams for Sequential Phases

**What:** Using Claude Code's experimental agent teams feature for the sequential research pipeline.

**Why bad:** Agent teams are designed for parallel work where teammates need to communicate directly with each other. The GSD research pipeline is explicitly sequential (topic → lit review → experiment → paper). Agent teams add coordination overhead, are still experimental, and don't support session resumption reliably.

**Instead:** Use the subagent pattern (context: fork skills + specialist agents). Sequential phases with user checkpoints between them. Use agent teams only if a single phase genuinely needs parallel investigation (e.g., investigating 3 benchmark options simultaneously within the experiment design phase).

---

### Anti-Pattern 3: Monolithic System Prompt

**What:** Encoding all four agent roles (Literature Analyst + Experiment Designer + System Architect + Writing Advisor) into a single agent's system prompt.

**Why bad:** A single agent cannot maintain expertise across four fundamentally different tasks. Literature analysis requires different heuristics than experiment design. A monolithic prompt produces mediocre results across all tasks rather than excellent results in each specialist domain. Context is also wasted loading expertise irrelevant to the current task.

**Instead:** Separate agent files. Claude Code's `context: fork` + `agent:` pattern is precisely designed for this: each phase skill loads the exact right specialist for that task.

---

### Anti-Pattern 4: Storing State in Conversation History

**What:** Relying on the main conversation's context window to remember what was discovered in previous phases.

**Why bad:** Claude Code conversations compact when context fills up. A literature review completed in session 1 will be forgotten by session 3 unless it is persisted externally. Research projects span weeks; conversation history is ephemeral.

**Instead:** The `.research/` artifact store plus agent `memory: project`. Both persist across sessions, compaction events, and Claude Code restarts.

---

### Anti-Pattern 5: Auto-Advancing the Pipeline

**What:** Writing skill descriptions that cause Claude to automatically invoke the next phase when the previous one completes (e.g., "Use after lit-review completes to design experiments").

**Why bad:** The user must review and curate artifacts between phases. An automatically advancing pipeline removes the human judgment that is essential for academic research (choosing which topic from the shortlist, selecting which papers matter most, approving the experiment design before investing implementation time).

**Instead:** All phase skills have `disable-model-invocation: true`. The user explicitly runs `/gsd-lit-review`, `/gsd-experiment`, etc. Pipeline progression is always a deliberate user action.

---

## Scalability Considerations

| Concern | This Project (1 user) | If Extended (team) |
|---------|----------------------|---------------------|
| Context window | Phase skills with context: fork isolate phases; agents compact independently | Same pattern scales; agent memory with project scope is already version-control-friendly |
| Artifact size | Lit reviews can reach 10k+ words — pass file paths, not content | Split large artifacts into topic-specific files (e.g., `literature_review_topic_A.md`) |
| Parallel phases | Not needed for sequential pipeline; use within-phase parallelism (3 papers simultaneously) | Agent teams for truly parallel phase work (rare in academic research) |
| API costs | Sonnet for all agents is appropriate; Haiku could be used for search/indexing tasks | Model routing by task complexity; Haiku for retrieval, Sonnet for synthesis, Opus for final paper review |

---

## Sources

- [Claude Code Plugins Reference](https://code.claude.com/docs/en/plugins-reference) — Official documentation, HIGH confidence
- [Claude Code Subagents](https://code.claude.com/docs/en/sub-agents) — Official documentation, HIGH confidence
- [Claude Code Skills](https://code.claude.com/docs/en/skills) — Official documentation, HIGH confidence
- [Claude Code Agent Teams](https://code.claude.com/docs/en/agent-teams) — Official documentation, HIGH confidence
- [Anthropic Engineering: How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) — Anthropic first-party source, HIGH confidence
- [GSD Framework for Claude Code](https://code.claude.com) — Community research, MEDIUM confidence
