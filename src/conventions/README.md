# Physics Conventions

This directory contains accumulated domain knowledge for specific analysis
techniques and for the FCC-ee / Key4hep / EDM4hep experimental context.
These are **not** part of the methodology specification — the spec
describes process (phases, reviews, gates), while conventions encode what
experienced analysts know about how to do specific things correctly.

**Always read `fcc_ee.md` first.** It is the experimental-context document
for this repository: sample conventions, `fccanalysis run` usage, the
EDM4hep collection glossary, the Delphes↔CLD collection aliasing shim,
the FCC-ee plot-label style, and the **dual-chain comparison pattern**.
Every analysis here is reproduced on both simulation chains (Delphes
fast sim AND CLD full sim) with a final-result comparison at Phase 4c;
`fcc_ee.md` gives the chain-portable code pattern and the
`COMPARISON_dual_sim.md` artifact contract, while
`methodology/04-staged-validation.md` §4.3 gives the phase-by-phase
execution protocol. Every analysis reads `fcc_ee.md` at Phase 1 in
addition to the technique-specific file (`extraction.md`, `unfolding.md`,
or `search.md`).

## Relationship to the spec

The spec principle "no encoded physics" remains intact. The spec tells the
agent *that* it must evaluate systematic uncertainties; a conventions document
tells it *which* systematic sources are standard for a given technique. The
spec is stable across analysis types; conventions grow as the collaboration
gains experience.

Conventions are consulted, not blindly followed. If a convention doesn't
apply to a specific analysis, the agent documents why and proceeds. If a
convention is missing, the agent uses literature and RAG to fill the gap,
then adds what it learned here for future analyses.

## Structure

One file per analysis technique or method. Each file covers:

- When to use this technique
- Standard configuration and parameters
- Required validation checks
- Required systematic sources with rationale
- Known pitfalls

## Maintenance

- **Living documents.** Updated after each analysis that uses the technique.
- **Empirically grounded.** Entries come from actual analysis experience and
  published reference analyses, not from speculation.
- **Agent-maintained.** Agents update conventions when they encounter new
  knowledge. Human review before merging.
- **Consulted at Phase 1 (Strategy) and Phase 4a (Systematics).** The
  strategy phase uses conventions to plan the systematic program. The
  Phase 4a review checks completeness against conventions.

## LaTeX Preamble Governance

`preamble.tex` is the standard LaTeX preamble shared across all analyses.
It is **not modified during an analysis.**

If an analysis needs additional LaTeX packages (e.g., `tikz` for diagrams,
`siunitx` for units):

1. Create `phase5_documentation/outputs/analysis_preamble.tex` with the
   additional `\usepackage` commands
2. Document the additions and rationale in the experiment log
3. Include both preambles in the pandoc command:
   `--include-in-header=../../conventions/preamble.tex --include-in-header=outputs/analysis_preamble.tex`
4. Reviewer checks at Phase 4a (first PDF compilation) that custom packages
   do not conflict with the standard preamble
5. Document custom packages in the AN appendix

Post-analysis: if a custom package proved generally useful, propose its
addition to the standard `preamble.tex` as a conventions update.
