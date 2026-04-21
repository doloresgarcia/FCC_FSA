# Phase 4: Inference

> Read `methodology/03-phases.md` → "Phase 4" for full requirements.
> Read `methodology/appendix-plotting.md` for figure standards.
> Read `methodology/04-staged-validation.md` for the MC-only staging protocol.
> Read `conventions/fcc_ee.md` for FCC-ee sample and statistical conventions.

You are building the statistical model and computing results for a
**{{analysis_type}}** analysis.

**Start in plan mode.** Before writing any code, produce a plan: what
systematics you will evaluate, what validation checks you will run, what
the artifact structure will be. Execute after the plan is set.

## Output artifacts and flow

**Both measurements and searches follow the same 4a → 4b → 4c structure.
All inputs are simulation — Delphes or CLD EDM4hep samples. There is no
real data; "pseudo-data" below means Asimov or Poisson-toy pseudo-data
generated from the nominal MC model.**

- **4a:** Statistical analysis — systematics, expected results on
  Asimov pseudo-data at full design luminosity. Executor (stats) → note
  writer (AN v1 with expected results) → typesetter (compile). Review
  includes BibTeX validation.
- **4b:** Toy-MC coverage scan. Generate ≥500 Poisson toys from the
  nominal model at full luminosity, refit each, produce pull and
  coverage plots. Executor (stats) → note writer (update AN with toy
  results) → typesetter (compile for human gate). Review includes
  BibTeX validation. Human gate after review.
- **4c:** Full-statistics expected result + final covariance for
  each chain. Compare to both the 4b toy distribution and the 4a
  Asimov. After both chain's fits are complete, produce
  `COMPARISON_dual_sim.md` comparing the two chains at the level of
  the final fitted parameter, per-source systematic budget, and
  covariance matrix (see "Dual-chain comparison" below). Executor
  (stats) → note writer (update AN with final numbers for both chains
  + dual-chain chapter inheriting COMPARISON_dual_sim.md).

**Note writer figure composition annotations.** When the note writer
references groups of related figures (per-variable data/MC, per-systematic
shifts, nominal + uncertainty pairs), it MUST annotate the grouping with
`<!-- COMPOSE: NxM grid -->`, `<!-- COMPOSE: side-by-side -->`, or
`<!-- FLAGSHIP -->` comments in the markdown. These annotations tell the
typesetter what to merge, persist across AN versions, and save the
typesetter from re-discovering groupings at each compilation. See
`methodology/03-phases.md` → Phase 5 "Figure composition annotations."

**Typesetting at 4a/4b/4c.** Spawn the typesetter agent (read
`agents/typesetter.md` for the full prompt). The typesetter handles the
entire pipeline: pandoc → postprocess_tex.py → read composition
annotations → figure combining → compile → verify. Provide the
phase-stamped filename (e.g., `ANALYSIS_NOTE_4a_v1.md`). The 4a/4b/4c
PDFs are review inputs — they must meet the same formatting standard as
the final Phase 5 PDF.

| Sub-phase | Artifact | Review |
|-----------|----------|--------|
| 4a | `INFERENCE_EXPECTED_delphes.md` AND `INFERENCE_EXPECTED_cld.md` + `ANALYSIS_NOTE_4a_v1.{md,tex,pdf}` | 4-bot+bib |
| 4b | `INFERENCE_TOYS_delphes.md` AND `INFERENCE_TOYS_cld.md` + `ANALYSIS_NOTE_4b_v1.{md,tex,pdf}` | 4-bot+bib → human gate |
| 4c | `INFERENCE_FULLSTATS_delphes.md` AND `INFERENCE_FULLSTATS_cld.md` + `COMPARISON_dual_sim.md` + `ANALYSIS_NOTE_4c_v1.{md,tex,pdf}` | 1-bot |

