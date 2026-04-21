# FCC_FSA — FCC-ee Full-Sim / Fast-Sim Analysis Framework

`FCC_FSA` is a retarget of the [JFC](https://github.com/doloresgarcia/FCC_FSA/tree/main) (Just Furnish Context)
multi-agent HEP analysis framework for **FCC-ee simulation-only analyses**.
The phased executor/reviewer architecture, experiment log, and artifact
contract from JFC are preserved; the LHC-specific data-vs-MC assumptions
are replaced with an FCC-ee / Key4hep / EDM4hep domain layer and an
MC-only staged validation protocol.

The primary target analysis implemented in this repository is the
**model-independent ZH production cross-section measurement at FCC-ee**
using the recoil-mass technique, following arXiv:2512.21290.

**A core goal of this framework is dual-chain execution.** Every
analysis is reproduced end-to-end on both simulation chains —
Delphes winter2023 IDEA fast sim and CLD/IDEA full sim produced via
the Key4hep tutorial — and the final physics results (fitted parameter,
uncertainty budget, covariance) are compared. A single-chain analysis
is an incomplete deliverable in this framework. The chain-portable code
pattern (one physics graph + an aliasing shim) keeps the two passes
code-free-of-duplication; see
`src/methodology/04-staged-validation.md` §4.3 for the dual-chain
execution protocol and `src/conventions/fcc_ee.md` "Dual-chain
comparison" for the technical pattern.

## Framework

| Component | Location | Role |
|-----------|----------|------|
| Methodology | `src/methodology/` | Phase 1–5 definitions, orchestration, staged validation, review, plotting, analysis-note spec |
| Conventions | `src/conventions/` | Domain knowledge — `fcc_ee.md` for Key4hep/EDM4hep/FCCAnalyses, plus technique files (`extraction.md`, `unfolding.md`, `search.md`) |
| Agents | `src/agents/` | Executor and reviewer role definitions (physics, critical, constructive, plot validator, note writer, typesetter, bibtex, arbiter, investigator, fixer) |
| Templates | `src/templates/` | `root_claude.md` and `phase{1..5}_claude.md` + `pixi.toml` stamped into each new analysis directory |
| Scaffolder | `src/scaffold_analysis.py` | `pixi run scaffold analyses/<name> --type measurement` |

## Quick start

```bash
# 1. Source Key4hep from cvmfs (needed for fccanalysis, k4run, ddsim, podio-dump)
source /cvmfs/sw.hsf.org/key4hep/setup.sh

# 2. Scaffold a new analysis — e.g. the ZH recoil one this repo targets
pixi run scaffold analyses/zh_recoil_mumu --type measurement
cd analyses/zh_recoil_mumu

# 3. Edit .analysis_config: set data_dir to the FCCAnalyses sample catalog
#    (use prodTag="FCCee/winter2023/IDEA/" from lxplus / AFS)
pixi install

# 4. Launch the orchestrator with your physics prompt
claude
```

See `src/conventions/fcc_ee.md` for FCC-ee-specific conventions (sample
discovery, collection names, Delphes↔CLD shim, plot style).

## Phased workflow

```
┌─────────────────────────────────────────────────────────────┐
│                     ORCHESTRATOR                             │
│  Never writes code. Holds: prompt, summaries, verdicts only  │
└─────┬───────────────────────────────────────────────────────┘
      │
      ▼                          ┌── runs on both chains ──┐
 ┌──────────┐  ┌──────────┐     ▼                          ▼
 │ Phase 1  │─▶│ Phase 2  │─▶ Phase 3 ─▶ Phase 4a ─▶ Phase 4b ─▶ Phase 4c ─▶ Phase 5
 │ Strategy │  │ Explore  │   (1-bot)   (4bot+bib)  (4bot+bib)   (1-bot)    (5-bot)
 │ (4-bot)  │  │(self+plt)│                                    + dual-chain  + AN
 │          │  │          │                                    comparison    with
 │ both     │  │ both     │                                                  both
 │ chains   │  │ chains'  │                                                  chains'
 │ planned  │  │ samples  │                                                  results
 └──────────┘  └──────────┘                                    │
                                                               ▼
                                                         HUMAN GATE (after 4b)
```

Strategy (Phase 1) and exploration (Phase 2) are shared across chains —
one artifact covers both the Delphes and CLD samples. From Phase 3
onwards every artifact is chain-stamped: `SELECTION_delphes.md`,
`SELECTION_cld.md`, `INFERENCE_EXPECTED_delphes.md`, etc. Phase 4c
additionally produces `COMPARISON_dual_sim.md` comparing the two chains
at the level of the final fitted parameter, its uncertainty budget,
and the covariance matrix. Phase 5 is a single AN that includes both
chains' results and a mandatory "Dual-chain comparison" chapter.

Each phase runs the same loop:

```
  1. EXECUTE ── spawn executor subagent (enters plan mode first)
  2. REVIEW ─── spawn reviewer(s) per review type
  3. CHECK:
       Regression trigger? → Investigator → fix origin + downstream → resume
       A or B items?       → fix agent + fresh reviewer → re-review (loop)
       Only C items?       → PASS, executor applies Cs before commit
  4. COMMIT
  5. HUMAN GATE (after 4b)
  6. ADVANCE
```

### Phases

| Phase | Review | Key deliverable |
|-------|--------|-----------------|
| **1. Strategy** | 4-bot | Observable, sample inventory, selection approaches, systematic plan, reference analysis table (e.g. arXiv:2512.21290 for ZH recoil) |
| **2. Exploration** | Self + plot validator | `podio-dump` schema of each sample, kinematic coverage, preselection cutflow |
| **3. Processing** | 1-bot | FCCAnalyses histmaker (`build_graph`), event selection, correction chain, closure tests |
| **4a. Expected (Asimov)** | 4-bot+bib | Systematic completeness table, binned likelihood, Asimov fit, covariance matrix, reference comparison |
| **4b. Toy-MC coverage** | 4-bot+bib → human gate | ≥500 Poisson toys, pull/coverage plots, draft AN with toy results |
| **4c. Full-stats expected** | 1-bot | Full design-luminosity expected result, final covariance, post-fit diagnostics |
| **5. Documentation** | 5-bot | Analysis note (pandoc markdown → PDF, 50-100 pages), machine-readable results |

There is no real data. Phases 4a/4b/4c all use simulation; the staging
is a coverage protocol (Asimov → toys → full-stats), not an unblinding
protocol. See `src/methodology/04-staged-validation.md`.

### Review classification

| Cat | Meaning | Action |
|-----|---------|--------|
| **A** | Would cause rejection | Fix + re-review + fresh reviewer |
| **B** | Weakens the analysis | Same — must be zero before PASS |
| **C** | Style / clarity | Arbiter PASses; executor applies before commit |

## Dual-chain execution (primary goal)

**Every analysis runs end-to-end on both simulation chains.** The
orchestrator designates one chain as primary (typically Delphes, since
the winter2023 IDEA samples are centrally produced and on EOS) and one
as secondary (typically CLD, produced locally via the Key4hep tutorial).
The primary chain passes through Phases 1–5; the secondary chain
re-runs Phases 3–4 (strategy and exploration are shared). Each
chain-stamped artifact receives the same review tier as the primary:
1-bot at Phase 3, 4-bot+bib at 4a/4b, 1-bot at 4c.

At Phase 4c the executor produces `COMPARISON_dual_sim.md` comparing
the two chains at the level of the final physics result — fitted
parameter with uncertainty, per-source systematic budget, bin-to-bin
covariance, and distribution overlays with ratio panels. Phase 5
folds this comparison into the AN as a mandatory chapter.

If one chain is genuinely unavailable (missing upstream production, no
reconstruction release), this is documented as a formal constraint at
Phase 1 and the AN carries a "Single-chain disclaimer" section naming
the missing chain and the trigger for completing the comparison. This
is the only acceptable single-chain outcome.

See `src/methodology/04-staged-validation.md` §4.3 for the full
execution protocol (primary/secondary designation, scheduling,
comparison artifact contents, review tiers) and `src/conventions/fcc_ee.md`
→ "Collection aliasing" and "Dual-chain comparison" for the
chain-portable code pattern.

## Key concepts

**FCC-ee / Key4hep domain.** `src/conventions/fcc_ee.md` is the anchor
document: EDM4hep collection glossary, FCCAnalyses histmaker pattern,
sample discovery via `prodTag` / `inputDir`, Delphes↔CLD aliasing, plot
style with mplhep. Every analysis reads it at Phase 1.

**Technique decided at Phase 1.** The scaffolder only takes
`--type measurement|search`. The strategy phase selects the technique
(template fit, unfolding, extraction), which activates technique-specific
conventions in later phases.

**Simulation only.** There is no real data. Asimov pseudo-data at 4a,
Poisson toys at 4b, full-stats expected result at 4c. All on MC.

**Dual-chain by design.** Every analysis is reproduced on both Delphes
fast sim and CLD full sim, and the final fitted parameter, uncertainty
budget, and covariance are compared at Phase 4c. The chain-portable
code pattern (shared `build_graph` + aliasing shim) keeps the two
passes code-free-of-duplication. See
`src/methodology/04-staged-validation.md` §4.3.

**Two environments.** FCCAnalyses / Key4hep / EDM4hep come from
`/cvmfs/sw.hsf.org/key4hep/setup.sh`. Python-only post-processing
(matplotlib, mplhep, uproot, hist, pyhf, pandoc) comes from the local
pixi environment. Pixi tasks that need both wrap the cvmfs source in a
`bash -lc` string.

## Directory structure

```
FCC_FSA/
  src/                        Framework infrastructure
    methodology/              Phases, orchestration, staged validation, review, plotting, AN spec
    conventions/              Domain knowledge: fcc_ee.md + technique files
    agents/                   Executor and reviewer role definitions
    templates/                CLAUDE.md and pixi.toml templates
    scaffold_analysis.py      Scaffolder
  analyses/                   Each analysis is its own git repo
    zh_recoil_mumu/           Reference analysis: model-independent ZH at FCC-ee (arXiv:2512.21290)
      CLAUDE.md               Self-contained instructions for the orchestrator
      pixi.toml               Environment + task graph
      .analysis_config        data_dir + allow paths
      conventions/ → src/conventions/
      phase{1..5}_*/          Phase dirs with CLAUDE.md, outputs/, src/, review/, logs/
                              Phase 3+ artifacts are chain-stamped (_delphes / _cld)
                              Phase 4c adds COMPARISON_dual_sim.md (final-result comparison)
```

## Requirements

- [Key4hep](https://key4hep.github.io/key4hep-doc/) via cvmfs for `fccanalysis`, `k4run`, `ddsim`, `podio-dump`, EDM4hep
- [pixi](https://pixi.sh) for the local Python-only environment
- [Claude Code](https://claude.ai/claude-code) as the agent runtime
- CERN AFS or lxplus access for the Delphes winter2023 IDEA samples via
  `prodTag = "FCCee/winter2023/IDEA/"`

## References

- FCC-ee feasibility study and the Key4hep project
- FCC full-sim tutorial: <https://hep-fcc.github.io/fcc-tutorials/main/full-detector-simulations/FCCeeGeneralOverview/FCCeeGeneralOverview.html>
- FCCAnalyses: <https://github.com/HEP-FCC/FCCAnalyses>
- FCCPhysics example analyses (Jeyserma): <https://github.com/jeyserma/FCCPhysics>
- Model-independent ZH cross section at FCC-ee: arXiv:2512.21290
- Upstream JFC framework:
  > *AI Agents Can Already Autonomously Perform Experimental High Energy Physics*
  > E. A. Moreno, S. Bright-Thonney, A. Novak, D. Garcia, P. Harris
