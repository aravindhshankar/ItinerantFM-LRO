# Code for the DMRG / iDMRG results of arXiv:2609.38581

This repository contains the code that generated the finite-size DMRG (Julia) and infinite DMRG (Python/TeNPy)
data of the paper, and the notebooks that produce the DMRG figures (Figs. 1, 2 and S4-S9). The field-theory
figures (S1-S3) are not part of this repository.

The code is shown as it was used. The scripts are in their final state (see "Provenance" below).

## Layout

```
LICENSE, Project.toml, Manifest.toml    Julia package Spin1LongRangeOrder (Julia 1.12.6)
src/                                    Julia package (Hamiltonian, I/O, helpers)
scripts/
  fixedNpScanHubbard.jl                 finite DMRG scan over Vpm (N = 64, 128, 256), writes MPS
  HubbardImprovePrecision.jl            re-convergence up to chi=1200 (output NOT included, no figure uses it)
slurm_scripts/                          SLURM jobs for the finite-DMRG scan and restarts
calculateFromData/
  driver.jl, correlations.jl            correlation functions from saved MPS -> data/Hubbard/corrs/*_corrs.h5
  runner.slurm
python/
  requirements.txt                      requirements as originally written
  requirements_lock.txt                 pip freeze of the environment used (Python 3.13.0, physics-tenpy 1.1.0)
  scanner_scanV_serial.py               iDMRG scan over Vpm at fixed chi (scanChi800/1000/1200/600)
  scanner_rev_fm.py                     reverse scan from the ferromagnet (rev/scanChi100/200/400)
  runner_scanV.slurm
  src/                                  iDMRG model (consNfextendedHubbard), scanners, observables
  utils/                                I/O helpers (io.py, julio.py reads the Julia .h5 files, slurmhelpers.py)
  central_charge/                       iDMRG chi-scans for central charge (Vpm3.8cc, Vpm4.350cc)
  gen_data/                             iDMRG correlation functions -> corrs/results.h5 and reader
  figmaker/                             figure notebooks, aps.mplstyle, set_size_helper.py, figures/ (output)
data/                                   NOT in git (on Zenodo); see "Data" below
```

## Environments

- Julia: `julia --project=.` from the repository root (Julia 1.12.6; `Manifest.toml` included).
  Julia jobs are run from the repository root.
- Python: Python 3.13.0, `python -m venv python/.venv && python/.venv/bin/pip install -r python/requirements_lock.txt`.
  Python jobs are run from `python/` (or from the subdirectory of the script, e.g. `python/figmaker` for the notebooks).
  Figures with LaTeX text need a working `latex` in the PATH.

## Pipeline

MPS are generated first, correlation functions are computed separately from the saved MPS.

Finite DMRG (Julia):
1. `slurm_scripts/scanHubbard.slurm` runs `scripts/fixedNpScanHubbard.jl`, which writes MPS to
   `data/Hubbard/N{64,128,256}consNf/` (restarts: `argo_restart_hubbard.slurm`).
2. `calculateFromData/runner.slurm` runs `driver.jl` (includes `correlations.jl`) and writes
   `data/Hubbard/corrs/*_corrs.h5`.
3. `python/utils/julio.py` reads these files in the notebooks.

iDMRG (Python / TeNPy, run from `python/`):
1. `runner_scanV.slurm` runs `scanner_scanV_serial.py` (writes `data/iDMRG/scanChi{800,1000,1200,600}`) and
   `scanner_rev_fm.py` (writes `data/iDMRG/rev/scanChi{100,200,400}`); both use `src/scanner_versions.py` and
   `src/consNfextendedHubbard.py`.
2. `central_charge/central_charge_Hubbard.py` writes `data/iDMRG/Vpm3.8cc`;
   `central_charge/fm_central_charge.py` (run by `runner_cc.slurm`) writes `data/iDMRG/Vpm4.350cc`.
3. `gen_data/driver_idmrg_datacollector.py` (via `runner_idmrg_datacollector.slurm`) and `gen_data/combiner.py`
   write `corrs/results.h5` in each data directory; `gen_data/read_corrs.py` reads them.

## Figures

