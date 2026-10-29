# REFIT Washing-Machine Analysis and Disaggregation

**Flock Energy — Junior Data Scientist take-home assignment**

A reproducible analysis of REFIT electricity measurements: raw House 1 data cleaning (Part 1), washing-machine usage across four households (Part 2), and estimation of washing-machine power from whole-house demand (Part 3). A short utility-facing proposal is in [`Recommendation.md`](Recommendation.md) (Part 4).

## Results at a glance

- **Part 1:** 8,518,763 raw House 1 rows inspected; 1,211,048 extra duplicate-timestamp rows; 3,335 out-of-range readings flagged; a 1-minute output grid with long outages retained as missing.
- **Part 2:** 307 **candidate** washing-machine events across Houses 1, 2, 3 and 5 (January–March 2015), identified by a transparent rule rather than confirmed cycle labels.
- **Part 3:** On validation House 5, Random Forest was selected from an always-off baseline, Ridge, Random Forest and HistGradientBoosting (F1 **0.322**). On retrospective Houses 7 and 10 its F1 scores were **0.533** and **0.427**. Daily energy errors (**40.9%** and **75.7%**) limit utility applications.
- **Part 3 — Seq2Point CNN:** The second Part 3 notebook trains a 99-minute Seq2Point CNN and compares it with the saved Random Forest on matched minutes. The CNN reduced false positives and improved F1 on Houses 7/10, but **performed worse on validation House 5**. Random Forest remains the validation-selected final estimator. The CNN requires future context and is an **offline** method.

Detailed tables, figures and limitations: [`RESULTS.md`](RESULTS.md). Data-quality decisions: [`DATA.md`](DATA.md).

## Project layout

```text
.
├── notebooks/
│   ├── House1_Part1_Cleaning.ipynb
│   ├── WashingMachine_Part2_EDA.ipynb
│   ├── WashingMachine_Part3_Model.ipynb
│   └── WashingMachine_Part3_Seq2Point.ipynb   # Part 3: CNN training and comparison
├── raw/                         # download: two House 1 raw CSV parts
├── clean/                       # download: official CLEAN_House*.csv files
├── outputs/                     # generated intermediate analysis outputs
├── figures/                     # generated figures referenced by RESULTS.md
├── results/                     # model metrics and CNN comparison; RF bundle generated locally, not tracked
├── README.md
├── DATA.md
├── RESULTS.md
├── Recommendation.md
├── requirements.txt
└── .gitignore
```

The notebook filenames above are the intended **submission filenames**: remove upload/download suffixes such as `(4)`, `(5)` and `(6)` when arranging your local repository. Notebook code supports running from `notebooks/` or the project root. Keep `RESULTS.md` in the project root so figure links resolve.

## Data and setup

1. Use **Python 3.11** (recommended) and install dependencies:
   ```bash
   python -m venv .venv
   # Activate .venv using the command for your operating system.
   python -m pip install -r requirements.txt
   ```
