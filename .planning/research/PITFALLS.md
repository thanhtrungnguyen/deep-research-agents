# Domain Pitfalls

**Domain:** Claude Code plugin — multi-agent academic research assistant for agentic code generation
**Researched:** 2026-03-15
**Confidence:** HIGH (verified against official Claude Code docs, peer-reviewed MAST taxonomy, Anthropic engineering blog)

---

## Critical Pitfalls

Mistakes that cause rewrites, data loss, or invalidated research output.

---

### Pitfall 1: Misplaced Plugin Directory Structure

**What goes wrong:** Developers put `skills/`, `agents/`, `hooks/`, or `commands/` directories inside `.claude-plugin/` instead of at the plugin root. The plugin loads silently with no error but none of the skills or agents are registered.

**Why it happens:** The `.claude-plugin/` directory looks like "the plugin folder" so developers put everything inside it. The manifest goes inside `.claude-plugin/`; everything else goes at the root.

**Consequences:** Skills never appear, agents are invisible, hooks never fire. The plugin appears installed but does nothing. Debugging is non-obvious because no error is thrown.

**Correct structure:**
```
deep-research-agents/
├── .claude-plugin/
│   └── plugin.json          ← ONLY this file goes here
├── skills/
│   └── literature-analyst/
│       └── SKILL.md
├── agents/
│   └── experiment-designer.md
├── hooks/
│   └── hooks.json
└── commands/
    └── research-start.md
```

**Prevention:** Verify structure against official docs before first test run. Use `claude --plugin-dir ./plugin-dir` and run `/help` — if no skills appear under the plugin namespace, structure is wrong.

**Detection:** After loading with `--plugin-dir`, run `/help` and look for `/plugin-name:skill-name` entries. Absent entries = structure problem.

**Phase:** Phase 1 (Plugin Scaffold). Catch this before any functional code is written.

---

### Pitfall 2: Citation Hallucination in Literature Agent Output

**What goes wrong:** The Literature Analyst agent generates plausible-sounding but fabricated paper citations. The user incorporates these into a capstone literature review, submitting hallucinated references to an academic committee.

**Why it happens:** LLMs learn citation formatting patterns but don't have access to live paper databases during generation. When asked to recall specific papers, models blend real papers, guess author names, paraphrase titles, or invent DOIs. GPT-4o class models hallucinate 18-29% of citations under controlled conditions; GPT-3.5 class models hallucinate 40-55%. Even NeurIPS 2025 accepted papers contained 100+ AI-hallucinated citations that passed peer review.

**Consequences:** Academic integrity violation. Committee rejection. At minimum, wasted literature review work that must be rebuilt from scratch after verification.

**Prevention:**
- Every paper cited by the Literature Analyst must be verified against arXiv, Semantic Scholar, or ACM Digital Library before inclusion in any output artifact
- The agent's `literature_review.md` artifact must include a `[UNVERIFIED]` tag on any paper the agent discovered via web search (not user-provided via `@` reference)
- Add an explicit verification step in the SKILL.md: "Before listing any paper, state the arXiv ID or DOI. If you cannot produce a valid ID, mark the paper as NEEDS_VERIFICATION rather than inventing one."
- Build a hook or post-processing step that detects citation patterns without accompanying URLs and flags them

**Detection warning signs:**
- Agent cites papers without DOIs or arXiv IDs
- Paper titles are unusually generic or too perfectly on-topic
- Author names don't appear in known author lists for the field
- Conference names are real but year/venue combination doesn't match known proceedings

**Phase:** Phase 2 (Literature Review Agent). Must be addressed in SKILL.md design before the agent produces any output.

---

### Pitfall 3: Agent Context Rot Across a Long Research Session

**What goes wrong:** The research pipeline spans multiple phases (topic selection → literature review → experiment design → writing). As the conversation grows, the lead agent loses fidelity on earlier decisions. At 70% context window, Claude starts losing precision; at 85%, hallucination rate increases; at 90%+, responses become erratic.

**Why it happens:** The "lost-in-the-middle" effect causes models to weight content at the beginning and end of context more heavily. Details from the topic selection phase (feasibility constraints, chosen direction, benchmark targets) get buried and effectively forgotten by the time the writing agent is active.

