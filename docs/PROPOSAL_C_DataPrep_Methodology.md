# Data Preparation and Data Methodology

**ISSS623 Group Project - Topic C (caregiver angle) · Owner: Data Preparation Lead**
*Every count and percentage below was computed directly from the 2024 BRFSS files listed in Section 1; nothing is quoted from a paper. Companion documents: `PROPOSAL_C_PreventiveCareGap.md` (full proposal) and `PROPOSAL_C_Rationale_Prose.md` (design rationale).*

---

## 1. Source data and provenance

| File | Source | Role |
|---|---|---|
| `LLCP2024ASC.zip` → `LLCP2024.ASC` (922 MB) | CDC BRFSS 2024 annual data (fixed-width ASCII, record length 2,111) | Raw source of truth; 457,670 respondents × 297 fields |
| `USCODE24_LLCP_082125.HTML` (from `codebook24_llcp-v2-508.zip`) | CDC | Official codebook: variable names, column positions, value labels, skip logic |
| `brfss2024_variable_dictionary.csv` | Course repo (prof-supplied) | Parsed codebook: variable, label, col_start, col_end, type |
| `brfss2024_topic_c.csv` (37 columns) | Course repo (prof-supplied) | Pre-extracted Topic C fields, all 457,670 respondents |
| `brfss2024_topic_c_caregiver.csv` (41 columns) | **Built by us** (Section 2) | Working dataset = topic C extract + 4 caregiver fields |

The raw ASCII file and all CSVs above 50 MB are excluded from the group GitHub repository via `.gitignore` (`*.ASC`); the dataset is shared through the group Drive, and the notebook reproduces the working dataset from the raw file deterministically.

## 2. Extraction and merge methodology

The focal variable `CAREGIV1` and the caregiver-detail items are optional-module fields absent from the prof's extract. We extracted them from the raw file as follows:

1. **Column positions** were taken from `brfss2024_variable_dictionary.csv`: `CAREGIV1` (col 325), `CRGVREL5` (326-327), `CRGVHRS2` (334), `CRGVLNG2` (335). These match the official codebook.
2. **Fixed-width read**: `pandas.read_fwf(LLCP2024.ASC, colspecs=[(324,325), ...])` - note the 0-indexed `colspecs` conversion from the 1-indexed dictionary (`col_start - 1`). This mirrors the logic in the course's `BRFSS_Extraction.ipynb`.
3. **Row-alignment verification before merge**: the raw-file extraction and the prof's `brfss2024_topic_c.csv` were compared on `_STATE` for all 457,670 rows; the match was exact (`assert (tc._STATE == cg._STATE).all()`), licensing a positional `concat` rather than a keyed join (BRFSS's `SEQNO` is not unique across states; `_STATE`-order verification is the appropriate alignment check).
4. **Output**: `brfss2024_topic_c_caregiver.csv`, 457,670 rows × 41 columns.

**Validity checks on the extraction:** (i) caregiver prevalence in the module sample is 22.2% - consistent with CDC's published ~1-in-5 estimate for US adults, which would not occur if column offsets were wrong; (ii) `CAREGIV1` value distribution (1: 21,137 · 2: 74,592 · 7: 241 · 8: 215 · 9: 102 · blank: 361,383) matches the codebook's expected code set exactly.

## 3. Universal recode rules

Applied consistently across all variables, per the 2024 codebook:

| Code pattern | Meaning | Treatment |
|---|---|---|
| 7 / 77 | Don't know / Not sure | → missing |
| 9 / 99 | Refused | → missing (exception: kept as an explicit category where prevalence makes dropping harmful - Section 5) |
| 88 in day-count items (`MENTHLTH`) | "None" (zero days) | → **0**, never missing |
| 8 in `CAREGIV1` | "Expects to provide care within two years" (n = 215) | → missing (documented; NOT "none" - same digit, different meaning than the 88 convention) |
| 8 in `CHECKUP1` | "Never had a check-up" | → **1** (counts as a care gap - a valid substantive value, not missing) |
| 88 in `PRIMINS2` | "No coverage of any type" (n = 5,649 in module sample) | → "uninsured" category (valid value) |
| Blank | Not asked (skip logic / module non-participation) | → structural missing, reported separately from item missingness |

## 4. Variable-by-variable recode dictionary