2. Download the original REFIT House 1 data and the **official cleaned** REFIT CSV files from the [University of Strathclyde raw release](https://pureportal.strath.ac.uk/en/datasets/refit-electrical-load-measurements/) and [cleaned release](https://pureportal.strath.ac.uk/en/datasets/refit-electrical-load-measurements-cleaned/) (also available as [individual cleaned CSVs on Zenodo](https://zenodo.org/records/5063428)). The dataset is licensed **CC BY 4.0**; cite the source.
3. Extract the two required House 1 raw files as `raw/RAW_House1_Part1.csv` and `raw/RAW_House1_Part2.csv`. Put `CLEAN_House1.csv`, `CLEAN_House2.csv`, `CLEAN_House3.csv`, `CLEAN_House5.csv`, `CLEAN_House7.csv`, `CLEAN_House8.csv`, `CLEAN_House9.csv` and `CLEAN_House10.csv` in `clean/`.
4. Open Jupyter with `jupyter lab` and use **Restart Kernel & Run All** in this order:
   1. `notebooks/House1_Part1_Cleaning.ipynb` — Part 1 raw-data audit and figures (separate from modelling data).
   2. `notebooks/WashingMachine_Part2_EDA.ipynb` — Part 2 figures, 1-minute exports in `outputs/part2/`, and the Part 2 section of `RESULTS.md`.
   3. `notebooks/WashingMachine_Part3_Model.ipynb` — Part 3 model comparison, evaluation figures, `results/evaluation_metrics.csv`, and the locally generated, ignored `results/washing_machine_model.joblib`.
   4. `notebooks/WashingMachine_Part3_Seq2Point.ipynb` — the **second Part 3 modelling notebook**. It loads the saved Random Forest, trains the Seq2Point CNN, and creates matched-household comparison metrics and figures. **Run Part 3 Model first**; this notebook depends on its saved model.

Part 3 can also load official cleaned CSVs directly when relevant Part 2 exports are absent, using the notebook's documented preprocessing. Part 1 uses **raw** House 1, while Parts 2 and 3 use **official cleaned** REFIT; Part 1's manual cleaning is **not** substituted for official cleaned model targets. The official cleaned files are approximately 8-second samples; the notebooks average available values within each minute.

**Data are not bundled in this repository.** Large raw/cleaned CSVs and generated intermediates are excluded via `.gitignore`; download them before reproducing results. Computational time and exact TensorFlow predictions can depend on hardware/software versions. Seq2Point was rerun in a fresh Python 3.11 virtual environment on Windows; all nine of its code cells have execution counts. All four notebooks completed in README order from the project root in a fresh Python 3.11 virtual environment on Windows.

## Method and decisions

**Cleaning (Part 1):** sort by Unix timestamp, reconcile duplicates using the last non-null reading per channel, flag negative/extreme power, average into minutes, interpolate only 1–2 minute missing runs with valid neighbours and no rejected-only observations, and leave long outages missing. These assumptions and raw-data limitations are in `DATA.md`.

**Usage analysis (Part 2):** four homes; mean minute power; active >20 W; join at most 10 quiet minutes without crossing missing samples; accept candidate events meeting duration/active-minute/peak criteria. Plots show a representative day, example cycle, time patterns and household differences. REFIT `Issues` flags are reported rather than treated as confirmed ground truth errors.

**Disaggregation (Part 3, first modelling notebook):** fixed January–March 2015 period. Train on Houses **2/3/8/9**, validate on **5**, and retrospectively evaluate on **7/10**. Twelve aggregate/calendar features use only current and past minutes, with a complete 61-minute history. Compare always-off, scaled Ridge, Random Forest and HistGradientBoosting on the same eligible samples; tune a small number of settings using the validation home only. Select by validation F1 subject to baseline RMSE/daily energy-error guards. Report MAE/RMSE, active-minute precision/recall/F1, energy bias, daily energy error, and observed failures. The house-level split avoids mixing windows from the same home across train/test.

**Seq2Point CNN (Part 3 second modelling notebook):** 99-minute centred windows; trained using the same four training homes; validation-only early stopping and post-processing selection. It needs 49 minutes of future context. Optional gap/duration post-processing was tested but *not adopted* (selected thresholds both zero). CNN did better on some retrospective homes but worse on validation F1; do not select it because its retrospective plots look better.

The helper functions are intentionally copied from the Part 3 Model notebook so each notebook runs standalone.

## Limitations and responsible interpretation

Some appliances have similar load signatures; a high aggregate pulse is not proof of washer activity. The cleaned release has sensor asynchrony, forward-filled values and zero replacements, and some periods where monitored appliance totals exceed aggregate demand. Zero/off readings cannot always be independently verified. The study covers a selected winter/early-spring quarter, few UK households and previously examined evaluation homes. Results are **not** a prospective blind test, accurate billing data, measured monetary savings, or evidence of applicability to Indian homes without local validation. If only 15/30-minute meter readings were available, short signatures and overlapping loads would be harder to separate.

See [`Recommendation.md`](Recommendation.md) for a consent-based *pilot proposal*, not a demonstrated utility intervention.

## Possible improvements

- Evaluate on more households and a prospective time-based holdout.
- Save per-minute predictions and confirm detected cycles with appliance-level measurements before operational use.

## Sources and AI-use disclosure

- Murray et al., [REFIT data descriptor](https://doi.org/10.1038/sdata.2016.122) (*Scientific Data*, 2017); [official REFIT cleaned dataset](https://pureportal.strath.ac.uk/en/datasets/refit-electrical-load-measurements-cleaned/) (CC BY 4.0).
- Zhang et al., [Sequence-to-Point Learning With Neural Networks for Non-Intrusive Load Monitoring](https://doi.org/10.1609/aaai.v32i1.11873) (AAAI, 2018). Used as architectural inspiration; no published score is represented as our result.
- The assignment PDF and supplied cleaned-REFIT README were used to determine task requirements, mappings, limitations and validation choices.

**AI disclosure:** ChatGPT assisted with review, literature discussion and drafting/editing documentation. Dataset preparation, model choice, results and limitations are documented to support independent reproduction.