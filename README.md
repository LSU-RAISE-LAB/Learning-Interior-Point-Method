# L-IPM: Learning Interior-Point Central-Path Projection for OPF

Code and trained models for *"Learning Interior Point Method Central Path Projection for Optimal Power Flow"*.
L-IPM runs MATPOWER's MIPS interior-point solver for the first `K = 3` Newton iterations, lets a
grid-informed LSTM (GI-LSTM) project the iterate toward the converged solution, screens the projection
for feasibility, and lets MIPS finish from the projected point.

## Requirements

* MATLAB (tested with R2023b) and MATPOWER 7.0.
* Python 3.11 with `tensorflow==2.21`, `numpy`, `scipy`, `pandas`, `h5py`, `openpyxl`, `matplotlib`
  (e.g. `conda create -n lipm python=3.11 && conda activate lipm && pip install tensorflow==2.21 numpy scipy pandas h5py openpyxl matplotlib`).
* All MATLAB scripts locate the repository through `mfilename('fullpath')`; the only machine-specific
  setting is the Python interpreter, read from the environment variable `LIPM_PYTHON`
  (e.g. `C:\...\envs\lipm\python.exe`) or edited at the top of the script.

## What is in the folder

| `lib/`, `mips/` | MATPOWER 7.0 library and MIPS solver. |
| `mips.m` | logs every iterate through `ipm_logger`, contains the L-IPM hook after iteration `K`, and records per-iteration wall time. |
| `ipm_logger.m` | Collects the IPM iterates during data generation and during L-IPM runs. |
| `lipm_project.m` | The hook body: sends the first `K` iterates and the load vector to Python, receives the projected state, applies the acceptability screen, and either replaces the MIPS iterate or falls back to classical IPM. |
| `lipm_infer.py` | Python side of the bridge: loads the trained GI-LSTM once, standardizes inputs, runs the forward pass, clips/floors the output, and returns the screen residuals. Set `SYSTEM`, `OPF`, `MODEL_FILE` at the top to pick the model. |
| `case3.m`, `case24_ieee_rts.m`, `case118.m`, `case2869pegase.m` | Test systems (MATPOWER format). |
| `Data_Generation.m` | Generates the training data: Latin-hypercube load scenarios (seed 42), classical MIPS solves, first iterates + converged solution per scenario. |
| `export_case_data.m` | Writes `<system>_<opf>_meta.mat` (network data and MIPS variable layout) needed by the Python code. |
| `gi_lstm_<system>_<opf>.py` | GI-LSTM training script for one system/formulation; running it with no arguments reproduces the deployed model (`gilstm_<system>_<opf>/`). |
| `gilstm_<system>_<opf>/` | Trained model: `gi_lstm.weights.h5`, `scalers.npz`, `results.json` (metrics, architecture, training time), `<tag>_lstm_pred.mat` (test-split predictions and run ids). |
| `IPM_vs_LIPM_24bus.m` | Classical IPM vs L-IPM comparison. |
| `timeing_periter.m` | Wall time of every Newton iteration of classical MIPS (per-iteration timing table). |
| `bench_infer.py` | Pure GI-LSTM inference latency (graph-compiled forward pass, median of 300 calls). |