| Variable | Role | Original coding | Recode |
|---|---|---|---|
| CHECKUP1 | Primary outcome | 1 ≤1yr; 2 ≤2yr; 3 ≤5yr; 4 >5yr; 8 never; 7/9 | 1→0 (attended); 2/3/4/8→1 (gap); 7/9/blank→missing |
| MEDCOST1 | Secondary outcome + predictor in primary model | 1 yes; 2 no; 7/9 | 1→1; 2→0; 7/9→missing |
| CAREGIV1 | Focal predictor | 1 yes; 2 no; 8 expects-within-2yrs; 7/9 | 1→1; 2→0; 7/8/9→missing |
| PRIMINS2 | Predictor | 1-10 insurance types; 88 none; 77/99 | Grouped: employer(1)/private(2)/Medicare(3,4)/Medicaid-state(5,6,9)/military-IHS(7,8)/other-gov(10)/uninsured(88)/not-reported(77,99) |
| PERSDOC3 | Predictor | 1 one provider; 2 more than one; 3 none; 7/9 | 1/2→1; 3→0; 7/9→missing |
| GENHLTH | Predictor | 1-5 excellent→poor; 7/9 | Ordinal 1-5; 7/9→missing |
| MENTHLTH | Predictor | 1-30 days; 88 none; 77/99 | 88→0; 1-30 kept; 77/99→missing |
| DIABETE4 | Predictor | 1 yes; 2 pregnancy-only; 3 no; 4 pre-diabetes; 7/9 | 1→1; 2/3/4→0; 7/9→missing |
| _MICHD | Predictor | 1 yes; 2 no; blank | 1→1; 2→0; blank→missing |
| DIFFWALK / DECIDE / DIFFALON | Predictors | 1 yes; 2 no; 7/9; blank | 1→1; 2→0; else missing |
| _SMOKER3 | Predictor | 1/2 current; 3 former; 4 never; 9 | current/former/never; 9→missing (only 0.63% in module sample) |
| _TOTINDA | Predictor | 1 active; 2 inactive; 9 | 1→1; 2→0; 9→missing |
| SEXVAR | Predictor | 1 M; 2 F | As is (0 missing) |
| _AGEG5YR | Predictor | 1-13 five-year bands; 14 unknown | Bands 1-13; 14→missing |
| _RACEGR3 | Predictor | 1-5 groups; 9 | Groups 1-5; 9→missing |
| _EDUCAG | Predictor | 1-4 levels; 9 | Levels 1-4; 9→missing |
| _INCOMG1 | Predictor | 1-7 bands; 9 not reported | Bands 1-7 + **"not reported" kept as category** (16.65% - Section 5) |
| EMPLOY1 | Predictor | 1-8 statuses; 9 refused | Grouped: employed(1,2)/unemployed(3,4)/homemaker-student(5,6)/retired(7)/unable(8); 9/blank→missing |
| CRGVREL5, CRGVHRS2, CRGVLNG2 | **Descriptive only - never predictors** | Asked only if CAREGIV1=1 | Caregiver-profile table only; using them as predictors would leak caregiver status (Section 7) |

Encoding for modelling: nominal variables (`PRIMINS2` groups, `EMPLOY1` groups, `_RACEGR3`, marital excluded) one-hot encoded with a reference level; ordinal variables (`GENHLTH`, `_AGEG5YR`, `_EDUCAG`, `_INCOMG1` bands) kept as ordered integers; binary flags as 0/1.

## 5. Missingness handling framework and measured table

