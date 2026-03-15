# Feature Landscape

**Domain:** AI research assistant Claude Code plugin for academic capstone in agentic code generation
**Researched:** 2026-03-15
**Confidence:** HIGH (table stakes verified across 8+ tools), MEDIUM (differentiators, based on gap analysis of existing tools)

---

## Context

This analysis covers features for a Claude Code plugin acting as a multi-agent research team for a Master of Software Engineering capstone focused on LLM-based agentic code generation. The "users" are researchers (primarily: the capstone student) who need to go from topic selection through literature review, experiment design, implementation support, and paper writing.

Reference tools surveyed: Elicit, SciSpace, Consensus, Agent Laboratory (NeurIPS 2025), AI-Researcher (NeurIPS 2025 Spotlight), Paperguide, Anara, Web of Science Research Assistant, and Claude Code plugin ecosystem.

---

## Table Stakes

Features users expect. Missing = the tool feels broken or useless.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Paper search via arXiv / Semantic Scholar | Every academic AI tool has this. Users have been trained by Elicit, SciSpace, etc. | Low | Semantic Scholar has a free MCP server with full API. arXiv API is free with no key required. Both have well-documented endpoints. |
| PDF / paper ingestion via @ file reference | Claude Code natively supports @ references — users will expect to pass papers this way | Low | Claude Code built-in; plugin just needs to define how to handle the content |
| Structured per-stage artifact files | Users expect to produce files, not just conversation. Agent Laboratory, Elicit, AI-Researcher all produce structured output | Medium | Output files like `topic_shortlist.md`, `literature_review.md`, `experiment_plan.md` per stage |
| Multi-stage research pipeline with clear phases | Users with research training expect distinct phases (lit review → experiment design → writing). Freeform chat feels unstructured | Medium | GSD-style stage gates: topic selection → literature review → experiment design → implementation → paper draft |
| Literature summary generation | Every research tool generates summaries. Minimum: per-paper summary with key claims, methods, results | Low-Medium | Must be grounded — always cite source paper, never paraphrase without reference |
| Research gap identification | SciSpace, AnswerThis, and Fynman all have dedicated gap-finder features. Users will compare | Medium | Identify under-explored themes, inconsistencies, missing benchmarks in the LLM code-gen space |
| Slash-command invocation for each stage | Claude Code plugin convention — users invoke stages explicitly (`/research:lit-review`, `/research:experiment-design`) | Low | One command per pipeline stage; arguments allow scoping to topic or subtopic |
| Progress state across sessions | Research spans days/weeks. Users need to resume without re-reading context | Medium | State stored in artifact files, not agent memory — files are the source of truth |
| Grounded citations (no hallucination) | Citation hallucination rates of 18-55% in generic LLMs are well-documented. Users demand accuracy after NeurIPS 2025 scandals | High | Only cite papers retrieved or user-provided. Never assert a paper exists without a retrieved record. RAG-style retrieval required. |

---

## Differentiators

Features that set this plugin apart. Not universally expected, but create meaningful advantage over generic tools.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Domain-seeded knowledge: agentic code generation landscape | Generic tools don't know SWE-bench saturation levels, FeatureBench gaps, or 2024-2026 survey content. This tool is pre-loaded with the field | Medium | Embed key survey knowledge in CLAUDE.md and agent system prompts: arXiv 2508.00083, 2508.11126, 2509.06216 |
| Feasibility scoring for research topics | Only specialized tools (Aveksana, Fynman) do this. A Research Gap Score (0-100%) plus one-person-feasibility check is rare and highly useful for capstone selection | Medium | Score based on: novelty, benchmark availability, implementation complexity, timeline fit (weeks, not months), prior art density |
| Benchmark-aligned experiment design | No tool specifically generates experiment designs targeting SWE-bench Verified / SWE-EVO / FeatureBench / HumanEval. This is a unique value add | High | Agent produces: baseline approach, metric choice, evaluation harness plan, threat-to-validity analysis — all targeting established benchmarks |
| 4 specialized sub-agents with distinct roles | Agent Laboratory and AI-Researcher use specialized agents but are not Claude Code plugins. Most Claude Code plugins are single-agent. Multi-role specialization (Literature Analyst, Experiment Designer, System Architect, Writing Advisor) is rare | High | Each agent has its own system prompt tuned to its specialty. Invoked by the orchestrating slash command based on current stage |
| Self-referential case study angle | The plugin is itself an instance of agentic code generation — it can document itself as a case study, which no competing tool offers | Low | Just requires recognizing and surfacing the meta-angle in the Writing Advisor prompts |
| Experiment design → prototype scaffolding bridge | Most tools stop at experiment design. This plugin also helps with system architecture and prototype scaffolding for the chosen research system | High | System Architect agent produces: architecture diagram sketch, component boundaries, code structure, integration points with benchmarks |
| Tight timeline prioritization | Generic tools don't know you need a topic locked in weeks. This plugin treats timeline as a first-class constraint in feasibility scoring and stage ordering | Low | Embed in topic-selection prompt: rank by "lockable in 2 weeks" ahead of "most impactful" |
| Research question refinement loop | Tools like Elicit answer questions but don't help refine the question itself. A "sharpen your research question" loop (broad → specific → evaluable) is genuinely rare | Medium | Literature Analyst agent runs a structured narrowing: domain → gap → specific question → measurable claim |