**Consequences:** The Writing Advisor generates a paper introduction that contradicts the Experiment Designer's methodology. The chosen research direction drifts. Artifacts from earlier phases become inconsistent with later ones.

**Prevention:**
- Maintain a persistent `research_state.md` file that is explicitly loaded at the start of every agent invocation — this is the single source of truth for current topic, chosen benchmarks, open hypotheses, and pipeline stage
- Keep `research_state.md` under 500 lines by summarizing (not appending) at each phase transition
- Each agent skill should begin its SKILL.md with: "Read research_state.md before taking any action. Do not proceed if the file is absent."
- When the context exceeds 150K tokens, spawn a fresh subagent with a clean context and pass `research_state.md` as the handoff artifact

**Detection warning signs:**
- Agent references a benchmark or paper that was rejected in an earlier phase
- Agent asks clarification questions that were answered earlier in the session
- Output artifacts contradict each other on key decisions

**Phase:** Phase 1 (Plugin Scaffold) — design the state file schema. Phase 2+ — enforce it in every agent skill.

---

### Pitfall 4: Bag-of-Agents Anti-Pattern (No Orchestration Layer)

**What goes wrong:** Four agents are defined (Literature Analyst, Experiment Designer, System Architect, Writing Advisor) but there is no orchestration logic. Each agent operates as a standalone slash command with no shared context, no sequencing enforcement, and no verification between stages. The user manually calls each agent and manually pastes output between them.

**Why it happens:** This is the default if agents are implemented as independent skills without a pipeline skill that sequences them. The MAST taxonomy (2503.13657) identifies this as the "Bag of Agents" failure — flat topology, no assurance plane, open-loop execution. Analysis of 1,600+ agent traces found this causes 17x higher error propagation compared to orchestrated pipelines.

**Consequences:** Each agent makes locally reasonable decisions that are globally inconsistent. The Experiment Designer designs experiments for a benchmark the Literature Analyst already determined is saturated. Research artifacts diverge.

**Prevention:**
- Implement a `research-pipeline` orchestrator skill that calls sub-agents in sequence with explicit artifact handoffs
- Each sub-agent receives: (1) `research_state.md`, (2) the output artifact from the prior stage, (3) its specific task description
- Add a verification gate between stages: before advancing from literature review to experiment design, a brief check confirms that topic selection artifacts exist and are complete
- Define termination conditions explicitly in each agent's SKILL.md — agents must know when their task is done

**Detection warning signs:**
- User is manually copying content between agent invocations
- Later-stage agents ask questions that earlier-stage artifacts already answer
- No clear artifact dependency chain exists

**Phase:** Phase 3 (Agent Orchestration). The orchestration layer is more important than any individual agent.

---

### Pitfall 5: SWE-bench Evaluation Overconfidence

**What goes wrong:** The Experiment Designer recommends benchmarking against SWE-Bench Verified as the primary evaluation. The user treats a percentage score as definitive proof of research contribution. Reviewers (or the committee) point out that SWE-Bench Verified is approaching saturation (~77% solved), has 10.6% data leakage to open-source models, and 29.6% of "passed" patches contain behavioral discrepancies versus ground truth.

**Why it happens:** SWE-Bench is the most cited benchmark in the field, so it appears to be the obvious choice. The saturation problem, data leakage, and test coverage gaps are documented in 2025-2026 literature but not widely known to new researchers.

**Consequences:** The capstone contribution appears incremental or meaningless. The committee questions whether the reported improvements are genuine. The research direction may need to be pivoted late in the timeline.

**Prevention:**
- The Experiment Designer agent must explicitly flag SWE-Bench Verified saturation in every experiment plan
- Recommend pairing SWE-Bench with harder benchmarks: SWE-EVO (~21% solved), FeatureBench, SWE-bench Pro
- If using SWE-Bench Verified, acknowledge test coverage limitations and run additional tests beyond the standard harness
- Frame contributions against harder sub-tasks rather than overall solve rate (e.g., "long-horizon issues," "multi-file changes," "near-miss recovery")
- Seed the Experiment Designer's SKILL.md with the benchmark landscape knowledge from PROJECT.md

