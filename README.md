# 36120-26SP-group25-experiments

<a target="_blank" href="https://cookiecutter-data-science.drivendata.org/">
    <img src="https://img.shields.io/badge/CCDS-Project%20template-328F97?logo=cookiecutter" />
</a>

**AT1 — Predicting sustained NBA careers from US college basketball statistics**

Advanced Machine Learning Application (36120), UTS, 2026 Spring.

| | |
|---|---|
| Student | Muhammad Asif |
| Student ID | 25422306 |
| Group | 25 |

---

## Result

| Measure | Value |
|---|---|
| Kaggle public leaderboard (AUPRC) | **0.60409** |
| Player-level cross-validated AUPRC | 0.5088 |
| Row-level cross-validated AUPRC | 0.4142 |
| Random baseline (class prevalence) | 0.0344 |
| Lift over random | **17.6×** |

**Final model:** logistic regression without class weighting, inside a
preprocessing pipeline (median imputation → standardisation → one-hot encoding),
with predictions aggregated to one row per player using the most recent season.
At a 60-player shortlist it achieves 86.7% precision against a 3.37% base rate.

---

## Setup

Requires Python 3.12.13 and [uv](https://docs.astral.sh/uv/).

```bash
uv python pin 3.12.13
uv sync
```

Dependencies are pinned in `pyproject.toml` and `uv.lock`; `requirements.txt`
is provided for pip users.

## Data

Competition data is **not committed**. Download from the Kaggle competition
page and place in `data/raw/`:

```
data/raw/train.csv
data/raw/test.csv
data/raw/sample_submission.csv
data/raw/metadata.csv
```

If Kaggle names the metadata file differently, rename it to `metadata.csv` —
the notebook expects that filename.

## Running

```bash
uv run jupyter lab
```

Open `notebooks/36120-26SP-group25-25422306-AT1-experiment-1.ipynb` and run
**Kernel → Restart Kernel and Run All Cells**. Runtime is 10–15 minutes,
dominated by the cross-validated model comparison in section J.

Outputs: `models/submission_final.csv` (6,261 rows, one per player) and six
split CSVs in `data/processed/`.

---

## Method

**Validation.** Stratified group k-fold, five folds, grouped on `pid`. Grouping
is essential because one row is a player-season: a naive stratified split places
4,711 players on both sides of the first fold, letting the model recognise
individuals rather than generalise. The group-aware scheme reduces that to zero,
verified per fold.

**Preprocessing.** All transformations sit inside a scikit-learn `Pipeline`, so
imputation medians, scaler parameters and encoder categories refit on each
fold's training portion alone. Fitting before splitting would leak validation
information into every estimate.

**Feature engineering.** Height, class year and age at season parsed from text
columns; per-minute production rates; a flag marking absent recruiting scores.

## Key findings

1. **Grain mismatch beat model selection.** Rows are player-seasons (41,195 rows,
   17,938 players) but the target is per-player. Aggregating predictions to
   player level improved AUPRC by **+0.0946** — roughly ten times the spread
   between the best and worst of four algorithms.
2. **Class weighting hurt.** `class_weight="balanced"` reduced AUPRC from 0.4181
   to 0.3818 and destroyed calibration (mean predicted probability 0.2829 against
   a prevalence of 0.0344). AUPRC is rank-based; class weighting addresses
   calibration.
3. **Model families were equivalent within noise.** The best-versus-second gap
   (0.0246) barely exceeds one fold standard deviation (0.0220).
4. **Four data-quality defects documented**: season 2010 duplicated wholesale,
   35.8% of birthdates a placeholder, `Rec Rank` a percentile score rather than a
   rank, and `sample_submission.csv` containing 5,800 duplicate IDs.

## Related repository

**Python package:** `36120-26SP-group25-25422306-package` — the feature
engineering functions used here, published to TestPyPI as
`nba-college-features-25422306`.

## AI usage acknowledgement

Generative AI (Claude, Anthropic) was used to support code development, explain
machine learning concepts and review analytical reasoning. All code was executed
and validated by the author, all outputs independently verified, and all
analytical conclusions are the author's own.

---

## Project Organization

```
├── LICENSE            <- Open-source license if one is chosen
├── Makefile           <- Makefile with convenience commands like `make data` or `make train`
├── README.md          <- The top-level README for developers using this project.
├── data
│   ├── external       <- Data from third party sources.
│   ├── interim        <- Intermediate data that has been transformed.
│   ├── processed      <- The final, canonical data sets for modeling.
│   └── raw            <- The original, immutable data dump.
│
├── docs               <- A default mkdocs project; see www.mkdocs.org for details
│
├── models             <- Trained and serialized models, model predictions, or model summaries
│
├── notebooks          <- Jupyter notebooks.
│
├── pyproject.toml     <- Project configuration file with package metadata for
│                         nba_longevity and configuration for tools like black
│
├── references         <- Data dictionaries, manuals, and all other explanatory materials.
│
├── reports            <- Generated analysis as HTML, PDF, LaTeX, etc.
│   └── figures        <- Generated graphics and figures to be used in reporting
│
├── requirements.txt   <- The requirements file for reproducing the analysis environment
│
├── setup.cfg          <- Configuration file for flake8
│
└── nba_longevity      <- Source code for use in this project.
 