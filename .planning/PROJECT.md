# Deep Research Agents

## What This Is

A Claude Code plugin that acts as a team of senior AI researchers specializing in LLM-based code generation and software engineering agents. It provides a GSD-style structured pipeline — from research topic discovery through literature review, experiment design, implementation, and paper writing — tailored for a Master of Software Engineering capstone focused on agentic coding systems.

## Core Value

Help the user select, validate, and execute a high-quality capstone research project in agentic code generation — producing a publishable paper backed by benchmark evaluations — within a tight timeline (weeks to lock topic).

## Requirements

### Validated

(None yet — ship to validate)

### Active

- [ ] Multi-agent research team with 4 specialized roles: Literature Analyst, Experiment Designer, System Architect, Writing Advisor
- [ ] GSD-style phased workflow: topic selection → literature review → experiment design → implementation → paper writing
- [ ] Literature discovery via web search (arXiv, Semantic Scholar) + user-provided paper analysis via @ references
- [ ] Research topic shortlist generation with feasibility scoring and gap analysis
- [ ] Experiment design support targeting coding benchmarks (SWE-bench, SWE-EVO, FeatureBench, HumanEval)
- [ ] Prototype/system building assistance for the chosen research direction
- [ ] Paper draft generation with structured sections (abstract, intro, related work, methodology, results, discussion)
- [ ] Domain knowledge seeded with agentic code generation landscape (surveys, benchmarks, open problems from 2024-2026)
- [ ] Claude Code plugin best practices: proper plugin.json, skills, hooks, agents, CLAUDE.md configuration
- [ ] Artifact management: structured output files for each pipeline stage (topic_shortlist.md, literature_review.md, experiment_plan.md, etc.)

### Out of Scope

- General-purpose research tool for arbitrary domains — this is capstone-focused on agentic code generation
- Real-time paper tracking/alerting — user provides papers or agents search on-demand
- Automated benchmark execution — the plugin helps design and plan experiments, user runs them
- Citation management / bibliography formatting — use Zotero or similar external tools
- Peer review simulation — agents advise on writing quality but don't simulate reviewers

## Context

**Research domain**: LLM-based code generation agents — a rapidly growing field (53% of papers published in 2024, accelerating in 2025-2026).

**Key landscape findings**:
- SWE-bench Verified is saturating (~77%) but harder benchmarks like SWE-EVO (~21%) and FeatureBench are wide open
- Multi-agent collaboration, self-evolving agents, and hierarchical debugging are documented research gaps
- The "near-miss syndrome" (agents get close but fail on subtle errors) is an active problem
- Long-horizon software evolution tasks remain largely unsolved
- Industry adoption is high (57% have agents in production) but academic evaluation frameworks lag behind

**User profile**: Master of Software Engineering student, comfortable with Python, interested in AI engineering and practical implementation, prefers research + system building combination.

**Key surveys to build on**:
- "A Survey on Code Generation with LLM-based Agents" (arxiv 2508.00083, Sept 2025)
- "AI Agentic Programming: A Survey of Techniques, Challenges, and Opportunities" (arxiv 2508.11126)
- "Agentic Software Engineering: Foundational Pillars and a Research Roadmap" (arxiv 2509.06216)

**Meta angle**: The plugin itself is built on the same agentic coding technology it studies — potential case study material.

## Constraints

- **Timeline**: Topic must be locked within weeks — prioritize topic selection and feasibility analysis
- **Tech stack**: Claude Code plugin (skills, hooks, agents, CLAUDE.md), Python for any tooling/scripts
- **Evaluation**: Must use established benchmarks (SWE-bench family, HumanEval, FeatureBench) for experimental credibility
- **Scope**: Master's capstone level — feasible for one person, experimentally evaluable, technically solid
- **Platform**: Claude Code plugin architecture — skills, agents, hooks, plugin.json structure

## Key Decisions

| Decision | Rationale | Outcome |
|----------|-----------|---------|
| GSD-style phased workflow | User prefers structured stages over freeform conversation — matches research methodology | — Pending |
| 4 agent roles (Lit Analyst, Experiment Designer, System Architect, Writing Advisor) | Covers full research pipeline; each role has distinct expertise and output artifacts | — Pending |
| Capstone-focused, not general-purpose | Tight timeline demands focus; domain knowledge can be deeply embedded | — Pending |
| Benchmark evaluation approach | SWE-bench family provides established baselines; SWE-EVO/FeatureBench have clear gaps to contribute to | — Pending |
| Both web search + user-provided papers | Maximizes paper coverage; user can feed specific papers while agents discover related work | — Pending |

---
*Last updated: 2026-03-15 after initialization*