**Detection warning signs:**
- Experiment plan lists only SWE-Bench Verified as the evaluation target
- No baseline comparison beyond the current leaderboard top entries
- No discussion of test coverage adequacy or data contamination checks

**Phase:** Phase 2 (Experiment Designer). Must be embedded as domain knowledge in the agent's SKILL.md.

---

## Moderate Pitfalls

---

### Pitfall 6: Over-Engineering the Plugin Before Validating the Workflow

**What goes wrong:** All four agents, the orchestrator, hooks, and MCP integrations are built before the user has run a single end-to-end research session. The resulting system is complex but does not match how the user actually works.

**Why it happens:** It is tempting to build the complete architecture upfront, especially since the plugin structure is known. The GSD methodology patterns also create pressure to deliver all phases.

**Consequences:** Significant rework when it turns out the agent interaction model doesn't fit the user's research style. Plugin complexity makes debugging hard.

**Prevention:** Start with one skill (topic selection or literature search) and run it against a real research task before building subsequent agents. The official Claude Code docs explicitly recommend: "Start with basic CLAUDE.md + a few commands, test in production for 2 weeks, and add agents/skills only if need is proven."

**Phase:** Phase 1 (Plugin Scaffold). Design for extensibility but implement incrementally.

---

### Pitfall 7: CLAUDE.md Context Bloat

**What goes wrong:** All domain knowledge — benchmark descriptions, survey papers, research methodology, agent role definitions, and coding conventions — is loaded into a single CLAUDE.md. This context loads on every interaction, degrading performance even for simple tasks.

**Why it happens:** CLAUDE.md is the obvious place for project context. It is easy to append without discipline.

**Consequences:** Every Claude Code interaction pays a large context tax. Important instructions get buried and lose weight (lost-in-the-middle effect). The file becomes unmaintainable.

**Prevention:**
- Keep CLAUDE.md under 200 lines; restrict to: project purpose, active research phase, location of key artifact files, and which plugin skills are available
- Move benchmark knowledge and survey content into agent-specific SKILL.md files where it is only loaded when that agent is invoked
- Use `research_state.md` for current session state, not CLAUDE.md

**Phase:** Phase 1 (Plugin Scaffold). Establish CLAUDE.md scope discipline from the start.

---

### Pitfall 8: Vague Sub-Agent Task Descriptions Causing Duplicated Work

**What goes wrong:** The orchestrator spawns multiple sub-agents with instructions like "research agentic code generation" without explicit scope partitioning. Multiple agents investigate the same papers, time periods, or topics.

**Why it happens:** Anthropic's own engineering blog documents exactly this failure: early agents given "research the semiconductor shortage" had two agents redundantly investigating 2025 supply chains while the 2021 automotive crisis was missed.

**Consequences:** Token budget wasted. Research gaps form because no agent was explicitly assigned to cover them. Orchestrator receives redundant reports that don't cover the full topic space.

**Prevention:**
- Each sub-agent invocation must include: explicit scope ("papers published 2024-2026 only"), explicit exclusions ("do not search for SWE-Bench papers, that is covered by another agent"), and explicit output format
- When the orchestrator spawns parallel research agents, partition by: time period, subtopic, or benchmark category — never by vague keywords

**Phase:** Phase 3 (Agent Orchestration). Encode scope partitioning templates in the orchestrator SKILL.md.

---

### Pitfall 9: Inter-Agent State Hallucination During Handoff

**What goes wrong:** When the Writing Advisor agent receives the literature review artifact and begins generating prose, it hallucinates that it performed the literature search itself. It may contradict the search methodology or claim to have read papers it didn't analyze.

**Why it happens:** If a handoff presents "Assistant" messages from a prior agent to a new agent, the new agent interprets those messages as its own prior outputs. MAST taxonomy classifies this as "reasoning-action mismatch." Anthropic's engineering blog recommends "narrative casting" — re-framing prior agent outputs as narrative context rather than as the new agent's own messages.

**Consequences:** The Writing Advisor makes inconsistent claims about research methodology. The paper draft contradicts the experiment plan.

**Prevention:**
- Handoff prompt template: "The following is a summary produced by the Literature Analyst agent in a prior session. You are the Writing Advisor. Your task is to use this summary as source material..."
- Never pass raw conversation history from one agent as the direct input to another
- Pass structured artifact files (`literature_review.md`, `experiment_plan.md`) rather than conversation transcripts

