# Extraction Measurements (Double-Tag / Hemisphere Counting)

Conventions for analyses that extract a physical parameter using double-tag
or hemisphere-counting methods — where the result comes from a closed-form
formula applied to observed yields and MC-derived efficiencies, exploiting
the self-calibrating properties of multi-tag counting.

## When this applies

Any analysis where the primary result is computed from a closed-form
expression of tagged and untagged yields in hemisphere pairs — not from a
template fit or unfolding procedure. The canonical example is R_b extraction
from N_tt / N_had using hemisphere tagging efficiencies.

Other extraction techniques (tag-and-probe efficiency measurements,
branching fraction ratios, cross-section ratios) share some conventions
with this file but have distinct requirements. They should use dedicated
conventions files when available.

If the analysis uses a binned likelihood fit to a discriminant shape, the
`unfolding.md` or search conventions apply instead.

---

## Standard configuration

- **Asimov pseudo-data for Phase 4a.** The expected result must be
  computed on Asimov pseudo-data generated from the nominal MC model,
  with bin contents equal to the exact expected values (no fluctuations).
  For closed-form extractions, set the inputs to their MC-truth values
  and evaluate the formula. This is the baseline for the toy scan in
  Phase 4b.
- **Fixed random seed.** All toy generation and subsample selection
  use documented fixed seeds for reproducibility.
- **Per-sample-subset granularity.** When multiple MC sub-samples exist
  (generator variants, detector configurations), track the result per
  subset as a standard cross-check.
- **Counting vs. likelihood extraction.** Pure counting (closed-form
  formula applied to yields) is appropriate when the extraction formula is
  simple and the systematic treatment is transparent. Likelihood extraction
  (fitting a model to binned or unbinned data) is preferred when: multiple
  parameters are extracted simultaneously, nuisance parameters need profiling,
  or correlations between inputs are complex. Document the choice and justify.
- **Uncertainty propagation.** For counting extractions with few inputs,
  analytical error propagation (partial derivatives) is standard. For
  extractions with many correlated inputs or non-linear formulas, toy-based
  propagation (Poisson-fluctuate inputs, repeat extraction, take RMS) is
  more robust. Report which method is used and, if analytical, verify
  against toys for at least the dominant sources.
- **Efficiency binning.** Efficiency corrections must be derived in bins
  fine enough to capture kinematic dependence but coarse enough for adequate
  statistics per bin. As a rule of thumb, each efficiency bin should contain
  at least ~100 MC events; below this, statistical noise in the correction
  dominates. Document the binning choice and its motivation.
- **Cross-chain calibration (scale factors).** When the extraction depends
  on MC-derived efficiencies (e.g., tagging efficiency) and both Delphes
  and CLD samples are available, derive the fast-sim-to-full-sim scale
  factor per kinematic bin and apply it to the Delphes efficiencies
  before extraction. If only one chain is available, assign a systematic
  covering the expected fast/full difference from published FCC-ee
  detector-simulation comparisons.

  **Calibration independence is mandatory.** Each scale factor must come
  from an observable independent of the primary result. For example,
  tagging efficiency can be calibrated from the impact parameter
  resolution, or from a tag-and-probe method on a known sample. Deriving
  the correction by assuming the primary result equals a reference value
  (back-substitution) is a diagnostic, not a calibration — see
  `methodology/06-review.md` §6.8 Tier 2 for the full independence
  classification.

---

## Required systematic sources

### Efficiency modeling

| Source | What to vary | Rationale |
|--------|-------------|-----------|
| Tag/selection efficiency | Vary efficiency corrections within their uncertainties | The extracted quantity depends directly on efficiency estimates |
| Efficiency correlation | Evaluate hemisphere or object correlation effects | Double-tag methods assume independence; violations bias the result |
| MC efficiency model | Compare efficiencies from alternative MC generators | Generator-dependent fragmentation affects tagging efficiency |

### Background contamination

| Source | What to vary | Rationale |
|--------|-------------|-----------|
| Non-signal contamination | Vary background fractions within estimated uncertainties | Residual backgrounds in the tagged sample bias the yield |
| Background composition | Use alternative models for background mixture | The relative contribution of different backgrounds affects the correction |

### MC model dependence

| Source | What to vary | Rationale |
|--------|-------------|-----------|
| Hadronization model | Compare generators with different fragmentation (string vs. cluster) | Fragmentation model affects both efficiencies and acceptance |
| Physics parameters | Vary heavy-quark mass, fragmentation function parameters | Input physics parameters propagate to the extracted quantity |

### Sample composition

| Source | What to vary | Rationale |
|--------|-------------|-----------|
| Flavour composition | Vary assumed non-signal flavour fractions | Extraction formulas depend on the composition of the inclusive sample |
| Production fractions | Vary assumed production ratios if used as inputs | Any external input contributes its uncertainty |

