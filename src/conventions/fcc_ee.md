# FCC-ee Analysis Conventions (Key4hep / EDM4hep)

Conventions for analyses targeting the Future Circular Collider (FCC-ee),
using simulation samples produced either with Delphes fast simulation or
with CLD / IDEA full Geant4 simulation, in both cases written to the
EDM4hep event data model.

## When this applies

Any analysis whose inputs are EDM4hep ROOT files produced on the Key4hep
software stack — whether the detector response is Delphes (fast sim) or
DD4hep + Geant4 (full sim). All analyses in this repository should read
this file at Phase 1 in addition to the technique-specific conventions
(`extraction.md`, `unfolding.md`, `search.md`).

## Simulation chains

Two chains produce EDM4hep output. The physics model (generator, event
record, `MCParticles` collection) is identical; only the detector
response differs.

| Chain | Detector response | Typical input | Typical output |
|-------|-------------------|---------------|----------------|
| **Delphes (fast sim)** | Parameterised smearing and efficiency on `MCParticles`. IDEA or CLD-like card. | Pythia8 stdhep / lhe / hepmc | EDM4hep ROOT, e.g. `wzp6_ee_mumuH_ecm240` winter2023 IDEA production |
| **Full sim (DD4hep + Geant4)** | Full particle transport through DD4hep geometry (`CLD_o2_v07` is the tutorial default), then reconstruction via `k4run CLDReconstruction.py` producing Pandora PFOs and associated tracks / clusters. | Same generator `.stdhep` file as the Delphes chain | EDM4hep ROOT REC file (`*_REC.edm4hep.root`) plus an `*_aida.root` diagnostic file |

Both chains produce `ReconstructedParticle`, `MCParticle`, and
`MCRecoAssociation` collections. **Collection names are not identical**
between the two chains — see the "Collection aliasing" section below.

## Software stack glossary

- **Key4hep** — Umbrella software stack for future collider software,
  distributed via `/cvmfs/sw.hsf.org/key4hep/setup.sh`; it provides DD4hep
  geometry, Gaudi, podio, EDM4hep, Pandora, and FCCAnalyses as a versioned
  nightly or release set.
- **DD4hep** — Detector description toolkit; geometries are defined in XML
  "compact" files plus C++ detector builders.
- **ddsim** — DD4hep driver that runs a Geant4 simulation from a generator
  input file (stdhep / hepmc / lhe / hepevt).
- **k4run** — Python-driven Gaudi runner used to execute reconstruction and
  analysis options files.
- **Gaudi** — Event-processing framework that hosts reconstruction algorithms.
- **podio** — Event data model toolkit that generates the EDM4hep classes
  and their I/O layer. `podio-dump <file>.root` prints all collections.
- **EDM4hep** — Common event data model used by both Delphes fast-sim and
  CLD full-sim output: simulated hits, digitised hits, reconstructed
  particles, MC particles, and associations between them.
- **Pandora (PandoraPFA)** — Particle-flow algorithm that clusters tracks
  and calorimeter hits into reconstructed particles (PFOs) in full sim.
- **FCCAnalyses** — RDataFrame-based analysis framework. A user analysis
  is a single Python file that sets `processList`, `prodTag` / `inputDir`,
  `procDict`, and defines a `build_graph(df, dataset)` function returning
  a list of histogram results. Run with `fccanalysis run <file>.py`.

## Environment bootstrap

FCC-ee analysis scripts need Key4hep, which ships via cvmfs and is NOT
installed as a pixi package. The expected bootstrap at the top of every
analysis session:

```bash
source /cvmfs/sw.hsf.org/key4hep/setup.sh
# or to pin a release:
# source /cvmfs/sw.hsf.org/key4hep/releases/2025-01-28/x86_64-el9-gcc14.2.0-opt/key4hep-stack/<ver>/setup.sh
```

After this, `fccanalysis`, `k4run`, `ddsim`, `podio-dump`, and Python with
EDM4hep bindings are all on `PATH`. The analysis's `pixi.toml` still
provides matplotlib, mplhep, numpy, uproot for plotting and post-processing;
it does NOT try to conda-install FCCAnalyses or podio. If a pixi task
needs FCCAnalyses on `PATH`, wrap it with `source` inside the task string.