**Dual-chain execution (mandatory).** Phase 4 runs on both chains.
Every chain-stamped artifact receives the same review tier as the
primary chain — skipping reviews on the secondary chain is a process
failure. See `methodology/04-staged-validation.md` §4.3 for the full
protocol (primary/secondary designation, scheduling, comparison
artifact contents).

## Physics correctness gates (mandatory self-checks before review)

These checks promote critical rules from the methodology spec. They are
Category A at review — verify before submitting.

1. **Full covariance chi2.** Every chi2 value in the artifact and AN uses
   the full covariance matrix (not diagonal only). If the covariance matrix
   is available, report chi2(full) as the primary metric. See
   `methodology/analysis-note.md` → Statistical methodology standards.

2. **Systematic variation sizing.** Every systematic variation is justified
   by a measurement or published uncertainty. No arbitrary "±50%"
   variations without documented motivation. See `methodology/03-phases.md`
   → Systematic Variation Sizing.

3. **Extraction method hierarchy.** If a corrected differential distribution
   AND its covariance matrix are available, the differential fit is the
   primary extraction method. Mean-value extraction is a cross-check. See
   `methodology/03-phases.md` → Extraction Method Hierarchy.

4. **Independent closure.** Closure tests use an MC sample statistically
   independent from the one used to derive corrections. Self-consistent
   closure (same sample) is an algebra check, not a validation. See
   `conventions/extraction.md` → Required Validation Checks.

5. **Per-systematic impact figures.** Every systematic source has a figure
   showing how it shifts the result bin-by-bin. A summary table alone is
   insufficient. See `methodology/analysis-note.md` → Systematic
   uncertainties.

6. **Resolving power statement.** After every result, state what deviations
   the measurement can detect at 2-sigma. "Total uncertainty of 3.8% →
   can distinguish predictions differing by ~8% at 2σ." See
   `methodology/analysis-note.md` → Interpretive quality standards.

## Dual-chain comparison (Phase 4c, mandatory)

After both Delphes and CLD full-stats fits are complete, produce
`outputs/COMPARISON_dual_sim.md`. Mandatory contents
(§4.3.5 of `methodology/04-staged-validation.md`):

1. **Fitted-parameter table** — Delphes central value ± uncertainty
   vs. CLD central value ± uncertainty, difference, and pull.
2. **Side-by-side systematic budget** — per-source relative uncertainty
   on the fitted parameter for each chain, with a delta column. Flag
   any source that is dominant on one chain but subdominant on the
   other.
3. **Covariance matrix comparison** (unfolded / differential only) —
   bin-to-bin covariance from each chain + per-element relative
   difference or Frobenius-norm distance.
4. **Distribution overlays with ratio panels** — the observable and
   every MVA input / correction input, overlaid between chains.
5. **Verdict** — chi² or combined-pull summary, physical interpretation
   (physically-expected detector-simulation effect vs. chain-specific
   analysis issue).

A dual-chain disagreement **larger than the assigned detector-simulation
systematic** is a Phase 3 regression trigger. Do not fold it into a
flat systematic.

The comparison artifact is reviewed alongside the chain-stamped 4c
artifacts (1-bot) and inherited by the Phase 5 AN as a chapter.

## Human gate (after 4b review)

After the 4b review panel returns PASS, present the **compiled PDF** and
the toy-MC coverage checklist (from `methodology/04-staged-validation.md`
§4.2) to the human. Do NOT proceed to 4c without explicit human approval.

The human may:
- **APPROVE** — proceed to 4c (full data)
- **ITERATE** — fix issues within 4b scope, re-review, re-present
- **REGRESS(N)** — fundamental issue traced to Phase N; follow the
  non-destructive regression protocol in `methodology/06-review.md` §6.6
- **PAUSE** — wait for external input

On regression: the Investigator assesses cascade scope, the executor
creates new artifact versions (not overwrites), downstream phases
re-evaluate, and the note writer produces a new AN version with a
cohesive narrative. Re-present the updated PDF to the human.

