## 4. Staged Validation Protocol (MC-only Studies)

FCC-ee analyses in this repository run on simulation only (Delphes fast
sim and/or DD4hep + Geant4 full sim via Key4hep). There is no real data
to blind. The 4a → 4b → 4c staging is preserved — what changes is the
*meaning* of each stage.

Two orthogonal axes of staging apply to every analysis:

1. **Statistical staging** (§4.1, §4.2): Asimov → toys → full-stats
   expected result. This replaces the LHC unblinding protocol.
2. **Dual-chain execution** (§4.3): the same analysis is run on both
   Delphes fast sim and CLD/IDEA full sim, and the final results are
   compared. **This is a primary goal of the framework, not an
   appendix.** Every Phase 3 / Phase 4 artifact is chain-stamped, and
   the Phase 5 AN has a mandatory dual-chain comparison chapter.

### 4.1 Stages

| Stage | Inputs | Gate |
|-------|--------|------|
| Phases 1–3 | Full nominal MC; selection, corrections, classifier training | — |
| Phase 4a | Asimov pseudo-data from nominal MC | 4-bot+bib review (§6.2) |
| Phase 4b | Toy-MC coverage scan (N ≥ 500 Poisson toys, fixed master seed) | 4-bot+bib review → human gate |
| Phase 4c | Full-statistics expected result at design luminosity (10.8 ab⁻¹ at 240 GeV, 3.12 ab⁻¹ at 365 GeV, or the analysis-appropriate value) | 1-bot review |

**Asimov pseudo-data** = synthetic pseudo-data from the nominal model,
bin contents at exact expected values (no fluctuations). This is the
"data" of Phase 4a. The expected uncertainty on the fitted parameter is
read off the Asimov fit.

**Toy-MC coverage scan (Phase 4b)** — the MC-only equivalent of the
10% unblinding gate. Generate N Poisson toys from the nominal model,
refit each, and check that:
1. Pull distributions are unit Gaussian (mean ~0, width ~1 within toy
   statistical error). Deviations indicate biased fits or
   under/over-coverage.
2. The fitted parameter distribution has mean ≈ injected truth (within
   1 toy-stat uncertainty) — the fit is unbiased.
3. Nuisance parameter pulls are centred on zero; no systematic is
   constrained away from its prior width without an explanation.
4. Goodness-of-fit on each toy is consistent with the observed chi²
   distribution from the full-stats Asimov — the model describes the data
   it was built from.

Fix problems detected in the toy scan *before* quoting the full-stats
result.

**Full-stats expected result (Phase 4c)** — run the analysis at full
design luminosity, produce the final covariance matrix, produce the
final AN draft with numerical results, compare to the target reference
(arXiv:2512.21290 for ZH recoil; published FCC-ee projections for
other analyses). This replaces the "full real data" phase.

### 4.2 Human gate

The human gate after Phase 4b is preserved but its content changes.
After the 4b review panel PASSes, present to the human:

- Compiled PDF of the draft AN with Phase 4b toy results
- Toy coverage summary table: mean pull, pull width, bias, chi² distribution
- Nuisance parameter pull summary (median, 68% interval per NP)
- Checklist:
  1. Systematic programme complete against `conventions/fcc_ee.md` and any technique file
  2. Fit converges on every toy; no toys hit parameter boundaries
  3. Pull distributions consistent with unit Gaussian
  4. Signal injection / closure tests pass
  5. All reviewer A/B items resolved (arbiter PASS)
  6. Draft AN reviewed and publication-ready modulo Phase 4c numerical update

The human approves, requests changes, or halts. The agent does not
advance to Phase 4c autonomously.

### 4.3 Dual-chain execution (fast sim + full sim)

**The primary goal of this framework is to reproduce the same physics
analysis on both simulation chains — Delphes fast sim and CLD/IDEA full
sim — and to compare the final results.** Dual-chain execution is not
optional, and it is not a post-hoc appendix: it is the central deliverable.
Every analysis here ends with two independent physics results (one per
chain) and a formal comparison between them.