**Phase:** Phase 3 (Agent Orchestration). Define handoff prompt templates before agents are connected.

---

### Pitfall 10: Insufficient Tool Scope in Skill Definitions

**What goes wrong:** The Literature Analyst skill is granted write access to all project files "just in case." A security monitor event fires, or the agent accidentally overwrites an existing artifact file.

**Why it happens:** Granting broad tool access is easier than thinking through minimum required permissions. The MindStudio guide documents this as the #1 skill configuration mistake.

**Consequences:** Security monitor blocks the agent mid-task. Artifact files from prior phases are overwritten. Trust in the pipeline degrades.

**Prevention:**
- Each SKILL.md `tools` declaration should list only the minimum required: a read-only literature search skill needs `WebSearch,Read` not `Write,Edit,Bash`
- Write-capable tools should only appear in skills that explicitly produce output artifacts
- Apply the principle of least privilege: add tools only when a specific functional need is identified

**Phase:** Phase 2 (Individual Agent Skills). Define tool scope per-skill during agent implementation.

---

## Minor Pitfalls

---

### Pitfall 11: Research Topic Too Novel for Benchmark Evaluation

**What goes wrong:** The topic shortlist surfaces a genuinely novel direction (e.g., "agents that evolve their own scaffolding") that has no established benchmark. The research direction is intellectually interesting but cannot be evaluated credibly within capstone scope.

**Why it happens:** The Topic Selection agent optimizes for novelty and gap analysis without simultaneously checking evaluability.

**Prevention:** The Topic Selection agent must produce a feasibility score that includes "benchmark alignment" as a required field. Any topic without a clear evaluation path against SWE-EVO, FeatureBench, HumanEval, or a defensible custom benchmark should be flagged as high-risk.

**Phase:** Phase 2 (Topic Selection). Bake evaluability scoring into the topic shortlist template.

---

### Pitfall 12: Bias Toward Open-Access Papers in Web Search

**What goes wrong:** The Literature Analyst's web search consistently surfaces arXiv preprints while missing relevant ACL, ICLR, NeurIPS, or ICSE papers that are behind paywalls or poorly indexed by search engines.

**Why it happens:** AI search agents preferentially return open-access content because it is better indexed and more likely to appear in training data. Anthropic's engineering blog documents that early research agents consistently selected "SEO-optimized content farms over authoritative academic PDFs."

**Prevention:**
- Hard-code search targets in the Literature Analyst SKILL.md: always search arXiv AND Semantic Scholar (which has broad coverage of subscription venues)
- Prompt the agent to explicitly note whether a paper was found on arXiv or in a peer-reviewed venue and to prefer peer-reviewed venues for foundational claims
- Encourage the user to supplement with `@`-referenced papers from their institution's library access

**Phase:** Phase 2 (Literature Review Agent). Encode search strategy diversity in SKILL.md.

---

### Pitfall 13: Treating Plugin-Scope vs. Standalone-Scope as Interchangeable

**What goes wrong:** The plugin is installed at user scope (global) when it should be project-scoped, causing the research agents and domain knowledge to pollute other unrelated Claude Code sessions.

**Why it happens:** Plugin scope is a single flag at installation time. The distinction between user scope (all projects) and project scope (this repo only) is easy to overlook.

**Prevention:** Install with explicit project scope. Document the intended scope in the plugin README. The plugin's domain knowledge (agentic code generation landscape, capstone constraints) is irrelevant and distracting in non-research contexts.

**Phase:** Phase 1 (Plugin Scaffold). Set installation scope in documentation from the start.

---

### Pitfall 14: Writing Phase Generating Paper Structure Before Results Exist

**What goes wrong:** The Writing Advisor is invoked too early — before experiments are run — and generates a complete paper draft with placeholder result sections. The user treats the structure as fixed and finds it difficult to revise after actual results differ from the assumed narrative.

**Why it happens:** The Writing Advisor is genuinely useful for methodology and introduction sections early on, but its helpfulness creates pressure to fill all sections prematurely.

**Prevention:** The Writing Advisor SKILL.md should enforce a gating check: "Do not generate results, discussion, or conclusion sections until `experiment_results.md` exists and is non-empty." Explicitly state this gate in the orchestrator's sequencing logic.