## FCCAnalyses histmaker pattern

Modern FCC-ee analyses use the histmaker pattern: a single Python file
with module-level configuration and a top-level `build_graph(df, dataset)`
that returns histogram results.

```python
import ROOT

# --- where the data lives ---
# Central EOS catalog (recommended on lxplus / AFS):
prodTag  = "FCCee/winter2023/IDEA/"

# --- or local mirror: ---
# inputDir = "/path/to/DelphesEvents/winter2023/IDEA/"
# procDict = "FCCee_procDict_winter2023_IDEA.json"

# --- processes and their cross sections (pb) ---
processList = {
    "wzp6_ee_mumuH_ecm240": {"fraction": 1.0},
    "p8_ee_WW_mumu_ecm240":  {"fraction": 1.0},
    "p8_ee_ZZ_mumu_ecm240":  {"fraction": 1.0},
    "wzp6_ee_mumu_ecm240":   {"fraction": 1.0},
}

nCPUS     = 8
doScale   = True
intLumi   = 10.8e6        # pb^-1  (10.8 ab^-1 at 240 GeV)
outputDir = "phase3_selection/outputs/"

# C++ helpers loaded into the RDF
includePaths = ["utils.h"]

def build_graph(df, dataset):
    results = []
    df = df.Alias("Muon0", "Muon#0.index")   # Delphes-only shim
    df = (
        df
        .Define("muons", "ReconstructedParticle::get(Muon0, ReconstructedParticles)")
        .Define("muons_p", "ReconstructedParticle::get_p(muons)")
        .Define("muons_sorted",
                "ReconstructedParticle::sel_p(muons, 20., 80.)")
        .Filter("muons_sorted.size() == 2")
        .Define("zed_leptonic",
                "ReconstructedParticle::resonanceBuilder(91)(muons_sorted)")
        .Define("zed_m",
                "ReconstructedParticle::get_mass(zed_leptonic)")
        .Define("recoil",
                "ReconstructedParticle::recoilBuilder(240)(zed_leptonic)")
        .Define("recoil_m",
                "ReconstructedParticle::get_mass(recoil)")
    )
    results.append(df.Histo1D(("recoil_m", "", 200, 120, 140), "recoil_m"))
    results.append(df.Histo1D(("zed_m",    "", 200,  70, 110), "zed_m"))
    return results
```

Run it:
```bash
fccanalysis run phase3_selection/src/histmaker.py
```

The output is one ROOT file per process in `outputDir`, each containing
the histograms returned by `build_graph`. These are what Phase 4 stacks,
fits, and plots.

## EDM4hep collection quick reference

These are the collections every FCC-ee analysis will touch. Names marked
(fast) appear only in Delphes output; names marked (full) appear only in
CLD reconstruction output; names with no marker appear in both.

| Collection | Chain | What it holds |
|------------|-------|---------------|
| `MCParticles` | both | Generator-level particles (particle, status, parent/daughter indices). Alias as `Particle` in FCCAnalyses helpers. |
| `Particle#0.index`, `Particle#1.index` | both | Parent/daughter relations for `MCParticles` (podio one-to-many as side-collections). |
| `ReconstructedParticles` | fast | Delphes reconstructed particles (PF-like: tracks, neutral hadrons, photons, leptons). |
| `PandoraPFOs` (or `TightSelectedPandoraPFOs`) | full | Pandora particle-flow output. Semantic equivalent of `ReconstructedParticles` for full sim. |
| `Muon#0.index`, `Electron#0.index`, `Photon#0.index` | fast | Flat integer side-collections indexing isolated muons / electrons / photons inside `ReconstructedParticles`. Delphes-only — these do NOT exist in full sim output. |
| `MCRecoAssociations` | both | Association between `ReconstructedParticle`s and their truth `MCParticle`s. Use `#0.index` (reco) and `#1.index` (truth) to dereference. |
| `EFlowTrack`, `EFlowPhoton`, `EFlowNeutralHadron` | fast | Delphes particle-flow sub-collections (rarely needed if you use `ReconstructedParticles`). |
| `SiTracks`, `PandoraClusters`, `TrackerHitPlanes` | full | Full-sim reconstruction primitives; useful for tracking / calorimeter performance studies. |