## Methodology references

- Phase requirements: `methodology/03-phases.md` → Phase 4
- Technique-specific requirements: `methodology/03-phases.md` → Phase 4 sub-phase descriptions
- Staged validation (MC-only): `methodology/04-staged-validation.md`
- FCC-ee / EDM4hep / FCCAnalyses domain: `conventions/fcc_ee.md`
- Review protocol: `methodology/06-review.md` → §6.2 (4-bot / 1-bot), §6.4
- Goodness-of-fit: `methodology/03-phases.md` → Phase 4 GoF requirements
- Plotting: `methodology/appendix-plotting.md`

## RAG queries (mandatory)

Query the experiment corpus for:
1. Systematic evaluation methods used in reference analyses
2. Published measurements for comparison (use `compare_measurements` for
   cross-experiment results)
3. Theory predictions or MC generator comparisons for the observable

Cite sources in the artifact.

## Applicable conventions

{{conventions_files}}

Re-read these before finalizing systematics.

## Key requirements

These are the critical items for Phase 4. See
`methodology/03-phases.md` → Phase 4 for full details.

- **Systematic completeness table.** Compare your implemented sources
  against the reference analyses from Phase 1 and the applicable
  `conventions/` files (see root CLAUDE.md → Conventions for which files
  apply). Format: `| Source | Conventions | Ref 1 | Ref 2 | This | Status |`.
  Any MISSING source without justification is a blocker. In particular,
  any source listed in the Phase 1 conventions enumeration as "Will
  implement" that is absent here is Category A (must resolve before
  advancing).
- **Statistical model construction.** Build a binned likelihood with all
  samples, Asimov data (pseudo-data generated under the background-only or
  nominal hypothesis), and systematic terms as nuisance parameters (NPs —
  parameters that encode systematic uncertainties in the fit). Validate:
  NP pulls small, fit converges, results physically sensible.
- **Fit validation.** Signal injection tests (searches) or closure tests
  (measurements) to confirm the model recovers known inputs.
- **Goodness-of-fit.** Report **both** chi2/ndf (quick assessment) **and**
  toy-based p-value using the saturated model GoF statistic (the saturated
  model treats each bin as an independent parameter — standard reference in
  pyhf/HistFactory). For pure counting extractions without a binned fit,
  use chi2 across bins or subperiods instead. chi2/ndf ~ 1 is good; >>1
  indicates mismodeling; <<1 indicates overestimated uncertainties.
- **All Phase 4a results are on Asimov pseudo-data.** There is no real
  data in this framework. Phase 4b is the toy-MC coverage scan, Phase 4c
  is the full-statistics expected result at design luminosity.
- **MC sample applicability.** Do not apply MC-derived quantities
  (efficiencies, corrections) beyond the kinematic or detector configuration
  they were derived from. If a correction was derived on Delphes and is
  then applied to CLD samples (or vice versa), the uncertainty on the
  result must reflect this extrapolation. The natural way to bound the
  effect is the dual-sim overlay (§4.3) — if Delphes and CLD agree on
  the corrected observable, the correction is transferable; if they
  disagree, the spread is the systematic.
- **Covariance matrix (measurements).** Full bin-to-bin covariance
  (statistical + each systematic + total) in the artifact and as
  machine-readable files.
- **Theory comparison (measurements).** Compare to at least one theory
  prediction or MC generator using the full covariance. If none available,
  justify and compare to published measurements.

**For extraction measurements:** read `conventions/extraction.md` for
additional required checks (independent closure test, parameter sensitivity
table, operating point stability, per-subperiod consistency, 10% diagnostic
sensitivity). These are technique-specific requirements defined in the
conventions file — do not skip them.

## COMMITMENTS.md (mandatory tracking artifact)

