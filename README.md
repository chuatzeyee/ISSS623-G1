# ISSS623-G1 - Group Project Workspace

SMU ISSS623 Applied Data Science in Healthcare (Jul-Aug 2026). Predictive modelling on the 2024 BRFSS dataset.

## Layout

| Folder | Contents |
|---|---|
| `data/` | BRFSS 2024 dataset (zipped ASCII), codebook (HTML + parsed CSV dictionary), prof's pre-extracted topic A/B/C CSVs (457,670 respondents each), extraction notebook, merged caregiver working dataset |
| `notebooks/` | `Data_Prep_Caregiver.ipynb` - reproducible pipeline: raw-file extraction → merge → recode → 86,676-case main analytic sample → 19,263-case caregivers-only subset → stratified 70/30 train/test indices |

## Data quick facts

- `LLCP2024ASC.zip` extracts to `LLCP2024.ASC` (922 MB fixed-width ASCII) - **do not commit the extracted file**; it exceeds GitHub's 100 MB limit. The prep notebook unzips it automatically.
- Topic extracts: A = 29 cols, B = 33, C = 37; all rows, unweighted, still contain 7/77 (don't know), 9/99 (refused), 88 (none) codes; recode rules are documented inside the prep notebook.
- Analytic samples (from the notebook): main 86,676 (event rate 18.2%, caregivers 22.2%); caregivers-only 19,263 (2,890 events, 15.0%).
- Key deadlines (corrected): proposal 25 Jul · Quiz 1 1 Aug · Quiz 2 + presentation 15 Aug · final + peer eval 21 Aug.