**Decision rule:** variables with invalid rates **< 5%** within the module sample → complete-case exclusion (unbiased at these magnitudes, keeps the design simple and defensible per the brief's "a good project does not need a sophisticated missing-data method"). Variables **≥ 5%** → the invalid responses are retained as an explicit "not reported" category, because deletion at that scale both discards data and biases the sample (non-response correlates with the construct itself, e.g. income refusal correlates with socioeconomic position).

**Measured missingness within the 96,287-respondent module sample** (invalid = don't know/refused/blank per Section 4 rules):

| Variable | Role | Invalid n | Invalid % | Decision |
|---|---|---:|---:|---|
| CHECKUP1 | Outcome | 1,080 | 1.12% | Exclude row (missing outcome) |
| CAREGIV1 | Focal predictor | 558 | 0.58% | Exclude row |
| MEDCOST1 | Sec. outcome/predictor | 274 | 0.28% | Exclude row |
| PRIMINS2 | Predictor | 3,245 | 3.37% | **Keep 77/99 as "not reported" category** (borderline; retains the uninsured-adjacent refusers) |
| PERSDOC3 | Predictor | 838 | 0.87% | Exclude row |
| GENHLTH | Predictor | 256 | 0.27% | Exclude row |
| MENTHLTH | Predictor | 1,620 | 1.68% | Exclude row (after 88→0) |
| DIABETE4 | Predictor | 186 | 0.19% | Exclude row |
| _MICHD | Predictor | 1,049 | 1.09% | Exclude row |
| DIFFWALK | Predictor | 401 | 0.42% | Exclude row |
| DECIDE | Predictor | 605 | 0.63% | Exclude row |
| DIFFALON | Predictor | 315 | 0.33% | Exclude row |
| _SMOKER3 | Predictor | 611 | 0.63% | Exclude row (module-sample rate is low; national 7% rule-of-thumb does not apply here) |
| _TOTINDA | Predictor | 233 | 0.24% | Exclude row |
| SEXVAR | Predictor | 0 | 0.00% | - |
| _AGEG5YR | Predictor | 1,282 | 1.33% | Exclude row |
| _RACEGR3 | Predictor | 1,598 | 1.66% | Exclude row |
| _EDUCAG | Predictor | 338 | 0.35% | Exclude row |
| **_INCOMG1** | Predictor | **16,035** | **16.65%** | **Keep code 9 as "not reported" category - too large and too non-random to drop** |
| EMPLOY1 | Predictor | 798 | 0.83% | Exclude row |

**Structural missingness reported separately:** 361,383 respondents (79.0% of the full file) were never asked `CAREGIV1` because their state did not field the Caregiver module. These are not non-respondents and do not appear in the missingness table above; they define the sampling frame, not item missingness. The distinction is stated explicitly in the report.

Caregiver-detail completeness among the 21,137 caregivers (for the descriptive profile): `CRGVREL5` 99.9% valid, `CRGVHRS2` 98.7%, `CRGVLNG2` 98.5%.

## 6. Cohort construction (data-flow)

```
457,670  full 2024 BRFSS file
   └─ keep respondents asked CAREGIV1 (16 module states)
96,287   module sample (21.0% of file)
   └─ drop invalid CAREGIV1 (558) and invalid CHECKUP1, overlapping
94,670   valid caregiver status + valid primary outcome
   └─ drop rows invalid on any complete-case predictor (Section 5)
87,098   FINAL ANALYTIC SAMPLE (92.0% within-module retention)
```

Final-sample characteristics (verified): outcome event rate **18.2%** (unchanged from the full national file - restricting to module states did not distort the outcome distribution); caregivers **22.2%** (20,976); ~15,800 outcome events across 19 predictors → **>4,000 events per predictor**, so complete-case power is a non-issue. The secondary-outcome analysis (MEDCOST1 as target) reuses the same cohort minus its own invalid outcome rows, with MEDCOST1 removed from that model's predictor set.

The 16 module states (FIPS): 9 CT, 10 DE, 17 IL, 21 KY, 22 LA, 28 MS, 32 NV, 33 NH, 37 NC, 41 OR, 46 SD, 48 TX, 49 UT, 51 VA, 55 WI, 56 WY.

## 7. Leakage prevention and separation of concerns

1. **Caregiver-intensity items** (`CRGVHRS2`, `CRGVREL5`, `CRGVLNG2`) are asked only of caregivers; any model containing them trivially reconstructs `CAREGIV1`. They are quarantined to the caregiver descriptive profile.
2. **MEDCOST1 double duty** is cleanly separated: predictor in the primary (CHECKUP1) model; target of the separate secondary model, from whose predictor set it is removed.
3. **No target-derived features**: no variable derived from `CHECKUP1` enters any predictor set.
4. **Split-before-fit**: the stratified 70/30 train/test split (fixed `random_state`) happens after cohort construction and before any model fitting, descriptive-informed feature tweaking, or threshold selection. All recodes in Section 4 are codebook-driven (not data-driven), so applying them before the split introduces no leakage. The test set is evaluated exactly once, for the final comparison table.

## 8. Reproducibility and handoff

- One canonical recode notebook produces the analytic dataframe; all members import it - no member-specific recodes.
- Every transformation is codebook-cited in comments; the variable dictionary (Section 4) is the single source of truth across proposal, notebook, report and slides.
- Deliverables from data prep to the rest of the team: (i) the analytic dataframe (87,098 × 21 recoded columns), (ii) the missingness table (Section 5), (iii) the data-flow diagram (Section 6), (iv) train/test indices, (v) the caregiver-profile descriptive table inputs.
- Required limitation statement carried through all artefacts: *"This project is an unweighted respondent-level predictive modelling exercise using the 2024 BRFSS analytic sample. The results should not be interpreted as nationally representative prevalence estimates."* Extended: findings describe the 16 Caregiver-module states and may not generalise to other states.