**Phase:** Phase 4 (Writing Agent). Implement the gate in SKILL.md before the writing agent is deployed.

---

## Phase-Specific Warning Map

| Phase | Topic | Likely Pitfall | Mitigation |
|-------|-------|---------------|------------|
| 1 — Plugin Scaffold | Directory structure | Misplaced `skills/` inside `.claude-plugin/` (Pitfall 1) | Verify against official docs at first load |
| 1 — Plugin Scaffold | CLAUDE.md design | Context bloat degrading all interactions (Pitfall 7) | Cap CLAUDE.md at 200 lines; move domain knowledge to SKILL.md files |
| 1 — Plugin Scaffold | State design | No `research_state.md` schema defined (Pitfall 3) | Design state file schema before any agent is implemented |
| 2 — Agent Skills | Literature agent | Citation hallucination in output (Pitfall 2) | Mandatory verification tags; no citation without DOI |
| 2 — Agent Skills | Experiment agent | SWE-Bench Verified overconfidence (Pitfall 5) | Embed benchmark saturation knowledge in SKILL.md |
| 2 — Agent Skills | Topic selection | Uneval uable research topic selected (Pitfall 11) | Require benchmark alignment in feasibility score |
| 2 — Agent Skills | Literature search | Open-access search bias (Pitfall 12) | Hard-code Semantic Scholar as a required search target |
| 2 — Agent Skills | Tool permissions | Over-broad tool scope (Pitfall 10) | Define minimum tool sets per skill |
| 3 — Orchestration | Agent pipeline | Bag-of-Agents, no sequencing (Pitfall 4) | Build orchestrator before connecting agents |
| 3 — Orchestration | Sub-agent tasking | Vague descriptions cause duplicated work (Pitfall 8) | Partition by scope/time/topic with explicit exclusions |
| 3 — Orchestration | Handoff design | Inter-agent state hallucination (Pitfall 9) | Use narrative casting; pass artifacts not transcripts |
| 4 — Writing Agent | Paper draft | Generating results sections before data exists (Pitfall 14) | Gate on `experiment_results.md` existence |
| All phases | Scope | Over-engineering before validating workflow (Pitfall 6) | Build one skill, validate end-to-end, then expand |

---

## Sources

- [Create plugins — Claude Code Official Docs](https://code.claude.com/docs/en/plugins) — HIGH confidence
- [Why Do Multi-Agent LLM Systems Fail? (MAST Taxonomy) arXiv:2503.13657](https://arxiv.org/abs/2503.13657) — HIGH confidence (peer-reviewed)
- [Anthropic Engineering: How we built our multi-agent research system](https://www.anthropic.com/engineering/multi-agent-research-system) — HIGH confidence
- [Anthropic Engineering: Effective context engineering for AI agents](https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents) — HIGH confidence
- [GPTZero: 100 hallucinations in NeurIPS 2025 papers](https://gptzero.me/news/neurips/) — HIGH confidence (documented empirical study)
- [Are "Solved Issues" in SWE-bench Really Solved Correctly? arXiv:2503.15223](https://arxiv.org/abs/2503.15223) — HIGH confidence
- [The SWE-Bench Illusion arXiv:2506.12286](https://arxiv.org/html/2506.12286v3) — MEDIUM confidence
- [Why Your Multi-Agent System is Failing — Towards Data Science](https://towardsdatascience.com/why-your-multi-agent-system-is-failing-escaping-the-17x-error-trap-of-the-bag-of-agents/) — MEDIUM confidence
- [How to Use Claude Code Skills Without Making These 3 Common Mistakes — MindStudio](https://www.mindstudio.ai/blog/claude-code-skills-common-mistakes-guide) — MEDIUM confidence
- [Context Rot: How Increasing Input Tokens Impacts LLM Performance — Chroma Research](https://research.trychroma.com/context-rot) — MEDIUM confidence
- [Building Reliable State Handoffs Between AI Agent Sessions — DEV Community](https://dev.to/aureus_c_b3ba7f87cc34d74d49/building-reliable-state-handoffs-between-ai-agent-sessions-1bk3) — MEDIUM confidence