---

## Anti-Features

Features to explicitly NOT build. Each has a clear reason and a "what to do instead."

| Anti-Feature | Why Avoid | What to Do Instead |
|--------------|-----------|-------------------|
| Citation management / bibliography formatting (BibTeX, APA, MLA) | Zotero, Mendeley, and Paperpile do this better. Building it badly is worse than not building it. Out of scope in PROJECT.md. | Tell users to export to Zotero. Surface paper metadata (title, authors, DOI, year, arXiv ID) in a copyable block for easy import. |
| Real-time paper alerts / scheduled tracking | Requires background process, cron job, and state management outside Claude Code's model. High complexity for low capstone value. | On-demand search when user asks. User checks arXiv manually or uses a dedicated alert service like Semantic Scholar Alerts. |
| Automated benchmark execution | Executing SWE-bench or HumanEval requires Docker, repositories, test runners, and compute. Way outside plugin scope. | Generate experiment plan and evaluation harness code. User runs evaluations. Plugin helps interpret results after user pastes them. |
| Peer review simulation | Simulating peer reviewer feedback requires deep domain calibration and risks false confidence. Out of scope in PROJECT.md. | Writing Advisor agent checks for clarity, structure, and argument strength — without pretending to be a reviewer. |
| General-purpose research assistant for arbitrary domains | Becoming general-purpose dilutes the domain-seeded advantage. Another Elicit, but worse. | Keep domain lock: agentic code generation. The seeded landscape knowledge is the moat. |
| Web scraping / full-text PDF download from publisher sites | Copyright risk, rate limiting, and legal liability. Most publishers (ACM, IEEE, Springer) prohibit automated scraping. | Support user-provided PDFs via @ references. Use arXiv (open access) for search. Direct users to institutional access for paywalled papers. |
| Plagiarism detection | Turnitin-class tools exist and are institution-mandated. Overlap detection built into a plugin would be low-quality and create liability. | Out of scope. User uses institution-provided tool before submission. |
| Automated paper submission or journal matching | Journal recommendation requires up-to-date acceptance criteria, impact factors, and scope data that go stale fast. High hallucination risk. | Writing Advisor can suggest venue types (workshop vs. conference vs. arXiv-only) based on scope and timeline, but not specific journals by name. |

---

## Feature Dependencies

```
Literature Search (arXiv/Semantic Scholar) → Gap Identification
                                           → Literature Summary Generation

Gap Identification → Research Topic Shortlist
                   → Research Question Refinement Loop

Research Topic Shortlist → Feasibility Scoring

Feasibility Scoring → Topic Lock (prerequisite for all downstream stages)

Topic Lock → Benchmark-Aligned Experiment Design
           → Literature Review (full)

Benchmark-Aligned Experiment Design → System Architect (prototype scaffolding)

Literature Review (full) → Paper Draft: Related Work section
System Architect output  → Paper Draft: Methodology section
Experiment Results (user-provided) → Paper Draft: Results + Discussion sections

PDF Ingestion (@ references) → Literature Summary Generation (any stage)
                             → Writing Advisor review of prior work
```