All notebooks are in `python/figmaker/` (run with that directory as working directory; data paths are `../../data/...`).

| Paper | Notebook | Output PDF (in `figures/`) | Data |
|---|---|---|---|
| Fig 1 | `combine_LRO_C.ipynb` | `combined_LRO_C_v2.pdf` | Hubbard/corrs, iDMRG/scanChi800/corrs, iDMRG/Vpm4.350cc/corrs |
| Fig 2 | `ShowLRO.ipynb` | `QLROSplusCrit.pdf` | Hubbard/corrs, iDMRG/scanChi800/corrs |
| S4 | `ShowLisaScaling.ipynb` | `dV3_2isLL_widetext.pdf` | Hubbard/corrs, iDMRG/scanChi1000/corrs, iDMRG/Vpm3.8cc/*.h5 (MPS) |
| S5 | `DMRG_conv_show.ipynb` | `dmrg_diagnostics.pdf` | iDMRG/Vpm4.350cc/Vpm_4.350_chi1000.h5 (MPS) |
| S6 | `Continuous_PT_magz.ipynb` | `ShowMagAcrossCrit.pdf` | Hubbard/corrs, iDMRG/rev/scanChi{100,200,400}/corrs, iDMRG/scanChi{800,1000}/corrs |
| S7 | `show_phase_sep_fdmrg.ipynb` | `ShowPhaseSep.pdf` | Hubbard/corrs |
| S8 | `finite_ent_scaling_fit.ipynb` | `finite_ent_scalingv3.pdf` | iDMRG/Vpm4.350cc/corrs, iDMRG/scanChi600/corrs |
| S9 | `SzcorrelatorCrit.ipynb` | `SzCorrChiScaling.pdf` | iDMRG/Vpm4.350cc/corrs, iDMRG/rev/*, iDMRG/scanChi{800,1000,1200}/corrs |

The notebooks are stored with their outputs, exactly as run.

## Provenance caveat

The scripts are in their final state. Several datasets were produced with earlier edits of the same scripts
(different chi lists, restart points and seeds, as the commented-out lines show). The original git history is
in the development repository.

## Data

`data/` is not tracked in git. It is available on Zenodo as tar archives of the complete `data/Hubbard/` (finite DMRG) and
`data/iDMRG/` (iDMRG) directories; extracting them in the repository root restores the `data/...` paths. The archives
also contain additional data (exploratory runs not used in the paper). Only the paths listed below are read by the code
and the figure notebooks.

Required, finite DMRG:
- `data/Hubbard/N64consNf/`, `data/Hubbard/N128consNf/`, `data/Hubbard/N256consNf/` (MPS; Vpm 4.000-4.750 is used)
- `data/Hubbard/corrs/` (N64, N128, N256 files)

Required, iDMRG (MPS and `corrs/results.h5`):
- `data/iDMRG/scanChi800/`, `data/iDMRG/scanChi1000/`, `data/iDMRG/scanChi1200/`
- `data/iDMRG/scanChi600/` (its `corrs/results.h5` is read by `finite_ent_scaling_fit.ipynb`)
- `data/iDMRG/rev/scanChi100/`, `data/iDMRG/rev/scanChi200/`, `data/iDMRG/rev/scanChi400/`
- `data/iDMRG/Vpm4.350cc/`, `data/iDMRG/Vpm3.8cc/`

Optional, seeds and provenance:
- `data/iDMRG/Vpm6.0cc/` (seed `Vpm_6.000_chi400.h5` / `_chi_100.h5` for the reverse scans)
- `data/iDMRG/scanChi500/` (seed named in `central_charge_Hubbard.py`)
- `data/iDMRG/v2fwd/t_1.00_U_0.10_Vpp_0.800_consNf0_25_chimax_100/` (seed for Vpm3.8cc)

The seed `python/central_charge/Vpm3.8chi80.h5` (commented out in `central_charge/*.py`) is not included; it can be
regenerated by an iDMRG run at Vpm = 3.8 with chi = 80.

Not used by any figure: `data/Hubbard/ImpPrec/`, `data/Hubbard/corrs_imp/` (output of `HubbardImprovePrecision.jl`;
present in the Zenodo archive if it is in `data/Hubbard/`).
