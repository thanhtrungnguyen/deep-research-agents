# Roadmap: Deep Research Agents

## Overview

This plugin is built in five phases, each delivering a vertically complete capability that can be tested immediately. The build order is dictated by strict dependency: the plugin shell must exist before agents can run, artifact schemas must exist before agents can produce output, topic selection must work before literature review, and the orchestration layer must exist before connecting any two agents. The Writing Advisor is last because it depends on all upstream artifacts. Quality hooks are added after the core pipeline is proven — not before.

## Phases

**Phase Numbering:**
- Integer phases (1, 2, 3): Planned milestone work
- Decimal phases (2.1, 2.2): Urgent insertions (marked with INSERTED)

Decimal phases appear between their surrounding integers in numeric order.

- [ ] **Phase 1: Plugin Foundation** - Installable plugin shell with domain knowledge, artifact schemas, and state schema
- [ ] **Phase 2: Literature Discovery and Topic Selection** - Literature Analyst agent with MCP search and citation verification; topic shortlist with feasibility scoring
- [ ] **Phase 3: Experiment Design and Orchestration** - Experiment Designer agent with benchmark knowledge; orchestration layer sequencing the full pipeline
- [ ] **Phase 4: Implementation Support and Quality Hooks** - System Architect agent for prototype scaffolding; hook-based quality gates enforcing artifact integrity
- [ ] **Phase 5: Paper Writing and End-to-End Validation** - Writing Advisor agent with results gating; full pipeline validated against a real research question

## Phase Details

### Phase 1: Plugin Foundation
**Goal**: A fully installable Claude Code plugin that seeds the research domain into every agent context and defines the artifact schema that all downstream agents write to
**Depends on**: Nothing (first phase)
**Requirements**: FOUND-01, FOUND-02, FOUND-03, FOUND-04, FOUND-05
**Success Criteria** (what must be TRUE):
  1. Running `/help` in a Claude Code session with `--plugin-dir` pointing to this plugin shows all registered skills and agents with no errors
  2. CLAUDE.md loads the agentic code generation domain landscape (surveys, benchmark saturation data, open problems) into every agent context without exceeding the 200-line limit
  3. The `.research/` directory contains typed markdown schemas for all five pipeline stages, and any agent invocation loads `research_state.md` as its first context action
  4. The `.mcp.json` declares Semantic Scholar and Brave Search MCP server entries that Claude Code can resolve
**Plans**: TBD

### Phase 2: Literature Discovery and Topic Selection
**Goal**: Users can discover relevant papers, ingest their own papers via `@` reference, and generate a scored topic shortlist that enables an informed capstone topic decision
**Depends on**: Phase 1
**Requirements**: LIT-01, LIT-02, LIT-03, LIT-04, LIT-05, LIT-06, TOPC-01, TOPC-02, TOPC-03, AGNT-01
**Success Criteria** (what must be TRUE):
  1. User runs `/gsd-topic` and receives a `topic_shortlist.md` with at least three candidate research directions, each scored on novelty, benchmark availability, one-person feasibility, and timeline fit
  2. User runs `/gsd-lit-review` and the Literature Analyst agent returns a `literature_review.md` with summaries of discovered papers where every citation includes a DOI or arXiv ID (no unverified references)
  3. User can provide a PDF or paper via `@` file reference and the Literature Analyst ingests and analyzes it alongside search results
  4. The Literature Analyst identifies and documents at least one research gap from the analyzed literature, written into `literature_review.md`
  5. User can query citation networks from a seed paper via Semantic Scholar MCP to discover related works
**Plans**: TBD

### Phase 3: Experiment Design and Orchestration
**Goal**: Users can generate an experiment plan for their chosen topic, and the pipeline sequences topic → literature review → experiment with enforced artifact preconditions that prevent Bag-of-Agents divergence
**Depends on**: Phase 2
**Requirements**: EXPR-01, EXPR-02, EXPR-03, EXPR-04, AGNT-02, AGNT-05, AGNT-06, AGNT-07
**Success Criteria** (what must be TRUE):
  1. User runs `/gsd-experiment` and the Experiment Designer generates an `experiment_plan.md` that pairs the chosen topic with a benchmark from the SWE-bench family, HumanEval, or FeatureBench — with current saturation data cited so the user can assess academic contribution
  2. The Experiment Designer actively warns when a proposed benchmark is saturated (e.g., SWE-bench Verified ~77%) and recommends less-saturated alternatives (SWE-EVO ~21%, FeatureBench)
  3. Invoking `/gsd-experiment` without a valid `topic_shortlist.md` artifact fails with a clear error explaining the missing precondition — the pipeline cannot be run out of order
  4. The orchestration skill sequences topic → lit-review → experiment with `research_state.md` updated between each phase, so the experiment plan is grounded in the literature review findings
**Plans**: TBD

### Phase 4: Implementation Support and Quality Hooks
**Goal**: Users can generate a system architecture and code scaffold for their research prototype, and all pipeline stage transitions are quality-gated by hooks that validate artifact completeness
**Depends on**: Phase 3
**Requirements**: IMPL-01, IMPL-02, AGNT-03, AGNT-04
**Success Criteria** (what must be TRUE):
  1. User runs `/gsd-implement` and the System Architect produces an `implementation_notes.md` containing a system architecture design and Python code scaffold tailored to the experiment plan in `experiment_plan.md`
  2. When the Literature Analyst completes, a SubagentStop hook validates that `literature_review.md` exists, contains at least one paper with a verified arXiv ID or DOI, and blocks advancement if the check fails
  3. Running `/gsd-implement` without a valid `experiment_plan.md` produces a clear precondition error — the skill cannot execute without the upstream artifact
**Plans**: TBD

### Phase 5: Paper Writing and End-to-End Validation
**Goal**: Users can generate structured paper drafts grounded in their actual research artifacts, and the full five-skill pipeline produces a coherent research scaffold from topic input to paper draft
**Depends on**: Phase 4
**Requirements**: PAPR-01, PAPR-02, PAPR-03
**Success Criteria** (what must be TRUE):
  1. User runs `/gsd-paper` and the Writing Advisor generates section-by-section paper content (abstract, introduction, related work, methodology) grounded in `literature_review.md` and `experiment_plan.md`
  2. Requesting results or discussion sections without a valid `experiment_results.md` artifact fails with a clear gate message — the Writing Advisor does not generate results from incomplete data
  3. The related work section cites only papers that appear in `literature_review.md` — no hallucinated references are present in generated paper content
  4. A user can run the full pipeline (`/gsd-topic` → `/gsd-lit-review` → `/gsd-experiment` → `/gsd-implement` → `/gsd-paper`) on a real agentic code generation research question and receive a coherent research scaffold with consistent terminology and grounded citations across all artifacts
**Plans**: TBD

## Progress

**Execution Order:**
Phases execute in numeric order: 1 → 2 → 3 → 4 → 5

| Phase | Plans Complete | Status | Completed |
|-------|----------------|--------|-----------|
| 1. Plugin Foundation | 0/TBD | Not started | - |
| 2. Literature Discovery and Topic Selection | 0/TBD | Not started | - |
| 3. Experiment Design and Orchestration | 0/TBD | Not started | - |
| 4. Implementation Support and Quality Hooks | 0/TBD | Not started | - |
| 5. Paper Writing and End-to-End Validation | 0/TBD | Not started | - |