Key constraint: topic lock must happen before experiment design. The plugin should enforce this by checking for `topic_shortlist.md` existence before allowing `/research:experiment-design`.

---

## MVP Recommendation

Prioritize the following for initial release, in order:

1. **Topic shortlist generation with feasibility scoring** — highest-urgency feature given the "weeks to lock topic" constraint. This is the reason the plugin exists at the start of the capstone.

2. **Literature search + gap identification** (arXiv + Semantic Scholar) — feeds the shortlist. Without this, topic scoring is just guessing.

3. **Per-paper ingestion and summary via @ references** — table stakes, low complexity, unlocks user-provided paper analysis from day one.

4. **Structured artifact files per stage** — the filesystem-as-state pattern means progress persists. Essential from the first command.

5. **Benchmark-aligned experiment design** — differentiator. Once topic is locked, this is the next critical deliverable. Ensures evaluation credibility.

6. **Paper draft scaffolding (section templates)** — useful but can be deferred. Writing phase comes last. A good experiment plan matters more than a draft template at MVP.

**Defer to post-MVP:**

- Research question refinement loop — useful but can be approximated by Literature Analyst in early iterations
- System Architect prototype scaffolding — valuable but requires topic + experiment plan to exist first
- Self-referential case study prompting — angle can be added late as a Writing Advisor prompt tweak

---

## Confidence Notes

| Area | Confidence | Source |
|------|------------|--------|
| Table stakes (search, summary, ingestion) | HIGH | Verified across Elicit, SciSpace, Consensus, Agent Laboratory, AI-Researcher docs |
| Citation hallucination risk | HIGH | NeurIPS 2025 post-mortem, Stanford 2025 study, INRA.AI blog (multi-source agreement) |
| Feasibility scoring as differentiator | MEDIUM | Only 2 tools found with this feature (Aveksana, Fynman); claim that it's differentiating is inference from absence |
| Benchmark-aligned experiment design as differentiator | MEDIUM | No tool found specifically targeting SWE-bench-family experiment templates; absence-based claim |
| Claude Code plugin mechanics (slash commands, skills, agents) | HIGH | Official Anthropic documentation at code.claude.com verified |
| Anti-feature rationale (citation mgmt, scraping, peer review) | HIGH | PROJECT.md explicit out-of-scope + legal/copyright well-documented |

---

## Sources

- [Elicit: AI for scientific research](https://elicit.com/) — table stakes reference
- [7 Best AI Research Assistant Tools for Scientific Research in 2026](https://paperguide.ai/blog/ai-research-assistant-tools-for-scientific-research/) — feature comparison
- [Agent Laboratory: Using LLMs as Research Assistants](https://agentlaboratory.github.io/) — multi-agent pipeline reference
- [AI-Researcher (NeurIPS 2025 Spotlight)](https://github.com/HKUDS/AI-Researcher) — autonomous research workflow stages
- [Create plugins - Claude Code Docs](https://code.claude.com/docs/en/plugins) — plugin architecture (HIGH confidence, official docs)
- [Elicit vs Consensus: Detailed Comparison 2026](https://paperguide.ai/blog/elicit-vs-consensus/) — tool comparison
- [How to Use AI to Detect Research Gaps - SciSpace](https://scispace.com/help/en/articles/10854966-how-to-use-ai-to-detect-research-gaps-in-your-field) — gap analysis features
- [NeurIPS research papers contained 100+ AI-hallucinated citations](https://fortune.com/2026/01/21/neurips-ai-conferences-research-papers-hallucinations/) — citation hallucination risk (HIGH confidence)
- [AI Hallucinations in Research: Why 40% of AI Citations Are Wrong](https://www.enago.com/academy/ai-hallucinations-research-citations/) — citation accuracy data
- [Semantic Scholar Academic Graph API](https://www.semanticscholar.org/product/api) — API integration reference
- [Best 7 AI Tools To Identify Research Gaps](https://insight7.io/best-7-ai-tools-to-identify-research-gaps/) — gap analysis tool landscape
- [Claude Code Agent Skills 2.0](https://medium.com/@richardhightower/claude-code-agent-skills-2-0-from-custom-instructions-to-programmable-agents-ab6e4563c176) — Claude Code skill mechanics