At Phase 4a start, read `COMMITMENTS.md` (created at Phase 1 completion).
Update every line's status:
- `[x]` — resolved (with phase and brief description)
- `[D]` — formally downscoped (with documented justification — "not
  attempted" is not a justification)
- `[ ]` — not yet addressed

Any `[ ]` remaining at Phase 5 is Category A. This prevents Phase 1
commitments from being silently dropped. See `methodology/03-phases.md`
for the full specification.

## Systematic implementation self-check (mandatory before submission)

For each systematic variation, verify and document in the artifact:
1. The varied quantity actually changes (print nominal vs varied values)
2. The impact is non-zero in at least some bins
3. The impact has the expected sign/direction
4. The evaluation level is consistent (gen vs reco — don't mix)
5. The variation was propagated through the chain, not borrowed as a flat %

See `methodology/03-phases.md` → Phase 4a for the full specification.

## Closure test alarm bands (mandatory)

These apply at Phase 4a (same as Phase 3):
- chi2/ndf < 0.1 → Category A (suspicious)
- chi2/ndf > 3 or pull > 5-sigma → Category A (failure)
- `passes: false` in JSON while text claims acceptable → Category A

See `methodology/03-phases.md` for the full specification.

## Number consistency gate (every AN compilation)

Before compiling any AN version (4a, 4b, 4c), verify that all numerical
values in the AN match the latest machine-readable outputs. Systematic
uncertainties, central values, event counts — everything must be current.
Any discrepancy > 1% relative is Category A. This gate prevents stale
numbers from earlier phases propagating forward.

## Pre-review self-check

Before submitting for review, verify:

- [ ] Systematic completeness table: every conventions source + every
      reference analysis source, row-by-row
- [ ] Every systematic has measured/cited variation size (not arbitrary
      round numbers — ±50% requires measured justification)
- [ ] No flat borrowed systematics (unless all 3 conditions documented:
      confirmed subdominant + propagation infeasible + cited measurement)
- [ ] Signal injection or closure test passes
- [ ] GoF: chi2/ndf AND toy-based p-value both reported
- [ ] Covariance matrix validated (PSD, reasonable condition number)
- [ ] Per-systematic documentation: running prose with physical origin,
      evaluation method, numerical impact, interpretation
- [ ] AN draft complete with PDF compiled (4a, 4b)
- [ ] For extraction: all `conventions/extraction.md` checks completed
- [ ] All figures pass plotting rules (see Phase 2 quick reference)
- [ ] Validation target check: results compared to PDG/references,
      any pull > 3-sigma or deviation > 30% triggers investigation (§6.8)
- [ ] Phase 1 traceability: re-read STRATEGY.md, verify every committed
      systematic/variation was implemented or formally downscoped ([D] label)
- [ ] Zero-systematic sanity: no source has impact exactly 0 without
      verified non-trivial variation input (0 = likely broken, not negligible)

Also check `methodology/appendix-checklist.md` for the full artifact and
AN completeness checklists.

**Your reviewer will check** (§6.4): Systematics complete vs conventions +
references? Variation sizes justified? Signal injection/closure passes?
MC coverage matches data? Validation targets (§6.8)?

## Review

**Review is mandatory at all three sub-phases — 4a, 4b, and 4c.**

| Sub-phase | Review tier | Panel composition | Then |
|-----------|-------------|-------------------|------|
| 4a | 4-bot+bib | physics + critical + constructive + plot validator + bibtex | arbiter |
| 4b | 4-bot+bib | physics + critical + constructive + plot validator + bibtex | arbiter → human gate |
| 4c | 1-bot | critical + plot validator | (no arbiter) |

4b is NOT "human gate only" — it gets the full 4-bot+bib review panel
first, then the human gate after the arbiter returns PASS. 4c is NOT
unreviewed — it gets 1-bot (critical + plot validator). Skipping or
omitting these reviews is a process failure.

See `methodology/06-review.md` for protocol. Write findings to
`review/{role}/` using session-named files (see
`methodology/appendix-sessions.md` for naming conventions).
