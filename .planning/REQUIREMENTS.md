# Requirements: Deep Research Agents

**Defined:** 2026-03-15
**Core Value:** Help the user select, validate, and execute a high-quality capstone research project in agentic code generation — producing a publishable paper backed by benchmark evaluations.

## v1 Requirements

Requirements for initial release. Each maps to roadmap phases.

### Plugin Foundation

- [ ] **FOUND-01**: Plugin manifest (plugin.json) with proper structure, auto-discovery of all components
- [ ] **FOUND-02**: CLAUDE.md seeded with agentic code generation domain knowledge (surveys, benchmarks, open problems, 2024-2026 landscape)
- [ ] **FOUND-03**: `.claude/rules/` files for overflow domain knowledge beyond CLAUDE.md 200-line limit
- [ ] **FOUND-04**: `.research/` artifact directory with typed markdown schemas per pipeline stage (topic_shortlist.md, literature_review.md, experiment_plan.md, implementation_notes.md, paper_draft.md)
- [ ] **FOUND-05**: `research_state.md` state file loaded at every agent invocation to prevent context rot across sessions

### Agent Team

- [ ] **AGNT-01**: Literature Analyst agent with system prompt, tool scoping (WebSearch, WebFetch, MCP tools), and `memory: project`
- [ ] **AGNT-02**: Experiment Designer agent with embedded benchmark saturation knowledge, tool scoping, and `memory: project`
- [ ] **AGNT-03**: System Architect agent with code scaffolding capabilities (Read, Write, Edit, Bash, Glob, Grep), and `memory: project`
- [ ] **AGNT-04**: Writing Advisor agent with paper section generation capabilities and `memory: project`
- [ ] **AGNT-05**: Methodology Critic agent that reviews experiment designs for methodological flaws, with `memory: project`
- [ ] **AGNT-06**: Orchestration pipeline skill that sequences agent invocations with explicit artifact handoffs, preventing Bag-of-Agents failure
- [ ] **AGNT-07**: `context: fork` isolation per pipeline phase to prevent cross-phase context contamination

### Literature Discovery

- [ ] **LIT-01**: Paper search via Semantic Scholar MCP server integration (structured metadata, field-of-study data)
- [ ] **LIT-02**: Paper search via Brave Search MCP for preprints and recent blog posts
- [ ] **LIT-03**: PDF/paper ingestion and analysis via `@` file references
- [ ] **LIT-04**: Literature summary generation with grounded citations (arXiv IDs required, no hallucinated references)
- [ ] **LIT-05**: Citation network traversal to discover related papers via citation graphs
- [ ] **LIT-06**: Research gap identification from analyzed literature

### Topic Selection

- [ ] **TOPC-01**: Research direction generation based on literature gap analysis
- [ ] **TOPC-02**: Topic shortlist with structured comparison (novelty, feasibility, benchmark availability, academic contribution)
- [ ] **TOPC-03**: Feasibility scoring incorporating capstone constraints (one-person scope, timeline, Python comfort, benchmark evaluability)

### Experiment Design

- [ ] **EXPR-01**: Benchmark selection guidance across SWE-bench family (Verified, Pro, EVO), HumanEval, FeatureBench with current performance data
- [ ] **EXPR-02**: Baseline identification from existing published work
- [ ] **EXPR-03**: Embedded knowledge about benchmark saturation (SWE-bench Verified ~77%, SWE-EVO ~21%, data leakage risks)
- [ ] **EXPR-04**: Experiment plan template generation tailored to specific benchmark families

### Implementation Support

- [ ] **IMPL-01**: System architecture design for research prototypes
- [ ] **IMPL-02**: Code scaffolding for experiment implementations in Python

### Paper Writing

- [ ] **PAPR-01**: Section-by-section draft generation (abstract, introduction, related work, methodology, results, discussion, conclusion)
- [ ] **PAPR-02**: Results-section gate requiring `experiment_results.md` artifact before generating results/discussion sections
- [ ] **PAPR-03**: Related work section grounded in previously analyzed literature (references back to literature_review.md artifact)

## v2 Requirements

Deferred to future release. Tracked but not in current roadmap.

### Enhanced Discovery

- **DISC-01**: Automated related work comparison matrices
- **DISC-02**: Trend detection across paper publication dates
- **DISC-03**: Author network analysis (who collaborates with whom in the field)

### Pipeline Automation

- **AUTO-01**: Auto-advance between pipeline stages in YOLO mode
- **AUTO-02**: Hook-based quality gates at phase transitions (SubagentStop hooks)
- **AUTO-03**: Data analyst agent for benchmark result analysis, tables/figures, statistical significance

### Collaboration

- **COLB-01**: Scope guardian agent that monitors for capstone scope creep
- **COLB-02**: Export artifacts to LaTeX format
- **COLB-03**: Integration with Zotero for bibliography management

## Out of Scope

| Feature | Reason |
|---------|--------|
| General-purpose research across arbitrary domains | Capstone-focused; domain knowledge is deeply embedded, not configurable |
| Real-time paper tracking/alerting | On-demand search is sufficient; alerting adds complexity without capstone value |
| Automated benchmark execution | Plugin designs experiments; user runs them. Execution requires hardware/environment setup |
| Citation management / bibliography formatting | Use Zotero or similar — better tools exist |
| Peer review simulation | Methodology Critic covers design flaws; simulating anonymous reviewers is unreliable |
| Web scraping of paywalled papers | Legal/copyright issues; use official APIs (Semantic Scholar, arXiv) |
| Plagiarism detection | External tools (Turnitin, iThenticate) are authoritative |
| Journal/venue matching | Out of scope for capstone; advisor provides this guidance |

## Traceability

Which phases cover which requirements. Updated during roadmap creation.

| Requirement | Phase | Status |
|-------------|-------|--------|
| FOUND-01 | Phase 1 | Pending |
| FOUND-02 | Phase 1 | Pending |
| FOUND-03 | Phase 1 | Pending |
| FOUND-04 | Phase 1 | Pending |
| FOUND-05 | Phase 1 | Pending |
| AGNT-01 | Phase 2 | Pending |
| AGNT-02 | Phase 3 | Pending |
| AGNT-03 | Phase 4 | Pending |
| AGNT-04 | Phase 5 | Pending |
| AGNT-05 | Phase 3 | Pending |
| AGNT-06 | Phase 3 | Pending |
| AGNT-07 | Phase 3 | Pending |
| LIT-01 | Phase 2 | Pending |
| LIT-02 | Phase 2 | Pending |
| LIT-03 | Phase 2 | Pending |
| LIT-04 | Phase 2 | Pending |
| LIT-05 | Phase 2 | Pending |
| LIT-06 | Phase 2 | Pending |
| TOPC-01 | Phase 2 | Pending |
| TOPC-02 | Phase 2 | Pending |
| TOPC-03 | Phase 2 | Pending |
| EXPR-01 | Phase 3 | Pending |
| EXPR-02 | Phase 3 | Pending |
| EXPR-03 | Phase 3 | Pending |
| EXPR-04 | Phase 3 | Pending |
| IMPL-01 | Phase 4 | Pending |
| IMPL-02 | Phase 4 | Pending |
| PAPR-01 | Phase 5 | Pending |
| PAPR-02 | Phase 5 | Pending |
| PAPR-03 | Phase 5 | Pending |

**Coverage:**
- v1 requirements: 30 total
- Mapped to phases: 30
- Unmapped: 0

---
*Requirements defined: 2026-03-15*
*Last updated: 2026-03-15 after roadmap creation — all 30 requirements mapped*