---

## Required validation checks

1. **Independent closure test (Category A if fails).** Apply the full
   extraction procedure to a statistically independent MC sample (not the
   sample used to derive efficiencies or corrections). Extract the quantity
   and compare to MC truth. The pull (extracted value minus truth, divided
   by the method's uncertainty) must be < 2 sigma. Failure indicates a bias
   in the method.

2. **Parameter sensitivity table.** For each MC-derived input parameter,
   compute |dResult/dParam| * sigma_param. Flag any parameter contributing
   more than 5x the data statistical uncertainty — these are the dominant
   systematics and require careful evaluation.

3. **Operating point stability (Category A if fails).** Scan the extracted
   result vs. the primary selection variable (e.g., the classifier working
   point or the cut threshold) over a range spanning at least 2x the
   optimized region. The result must be flat within uncertainties — a
   dramatic variation indicates the measurement is not robust and the
   operating point is not in a stable plateau. This is a physics red flag,
   not just a systematic: it means the result depends critically on an
   arbitrary choice. Investigate before proceeding.

   **The stability scan must include fit quality.** Report chi2/ndf (or
   equivalent GoF metric) at each scan point alongside the extracted
   value. A configuration that produces a small statistical uncertainty
   but poor GoF (chi2/ndf > 3) is not a stable operating point — it
   indicates the model does not describe the data at that configuration.
   When selecting among multiple configurations (e.g., kappa values,
   binning choices), the selection criterion must balance precision and
   GoF. If the minimum-variance configuration has poor GoF while other
   configurations have acceptable GoF, the latter should be preferred
   unless the GoF failure is understood and demonstrated not to bias the
   result.

4. **Per-subperiod consistency.** Extract the result independently for each
   data-taking period. Compute chi2/ndof across periods. A chi2/ndof >> 1
   indicates time-dependent effects (detector aging, calibration drift) not
   captured by the MC model.

5. **Toy-scan diagnostic sensitivity (Phase 4b).** The Phase 4b toy
   scan must include at least one diagnostic genuinely sensitive to
   model mis-specification — not just the distribution of the extracted
   quantity (which is dominated by correlated systematics and largely
   insensitive to toy-to-toy differences). Required: the distribution
   of intermediate tag rates, double-tag fractions, or per-category
   yields across toys, compared to the Asimov expectation from 4a.

---

## Pitfalls

- **Running 4a on the same sample that generates the toys.** 4a uses
  Asimov pseudo-data (exact expected bin contents). If the executor
  generates fluctuated toys in 4a instead, there is nothing left to
  measure in 4b — and the toy scan loses its diagnostic power. Asimov
  in 4a, toys in 4b.

- **Insensitive toy scan.** Comparing only the distribution of the final
  extracted quantity across toys tells you little — correlated
  systematics dominate. The toy scan must include intermediate
  diagnostics (tag rates, efficiency distributions, per-subset results)
  that can move toy-to-toy independently of the nuisance structure.

- **Missing independent MC for closure.** The closure test must use an MC
  sample statistically independent from the one used to derive corrections
  and efficiencies. Using the same sample tests self-consistency, not the
  method's validity. If only one MC sample is available, split it (using a
  documented fixed seed) into derivation and validation halves. **A
  self-consistent extraction (deriving efficiencies and counting yields
  from the same sample) always recovers the correct answer by
  construction — this is an algebra check, not a closure test.** If a
  closure test produces pull = 0.00 at every operating point, this is
  a red flag that it is self-consistent rather than independent.
  Investigate before proceeding.

- **Assuming hemisphere independence.** Double-tag methods often assume that
  tagging in one hemisphere is independent of the other. QCD correlations
  (gluon splitting, color reconnection) violate this. Evaluate the
  correlation coefficient from MC and propagate its uncertainty.

- **MVA-induced hemisphere correlations.** When using an MVA (BDT, NN)
  for hemisphere tagging in a double-tag analysis, classifier inputs
  that are correlated with event-level properties (total multiplicity,
  event shapes, or hemisphere quantities like mass that are coupled
  across hemispheres) inflate the hemisphere correlation factors C_q.
  Check C_q at the working point: values far from 1.0 (C_b < 0.8 or
  C_b > 1.3) indicate the classifier introduces correlations beyond
  the QCD effects. Identify which inputs cause the correlation and
  consider removing them — the loss in AUC may be small while the
  gain in C_b stability is large. The self-calibrating advantage of
  the double-tag method relies on C_b ~ 1; large corrections amplify
  the systematic uncertainty on C_b.

