# Phase 1: Strategy

> Read `methodology/03-phases.md` → "Phase 1" for full requirements.
> Read `conventions/fcc_ee.md` for the FCC-ee / Key4hep / EDM4hep domain.

You are developing the analysis strategy for a **{{analysis_type}}**
analysis at FCC-ee. All inputs are simulation (Delphes fast sim and/or
CLD/IDEA full sim). No real data — see `methodology/04-staged-validation.md`
for the MC-only staging protocol.

**Start in plan mode.** Before writing any code or prose, produce a plan:
what literature you will query, what samples you expect, what the artifact
structure will be. Execute after the plan is set.

## Output artifact

`outputs/STRATEGY.md` — analysis strategy with physics motivation, sample
inventory, selection approach, systematic plan, and technique selection.

## Methodology references

- Phase requirements: `methodology/03-phases.md` → Phase 1
- Review protocol: `methodology/06-review.md` → §6.2 (4-bot), §6.4
- Artifacts: `methodology/05-artifacts.md`

## Literature queries (mandatory)

Before writing the strategy, query for:
1. Prior FCC-ee projection studies on the same or similar observable
   (`fcc-physics-events`, arXiv, the FCC feasibility study report)
2. Legacy LEP measurements of the same observable where applicable
   (ALEPH/DELPHI/L3/OPAL) — these set the current world-average
3. Published systematic programmes used by reference analyses in
   FCCPhysics (<https://github.com/jeyserma/FCCPhysics/tree/main/analyses>)
   or FCCAnalyses examples
4. Theory predictions / MC generator benchmarks for the observable
   (Pythia8, Whizard, KKMCee, Sherpa)

Cite all retrieved sources in the artifact (paper ID / arXiv / URL +
section).

## Required deliverables

- Physics motivation and observable definition
- Sample inventory (data + MC) — **both chains**: Delphes winter2023
  IDEA samples (primary by default) AND CLD/IDEA full-sim samples
  (secondary by default). If either chain's samples are not yet
  available, name the trigger for production and commit to folding
  the second chain in via a scheduled regression (§4.3.4).
- Selection approach with justification (see "≥2 approaches" below)
- Systematic uncertainty plan — per chain, because some sources
  (e.g., PFA mis-assignment) are CLD-only while Delphes parameterisation
  changes are Delphes-only.
- Literature review from RAG corpus
- **Technique selection** — determine the analysis technique (unfolding,
  template fit, etc.) and justify the choice. This determines which
  technique-specific requirements apply in later phases.
- **Dual-chain plan (binding)** — name the primary and secondary
  chain, commit to running Phase 3 and Phase 4 on both, and commit to
  producing `COMPARISON_dual_sim.md` at Phase 4c and the dual-chain
  comparison chapter in the Phase 5 AN. This is recorded in
  `COMMITMENTS.md` and tracked by the orchestrator. See
  `methodology/04-staged-validation.md` §4.3.

## Applicable conventions

{{conventions_files}}

Read these before writing the systematic plan.

## Key requirements

These are the critical actionable items for Phase 1. See
`methodology/03-phases.md` → Phase 1 for full details.

- **Literature queries are mandatory.** Query for prior FCC-ee
  projections, legacy LEP measurements, reference FCCPhysics analyses,
  and theory predictions before writing anything. Cite all retrieved
  sources.
- **Enumerate backgrounds.** Classify each as irreducible, reducible, or
  instrumental. Estimate relative importance (order of magnitude is fine).
- **Define discriminating variables.** Identify the variable(s) for final
  statistical interpretation (invariant mass, BDT score, event shape, etc.).
- **≥2 selection approaches must be qualitatively different.** Two
  parametric variants of the same method (e.g., two different IP cuts) do
  NOT count as distinct approaches. At least one approach must be MVA-based
  (BDT on available discriminating variables) unless a concrete constraint
  makes MVA infeasible — in which case the constraint must be documented
  with a [D] label and validated at review. Phase 3 treats cut-based
  selection as a downscope from MVA (see `methodology/12-downscoping.md`),
  so the strategy must at minimum identify what MVA inputs are available
  and why an MVA is or isn't planned.
- **Systematic plan with conventions enumeration.** Read the applicable
  `conventions/` files listed above. For every required source listed, state
  "Will implement" or "Not applicable because [reason]." This enumeration
  is binding — Phase 4a reviews against it. Silent omissions are Category A
  (must-resolve findings that block advancement — see review protocol).
- **Reference analysis table.** Identify 2-3 published analyses closest in
  technique/observable. Tabulate their systematic programs. This table is a
  binding input to later reviews.

**For measurements additionally:**
- Define the observable(s) and their physical interpretation precisely.
- Identify the correction/unfolding strategy and its required inputs.
- Survey prior measurements — published data points become the primary
  validation target in Phase 4.
- Identify theory predictions or MC generators for comparison.
- **Flagship figures.** Identify ~6 figures that would represent the
  measurement in a journal paper. Examples: the final spectrum with
  uncertainties, the response matrix, the key data/MC comparison, the
  systematic breakdown, the theory comparison overlay. These are defined
  here and produced at the highest quality in Phase 5.

## Pre-review self-check

Before submitting for review, verify:

- [ ] Corpus queries executed — at least 3 searches, all results cited
- [ ] Backgrounds classified (irreducible, reducible, instrumental)
- [ ] >=2 qualitatively different selection approaches identified (not
      parametric variants of same method). At least one MVA-based, or
      MVA infeasibility documented with [D] label
- [ ] Systematic plan enumerates EVERY source in applicable conventions
      files: "Will implement" or "Not applicable because [reason]"
- [ ] Reference analysis table: 2-3 analyses with systematic programs
- [ ] Method parity: if references used a more sophisticated method,
      committed to matching it or implementing as cross-check
- [ ] Constraint [A], limitation [L], and decision [D] labels defined
- [ ] For measurements: flagship figures (~6) identified, correction
      strategy defined, theory comparison independence verified
- [ ] **Dual-chain plan (§4.3):** primary chain named, secondary chain
      named, samples for each inventoried (or trigger for production
      documented), binding commitment to `COMPARISON_dual_sim.md` at 4c
      and dual-chain chapter at Phase 5 recorded in COMMITMENTS.md.
      Single-chain outcome requires an explicit [A] constraint with a
      named trigger.

**Your reviewer will check** (§6.4): Backgrounds complete? Systematic
plan covers conventions? Reference analyses tabulated? >=2 qualitatively
different selection approaches (not variants of the same cut)? MVA
considered or infeasibility justified? Method parity with published analyses?

## Review

**4-bot review** — see `methodology/06-review.md` for protocol.
Write findings to `review/{role}/` using session-named files
(see `methodology/appendix-sessions.md` for naming conventions).
