# GLMs vs Neural Networks in Practice — IDI Pricing

This repository supports a practical actuarial case study: pricing an Individual Disability Income (IDI) policy using a frequency–severity structure and comparing **GLMs** (logit + Tweedie) to **FFNNs** (embeddings + governance controls), and hybrid approaches.

The workflow is designed so you can (a) iterate modularly during development, and (b) generate a “committee-ready” evidence pack on demand.

---

## Where the Data Lives

**GitHub (source of truth for code):**
- Notebooks and Python modules

**Do not commit full CSV datasets to GitHub** unless they are small samples.

---

## Required input files

The notebooks expect these two files:
- `idi_exposure_synth_v1.csv`
- `idi_claims_synth_v1.csv`

---

## Notebook roles (what each one does)

- **02_data_profile.ipynb**  
  Data profile + validation setup only (no modelling). Produces sanity table, time split timeline, incidence-by-month, severity histogram, workflow schematic.

- **03_glm_diagnostics.ipynb**  
  GLM modelling + diagnostics (logit incidence + Tweedie severity), plus figures/tables.

- **04_ffnn_diagnostics.ipynb**  
  FFNN modelling + diagnostics (frequency + severity), plus figures/tables.

- **05_scorecard_from_artifacts.ipynb**  
  Builds the **comparison scorecard + hybrid attribution rows** by reading exported “scored” CSV artifacts (no re-training).

- **00_master_runner.ipynb**  
  One-click runner that generates a versioned evidence pack (figures, scored datasets, scorecard, run metadata).

---

## Two supported workflows (choose one)

### Workflow A — Modular (recommended during development)
Use this to iterate quickly and keep responsibilities separated.

1) Run **02** to validate data and produce the profiling visuals  
2) Run **03** to fit GLMs and generate GLM visuals  
3) Run **04** to fit FFNNs and generate FFNN visuals  
4) Ensure **03 and 04 export artifacts** (see next section)  
5) Run **05** to create the scorecard and hybrid attribution rows from artifacts

This workflow is best when you’re tuning modelling, plots, or assumptions.

---

### Workflow B — One-click evidence pack (recommended for governance/committee)
Use this to generate everything consistently with run metadata.

1) Run **00_master_runner.ipynb**  
2) Collect outputs from the run folder:
   `outputs_artifacts/run_YYYYMMDD_HHMMSS/`

This workflow is best when you want a clean, reproducible output bundle with timestamps and environment fingerprinting.

---

## Artifact export (required for Notebook 05 in Workflow A)

Notebook 05 reads four CSVs from `outputs_artifacts/` by default:

- `glm_test_exposure_scored.csv`
- `glm_test_claims_scored.csv`
- `ffnn_test_exposure_scored.csv`
- `ffnn_test_claims_scored.csv`

To make Workflow A work perfectly, append the provided **export cells** to the end of:
- Notebook **03** (exports GLM scored exposure + claims)
- Notebook **04** (exports FFNN scored exposure + claims)

### Expected schema (what Notebook 05 assumes)

Scored exposure (GLM and FFNN):
- `life_id, month_start, exposure_months, onset, claim_id, p_hat, sev_hat, exp_cost`

Scored claims (GLM and FFNN):
- `claim_id, life_id, month_start, total_paid_ultimate, sev_hat`

If you change column names, update Notebook 05 accordingly.

---

## Outputs: where files are written

- Most notebooks write images/tables into a folder like:
  - `outputs_figures/` (for slide-ready PNGs)
  - `outputs_artifacts/` (for scored CSVs + scorecards)

- The master runner writes to a run-stamped folder:
  - `outputs_artifacts/run_YYYYMMDD_HHMMSS/`

Tip: For persistence in Colab, set output folders to Google Drive paths (e.g., `OUT_DIR = f"{DRIVE_ROOT}/<course_folder>/outputs_artifacts"`).

---

## Google Colab + Drive (recommended setup)

In each notebook, use the provided block:

1) Mount drive:
```python
from google.colab import drive
drive.mount('/content/drive')
```

2) Set:
```python
DRIVE_ROOT = "/content/drive/MyDrive/<YOUR_SHARED_FOLDER>"
```

3) Point the CSV paths to Drive:
```python
EXPOSURE_CSV = f"{DRIVE_ROOT}/02_Data/raw/idi_exposure_synth_v1.csv"
CLAIMS_CSV   = f"{DRIVE_ROOT}/02_Data/raw/idi_claims_synth_v1.csv"
```

---

## Governance checklist (minimum viable)

Before sharing results internally:
- Use a time split (already implemented)
- Confirm calibration-in-the-large for incidence (mean p vs observed)
- Confirm severity scale/back-transform is correct (log1p + smearing + mean calibration for FFNN)
- Review tail metrics (P90+ MAE)
- Produce portfolio and segment O/E (region × season)
- Save scored datasets + scorecard CSV + run metadata

---

## Quickstart

If you want the fastest path to selected results:
- Run **00_master_runner.ipynb**
- Assets stored in `outputs_artifacts/run_YYYYMMDD_HHMMSS/`

If you are iterating:
- Run **02 → 03 → 04**, append export cells, then run **05**.

---