- **MVA inputs correlated with the discriminant.** Every MVA input is
  correlated with the classifier output (the discriminant) — that is
  why it is an input. If the input is poorly modelled in MC (data/MC
  chi^2/ndf > 5), the discriminant distribution will differ between data
  and MC, and the efficiency at a given cut will not transfer. This
  failure mode is invisible to MC closure tests (which use only MC).
  The input variable quality gate (§3, Phase 3) exists to catch this
  before training. Inputs that are strong discriminators but poorly
  modelled should be calibrated (reweighted) or discarded — see §7.5
  for the required before/after verification.

- **Neglecting non-primary flavours.** Extraction formulas typically include
  terms for non-signal flavours (e.g., charm and light quark contributions
  in an R_b measurement). Setting these to nominal MC values without
  uncertainty propagation underestimates the systematic error.

- **Circular luminosity / cross-section inputs are Category A.** For
  FCC-ee projection studies the integrated luminosity is a design
  input (e.g. 10.8 ab⁻¹ at 240 GeV, 3.12 ab⁻¹ at 365 GeV). It MUST
  come from the cited FCC-ee feasibility study, not be derived from
  the simulated event count and assumed cross sections. If the
  analysis back-calculates luminosity from N_events / (σ × ε), the
  cross-section fit recovers σ trivially — this is an identity, not
  a measurement.

  Similarly, process cross sections feeding the `processList` / procDict
  must come from the matching FCCAnalyses samplesDict (which traces back
  to the generator). Do not hand-tune cross sections to improve agreement
  between two samples or between two sim chains.
- **Inflated uncertainties from coarse-scan systematics.** When a
  systematic is evaluated by scanning a parameter (kappa, binning
  choice, alternative method) on a reduced MC sample and then applied
  to the full-statistics result, verify that the full-stats evaluation
  is consistent with the coarse-scan spread. If the full-stats spread
  is significantly smaller (>2x), the coarse evaluation may be inflated
  — the coarse configuration space may include unphysical or irrelevant
  variations that the full-stats fit naturally constrains. Use the
  full-stats evaluation as the primary systematic, with the coarse
  evaluation as an upper bound only if the full-stats scan has too few
  points to be reliable. Inflated systematics are not "conservative" —
  they obscure the measurement's true sensitivity and make validation
  checks (pull < 2σ) meaningless. A measurement where the dominant
  systematic is 3x larger than necessary is not a measurement of the
  observable — it's a measurement of the systematic evaluation procedure.
- **Correlated uncertainties in combinations.** When combining results
  from multiple observables, methods, or subsamples, identify which
  systematic sources are shared (e.g., renormalization scale variation
  affects all observables identically; generator choice affects all
  channels). Shared sources must enter the combination as 100%
  correlated — not added in quadrature as if independent. Treating
  a dominant correlated systematic as uncorrelated can reduce the
  combined uncertainty by a factor of sqrt(N_observables), which is
  fictitious. Document the correlation assumption for each source
  in the combination formula.

---

## References

- LEP/SLD EWWG combination: "Precision electroweak measurements on the Z
  resonance" (Phys. Rept. 427, 257, 2006). INSPIRE: ALEPH:2005ab.
  Defines the standard methodology for heavy-flavour extraction at the Z.
- ALEPH R_b measurement: "A measurement of R_b using a lifetime-mass tag"
  (Phys. Lett. B401, 163, 1997). INSPIRE: Barate:1997ha. Reference for
  double-tag counting with hemisphere correlations.
- DELPHI R_b/R_c: DELPHI double-tag measurements of R_b and R_c — multiple
  papers across 1995-2000. Use `search_lep_corpus` with query "DELPHI R_b
  double tag" to retrieve specific papers and their systematic programs.
- SLD R_b: "A measurement of R_b using a vertex mass tag" (Phys. Rev. Lett.
  80, 660, 1998). INSPIRE: Abe:1997sb. Reference for high-purity
  single/double tag methods with self-calibrating efficiencies.
- PDG review: "Electroweak model and constraints on new physics" in the
  Review of Particle Physics. Current world averages for R_b, R_c, and
  correlation matrices between electroweak observables.
- "Model-independent ZH production cross section at FCC-ee",
  arXiv:2512.21290. Reference analysis for ZH recoil extractions at
  FCC-ee at √s = 240 GeV and 365 GeV (μμ, ee, hadronic channels).
- FCCAnalyses ZH recoil example:
  `examples/FCCee/higgs/mH-recoil/histmaker_mumu.py` at
  <https://github.com/HEP-FCC/FCCAnalyses>.
- FCCPhysics ZH leptonic reference (Jeyserma): `analyses/h_zh/h_zh_leptonic.py`
  at <https://github.com/jeyserma/FCCPhysics>.