**Verify before you write code.** Always run
`podio-dump <one_sample_file>.root` once on a sample of each kind, paste
the output into the experiment log, and implement against the names that
are actually present. Assumptions about collection names are the single
biggest source of "code ran on Delphes, silently crashes on CLD" bugs.

## Collection aliasing (Delphes ↔ CLD shim)

A histmaker built from the FCCAnalyses Delphes examples relies on
`Muon#0.index`, `Electron#0.index`, `Photon#0.index` to get pre-selected
lepton / photon lists. CLD full-sim output has none of these — the
Pandora PFOs are all in one flat collection and must be filtered by
`|PDG| == 13` (or reconstructed-particle type).

Keep the physics graph identical between chains; isolate the difference
in one small shim file that is imported by both the fast-sim and full-sim
run scripts.

```python
# common/build_graph.py
def build_graph(df, dataset, sim_chain: str):
    df = apply_sim_aliases(df, sim_chain)
    df = (
        df
        .Define("muons",          "sel_muons(ReconstructedParticles)")
        .Define("muons_sorted",   "ReconstructedParticle::sel_p(muons, 20., 80.)")
        .Filter("muons_sorted.size() == 2")
        .Define("zed_leptonic",   "ReconstructedParticle::resonanceBuilder(91)(muons_sorted)")
        .Define("zed_m",          "ReconstructedParticle::get_mass(zed_leptonic)")
        .Define("recoil",         "ReconstructedParticle::recoilBuilder(240)(zed_leptonic)")
        .Define("recoil_m",       "ReconstructedParticle::get_mass(recoil)")
    )
    # histograms (unchanged across chains) ...
    return [...]

def apply_sim_aliases(df, sim_chain: str):
    if sim_chain == "fast":
        # Delphes: pre-built muon side-collection
        df = df.Alias("Muon0", "Muon#0.index")
        ROOT.gInterpreter.Declare(r"""
            auto sel_muons(const ROOT::VecOps::RVec<edm4hep::ReconstructedParticleData>& rps) {
              return ReconstructedParticle::get(Muon0, rps);
            }
        """)
    elif sim_chain == "full":
        # Full sim: rename PandoraPFOs, filter by |PDG| == 13
        df = df.Alias("ReconstructedParticles", "PandoraPFOs")
        ROOT.gInterpreter.Declare(r"""
            auto sel_muons(const ROOT::VecOps::RVec<edm4hep::ReconstructedParticleData>& rps) {
              ROOT::VecOps::RVec<edm4hep::ReconstructedParticleData> out;
              for (const auto& p : rps) if (std::abs(p.PDG) == 13) out.push_back(p);
              return out;
            }
        """)
    else:
        raise ValueError(f"unknown sim_chain: {sim_chain}")
    return df
```

Each chain has a one-page run script (`run_fastsim.py` / `run_fullsim.py`)
that sets `processList`, `inputDir` / `prodTag`, and calls into the
common `build_graph`. See `analyses/zh_recoil_mumu/` in this repository
for a worked example.

## Sample discovery and procDict

FCCAnalyses has two equivalent ways to discover samples:

1. **Central EOS catalog via `prodTag`.** Setting
   `prodTag = "FCCee/winter2023/IDEA/"` resolves sample paths from the
   central YAML under
   `root://eospublic.cern.ch//eos/experiment/fcc/ee/generation/DelphesEvents/...`.
   Use this when running from lxplus or AFS — it is the path of least
   resistance for Delphes samples.

2. **Local directory via `inputDir` + `procDict`.** Setting `inputDir`
   to a filesystem path plus `procDict` pointing at a `samplesDict.json`
   overrides the prodTag lookup. Use this for locally produced samples
   — the full-sim CLD leg of any analysis is always in this mode,
   because no central CLD production has a shipped procDict.