This section defines the dual-chain execution protocol. Individual phase
requirements are in `03-phases.md`; the chain-portable code pattern is in
`conventions/fcc_ee.md` ("Collection aliasing"); the final-result
comparison template lives in `conventions/fcc_ee.md` ("Dual-chain
comparison").

#### 4.3.1 Why both chains

Delphes (fast sim) is cheap and available centrally (winter2023 IDEA
production on EOS); it is the default primary chain when no CLD
production is ready. CLD/IDEA full sim (DD4hep + Geant4 + Pandora) is
the reference detector simulation for the FCC-ee feasibility study but
is user-produced and compute-heavy. Running the same analysis on both
chains is what lets the collaboration:

1. Quantify the detector-simulation effect on the final observable
   (fitted parameter, uncertainty budget, covariance structure) —
   **not** just on input distributions.
2. Validate that the analysis design (selection, correction chain,
   fit model) survives the transition from a parameterised response to
   full tracking + Pandora PFA.
3. Identify which systematic sources are absorbed by fast-sim
   parameterisations and only exposed by full sim (tracking tails,
   PFA mis-assignment, PID edge effects).

An analysis that runs on only one chain without planning for the other
is incomplete by this framework's definition. If only one chain is
available at the time of the analysis, the strategy must state this
explicitly, and Phase 5 must schedule the second chain as a concrete
follow-up with a documented trigger (e.g., "when CLD production of
wzp6_ee_mumuH_ecm240 completes").

#### 4.3.2 Execution pattern

Dual-chain execution is achieved by keeping the physics graph
(`build_graph`) chain-independent and isolating the chain differences in
a small aliasing shim. The shim is defined in
`conventions/fcc_ee.md` ("Collection aliasing"). Once the shim is in
place, the same Phase 3 histmaker, Phase 4 statistical model, and
Phase 5 AN production run on both chains with only the
`inputDir` / `prodTag` and `sim_chain` arguments changing.

**One primary chain, one secondary chain.** The orchestrator designates
one chain as primary (the chain the analysis is first executed and
reviewed on) and one as secondary. The primary chain is typically
Delphes (because samples are centrally produced and readily accessible);
CLD is typically secondary because samples must be produced locally.
The primary chain passes through the full phase sequence first; the
secondary chain re-runs Phases 3 and 4 only (strategy and exploration
are shared).

**The secondary chain is not a review-free side study.** It produces its
own Phase 3 artifact (selection, closure tests) and its own Phase 4a/4b/4c
artifacts (systematics, Asimov fit, toy scan, full-stats result). Each
secondary-chain artifact receives the same review tier as the primary
chain — 1-bot at Phase 3, 4-bot+bib at 4a/4b, 1-bot at 4c. Skipping
reviews on the secondary chain is a process failure.

#### 4.3.3 Phase-by-phase dual-chain map

| Phase | Scope on primary chain | Scope on secondary chain |
|-------|------------------------|--------------------------|
| 1 Strategy | Full strategy, both chains planned explicitly | Shared — no separate strategy artifact |
| 2 Exploration | `podio-dump` + variable survey on both chain's samples | Same artifact, separate sample inventory subsection per chain |
| 3 Processing | Full selection + correction chain | Re-run with secondary `inputDir` / `sim_chain="full"`; produce `SELECTION_<chain>.md` |
| 4a Expected | Systematics + Asimov at full lumi | Re-run with secondary histograms; produce `INFERENCE_EXPECTED_<chain>.md` + AN_v1 per chain |
| 4b Toy scan | ≥500 Poisson toys | Same, secondary chain |
| 4c Full-stats | Final covariance + fitted parameter | Same, secondary chain |
| 5 Documentation | Single AN covering both chains + comparison section | — |

The strategy and exploration artifacts are shared across chains (the
physics graph and sample inventory are the same up to the aliasing
shim). From Phase 3 onwards every artifact is chain-stamped:
`SELECTION_delphes.md`, `SELECTION_cld.md`, `INFERENCE_EXPECTED_delphes.md`,
`INFERENCE_EXPECTED_cld.md`, etc. The Phase 5 AN is a single document
with a mandatory "Dual-chain comparison" chapter.

#### 4.3.4 Scheduling: serial vs. parallel

The orchestrator may run the two chains serially (primary through all
phases, then secondary through Phases 3–4) or in parallel (both chains
through each phase before advancing). The choice is operational, not
methodological:

- **Serial (default when secondary samples are not ready).** Delphes
  through Phases 1–5 first. When CLD samples become available, rerun
  Phases 3 and 4 on CLD and fold the results into the Phase 5 AN via
  a regression-style update (new AN version with Change Log entry).
- **Parallel (default when both chain's samples are ready from the start).**
  Each phase executes on both chains (possibly in parallel subagents);
  review panels cover both chains' artifacts in the same pass.

In both cases the final deliverable is the same: one AN with both
chains' results and the dual-chain comparison chapter.

#### 4.3.5 Dual-chain comparison (Phase 4c deliverable)

At Phase 4c, after both chains' full-stats fits are complete, the
executor produces a **dual-chain comparison artifact** —
`COMPARISON_dual_sim.md` in `phase4_inference/4c_fullstats/outputs/` —
that compares the two chains at the level of the final physics result,
not just at the level of input distributions. Mandatory contents:

1. **Fitted-parameter table.** Primary-chain central value and
   uncertainty vs. secondary-chain central value and uncertainty, with
   their difference and pull (difference divided by combined
   uncertainty).
2. **Systematic budget side-by-side.** Per-source relative
   uncertainty on the fitted parameter for each chain, with a delta
   column. Sources that are dominant on one chain but subdominant on
   the other are flagged.
3. **Covariance matrix comparison.** For unfolded / differential
   measurements, the bin-to-bin covariance from each chain and a
   per-element relative difference (or a Frobenius-norm distance with
   interpretation).
4. **Distribution overlays with ratio panels.** The observable and
   every MVA input / correction input, overlaid between the two chains
   with a Delphes/CLD ratio panel. This is the "classic" overlay but
   is now one row of the comparison, not the whole thing.
5. **Verdict.** Are the chains consistent within the expected
   detector-simulation effect? Quote a chi² or combined-pull summary.
   State whether any observed disagreement is physically expected
   (e.g., Pandora PFA vs. Delphes parameterised PF) or points to an
   analysis-chain issue that must be investigated.

The comparison artifact is reviewed as part of Phase 4c (1-bot); the
Phase 5 AN inherits this comparison as a chapter.

#### 4.3.6 When a chain is genuinely unavailable

If CLD (or Delphes) samples are genuinely unavailable at the time of
the analysis — not "hard to produce" but actually blocked by a missing
upstream deliverable (no generator sample, no reconstruction release) —
the strategy documents this as a formal constraint [A] and the Phase 5
AN includes a "Single-chain disclaimer" section naming:

- The chain that was run,
- The chain that was not run and why,
- The expected trigger for completing the dual-chain comparison,
- The person/team responsible for following up.

This is the only acceptable single-chain outcome, and it is a
downscope that must be approved at the Phase 1 review, not a default.

See `conventions/fcc_ee.md` → "Dual-chain comparison" for the technical
pattern (shim, run-script structure, overlay script).

---
