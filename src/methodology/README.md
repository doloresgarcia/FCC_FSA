# Methodology Specification

This repository is a retarget of the JFC framework for **FCC-ee analyses
on simulation only** (Delphes fast sim AND CLD/IDEA full sim via
Key4hep). The phase structure, review tiers, and artifact contract from
JFC are preserved. The LHC-specific data-vs-MC language has been
replaced with an MC-only staged-validation protocol and FCC-ee-specific
conventions. Read `src/conventions/fcc_ee.md` first — it contains the
Key4hep / EDM4hep glossary, FCCAnalyses histmaker pattern, and
Delphes↔CLD collection aliasing shim that every analysis in this
repository needs.

**Dual-chain execution is a primary goal.** Every analysis here is
reproduced on both Delphes fast sim and CLD full sim, and the final
physics results (fitted parameter, uncertainty budget, covariance) are
compared at Phase 4c. From Phase 3 onwards artifacts are chain-stamped
(`SELECTION_delphes.md`, `INFERENCE_FULLSTATS_cld.md`, …); Phase 4c
additionally produces `COMPARISON_dual_sim.md`; Phase 5 folds the
comparison into the AN as a mandatory chapter. The execution protocol
is in `04-staged-validation.md` §4.3.

## Structure

The spec is organized into three tiers:

### Tier 1: Core physics analysis — "what to do"
- `03-phases.md` — Phase 1–5 definitions (requirements, deliverables, gates)
- `04-staged-validation.md` — Asimov + toy-MC staged validation protocol (MC-only) **and dual-chain execution protocol (§4.3)**
- `09-multichannel.md` — Multi-channel analysis guidance

### Tier 2: Process and scaffolding — "how to manage it"
- `01-principles.md` — Scope and design principles
- `02-inputs.md` — Physics prompt, RAG retrieval
- `03a-orchestration.md` — Orchestrator loop, subagent management, context, scaling
- `05-artifacts.md` — Artifact format and experiment log
- `06-review.md` — Review protocol (classification, iteration, per-phase focus)
- `12-downscoping.md` — Scope management and feasibility

### Tier 3: Craft — "how to write good code, notes, and plots"
- `07-tools.md` — Tool preferences, paradigms, scale-out patterns
- `11-coding.md` — Git, code quality, testing, pixi tasks
- `appendix-plotting.md` — Figure template, sizing, labels, styling
- `appendix-heuristics.md` — Tool idioms (agent-maintained)

### Appendices — reference material
- `analysis-note.md` — Analysis note specification (required sections, depth, bibliography)
- `appendix-dependencies.md` — Phase dependency graph
- `appendix-checklist.md` — Per-phase artifact checklists

### Appendices — operational (agent execution)
- `appendix-prompts.md` — Literal prompt templates for each agent role
- `appendix-automation.md` — Orchestration pseudocode (review tiers, pipeline flow)
- `appendix-sessions.md` — Session naming, directory layout, isolation model
- `appendix-integration.md` — RAG/MCP setup, Claude Code team mapping (platform-specific)

## Reading guide

- **For a new analysis:** Read `src/conventions/fcc_ee.md` first for the
  FCC-ee / Key4hep / EDM4hep domain (including the "Dual-chain
  comparison" section), then Tier 1 — `03-phases.md` plus
  `04-staged-validation.md` (staging protocol **and** dual-chain
  execution in §4.3) — to understand what each phase does and on
  which chain.
- **For orchestration:** Read §3a (including §3a.6 on dual-chain
  coordination), then appendix-prompts/automation/sessions for
  operational details
- **For coding:** Read Tier 3 for tool choices, code style, and plotting rules
- **Templates** (`src/templates/`) are thin entry points pointing to these files