For full-sim files you generate yourself (the expected mode for this
repository's analyses), write an inline process dictionary:

```python
processList = {
    "wzp6_ee_mumuH_ecm240_cld": {
        "fraction":      1.0,
        "crossSection":  0.201868,  # pb, from Delphes procDict
        "numberOfEvents": 10000,
        "sumOfWeights":   10000,
    },
}
```

`crossSection` values for signal and background processes should come
from the matching Delphes `samplesDict.json` — they are generator-level
and detector-independent, so they transfer from fast sim to full sim
unchanged.

## Full-sim production (CLD via ddsim + k4run)

There is no central catalog of CLD-reconstructed samples that matches
the Delphes winter2023 production. Full-sim samples are produced by
the user, following the tutorial at
<https://hep-fcc.github.io/fcc-tutorials/main/full-detector-simulations/FCCeeGeneralOverview/FCCeeGeneralOverview.html>.

Starting from the same generator-level `.stdhep` that Delphes used:

```bash
source /cvmfs/sw.hsf.org/key4hep/setup.sh

# 1. Detector simulation (Geant4 via DD4hep)
ddsim -I wzp6_ee_mumuH_ecm240_GEN.stdhep -N 10000 \
      -O wzp6_ee_mumuH_ecm240_SIM.root \
      --compactFile $K4GEO/FCCee/CLD/compact/CLD_o2_v07/CLD_o2_v07.xml \
      --steeringFile cld_steer.py

# 2. Reconstruction (Pandora PF + tracking via k4run)
k4run CLDReconstruction.py \
      --inputFiles wzp6_ee_mumuH_ecm240_SIM.root \
      --outputBasename wzp6_ee_mumuH_ecm240_CLD \
      --num-events -1

# 3. Inspect
podio-dump wzp6_ee_mumuH_ecm240_CLD_REC.edm4hep.root | head -60
```

This produces `*_REC.edm4hep.root` (the EDM4hep file the histmaker reads)
plus `*_aida.root` (diagnostic histograms from the reconstruction).

Pin the Key4hep release when the analysis is locked — the tutorial
references `CLD_o2_v07` but reconstruction options and collection names
drift across nightly releases.

## Plot style

FCC-ee figures use mplhep's `CMS` stylesheet as the base, with the
experiment label set to one of `"FCC-ee"`, `"FCCee Delphes"`, or
`"FCCee CLD"` depending on what was run:

```python
import mplhep as mh
mh.style.use("CMS")
mh.label.exp_label(
    exp="FCC-ee",
    data=True,
    llabel="Delphes Simulation",     # or "CLD Simulation" for full sim
    rlabel=r"$\sqrt{s} = 240$ GeV, 10.8 ab$^{-1}$",
    loc=0,
    ax=ax,
)
```

`rlabel` is the place to put the centre-of-mass energy and integrated
luminosity. Never use the `com=` argument — the CMS stylesheet prints
the value in TeV, which is wrong for FCC-ee. Always set `rlabel`
explicitly as a string.

When both chains appear on the same figure (the small comparison panel
at the end of an analysis), use `llabel="Delphes vs CLD"` and
distinguish by line colour / marker. See `appendix-plotting.md` for the
full template.

## Required systematic sources (FCC-ee studies)

Most FCC-ee analyses are performed on simulation only (no real data).
The systematic programme reflects this — there are no JES/JER from data,
no pile-up, no trigger efficiency from tag-and-probe. The standard sources are:

| Source | How to evaluate |
|--------|-----------------|
| ISR modelling | Rerun with and without explicit ISR in the generator steering, OR compare Pythia8 ISR shower variations. |
| Generator choice | Compare the nominal generator (usually Pythia8) to at least one alternative (Whizard, KKMCee, Sherpa) at particle level. |
| Beam energy spread | Vary the gaussian beam spread input by its design uncertainty (order 1.3e-3 at 91 GeV, 2e-3 at 240 GeV). |
| Luminosity | Use the FCC-ee projected luminosity uncertainty (order 1e-4 at 91 GeV, 1e-3 at 240 GeV). This is a scale nuisance on every process, 100% correlated across processes. |
| Detector simulation | If both Delphes and CLD samples are available, the **full-sim vs fast-sim difference** on the observable is the natural systematic. See "Dual-sim comparison" below. |
| 4-fermion background modelling | At $\sqrt{s} \geq 161$ GeV, WW and ZZ 4-fermion backgrounds are irreducible. Vary their cross sections by PDG uncertainty. Irrelevant at the Z pole. |

## Dual-chain comparison (primary goal of the framework)

**The same physics analysis is reproduced on both simulation chains —
Delphes fast sim and CLD/IDEA full sim — and the final results are
compared.** This is not an appendix and it is not optional. Every
analysis in this repository ends with two independent physics results
(one per chain) and a formal comparison between them. See
`methodology/04-staged-validation.md` §4.3 for the execution protocol
(primary/secondary chain designation, per-phase scope, review tiers,
single-chain disclaimer rules); this section documents the technical
pattern.

### Chain-portable code

Keep the physics graph (`build_graph`) chain-independent and isolate the
chain differences in the aliasing shim shown in "Collection aliasing"
above. Each chain has a thin one-page run script
(`run_fastsim.py` / `run_fullsim.py`) that sets `processList`,
`inputDir` / `prodTag`, and `sim_chain`, then calls into the shared
`build_graph`. The Phase 3 histmaker, Phase 4 statistical model, and
Phase 5 AN production all run on both chains with no code duplication
— only the run scripts and the chain-stamped output directories differ.

### Comparison outputs

At Phase 4c, after both chains have produced their full-stats fits, the
executor produces `COMPARISON_dual_sim.md` in
`phase4_inference/4c_fullstats/outputs/`. Mandatory contents (detailed
in §4.3.5 of `methodology/04-staged-validation.md`):

1. **Fitted-parameter table** — Delphes central value ± uncertainty
   vs. CLD central value ± uncertainty, difference, pull.
2. **Systematic budget side-by-side** — per-source relative
   uncertainty on the fitted parameter for each chain.
3. **Covariance matrix comparison** — bin-to-bin covariance from each
   chain with a per-element relative difference (unfolded measurements).
4. **Distribution overlays with ratio panels** — observable + every
   MVA input / correction input, Delphes vs. CLD with a ratio panel.
5. **Verdict** — chi² or combined-pull summary, physical interpretation.

The distribution-overlay step (#4) is the "classic" dual-sim plot set.
It is a required input to the comparison but not the whole deliverable
— the final fitted parameter and its uncertainty budget are the
comparison's primary output.

### Physical expectations

Differences between chains are physically expected and informative — the
point of the comparison is to document their size, not to treat them as
failures:

- A few-percent shift in the recoil mass peak position is typical
  between Delphes parameterised smearing and CLD full tracking.
- Tracking tails and PFA mis-assignment that Delphes cannot model can
  change the relative importance of systematic sources.
- PID efficiency edge effects at low momentum typically differ between
  the two chains.

A difference is concerning only when it breaks the analysis design
(e.g., a cut defined on Delphes no longer separates signal from
background in CLD) or when it is larger than the assigned
detector-simulation systematic. In either case the Phase 4c arbiter
should trigger a Phase 3 regression.

### Example

See `analyses/zh_recoil_mumu/` for the worked dual-chain example
— `phase3_selection/src/build_graph.py` (shared physics graph),
`phase3_selection/src/run_fastsim.py` and `run_fullsim.py` (thin run
scripts), and `phase4_inference/4c_fullstats/outputs/COMPARISON_dual_sim.md`
(final-result comparison).

## References

- FCCAnalyses documentation: <https://hep-fcc.github.io/FCCAnalyses/>
- FCC full-sim general overview: <https://hep-fcc.github.io/fcc-tutorials/main/full-detector-simulations/FCCeeGeneralOverview/FCCeeGeneralOverview.html>
- FCC physics events portal (sample catalog): <http://fcc-physics-events.web.cern.ch/fcc-physics-events/>
- Key4hep project: <https://key4hep.github.io/key4hep-doc/>
- FCCPhysics example analyses (Jeyserma): <https://github.com/jeyserma/FCCPhysics/tree/main/analyses>
- Model-independent ZH cross section at FCC-ee, arXiv:2512.21290 —
  reference for the ZH recoil measurement implemented in this repository.
