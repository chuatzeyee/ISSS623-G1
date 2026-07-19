# Topic A — Predicting High Health Burden Among 2024 BRFSS Respondents

**ISSS623 Applied Data Science in Healthcare · Group Project Proposal Pack (Topic A)**
Status: candidate proposal (not the group's recommended pick — see rationale). All figures below were measured directly from `data/brfss2024_topic_a.csv` (457,670 rows) on the professor-supplied extract.

---

## PART 1 — Decision rationale (read this before the group meeting)

### 1.1 Why this outcome: poor/fair self-rated general health (GENHLTH)

- **It is THE canonical CDC health-related quality of life (HRQOL) indicator.** GENHLTH ("Would you say that in general your health is excellent, very good, good, fair, or poor?") is the first of the CDC "Healthy Days" core measures (Moriarty, Zack & Kobau 2003) and the primary HRQOL indicator in Jiang & Hesser's BRFSS analysis (2006). No other outcome in any of the three topic files has a literature anchor this direct: both brief-cited Topic A papers model exactly this variable, and even the Topic C anchor (Clark et al. 2021) predicts self-rated health. We would never be arguing by analogy.
- **Measured event rate is nearly ideal for classification: 19.7%** of respondents with a valid answer report fair (4) or poor (5) health (90,113 of 456,360). After our planned exclusions the analytic-sample rate is 18.9%. This is comfortably away from both the degenerate extremes: common enough that sensitivity/AUPRC are estimable and models cannot win by predicting the majority class blindly, rare enough that the classification problem is non-trivial.
- **Measured missingness is essentially zero: 0.29%** (1,305 respondents coded 7 = don't know / 9 = refused, plus 5 blanks). The outcome definition is a one-line recode: 4/5 → 1, 1–3 → 0, 7/9/blank → excluded. This directly avoids the brief's first "common mistake" (outcome not checked in the codebook) — we checked it, in the actual file.
- **Strong, monotone signal exists in the data.** Poor/fair health rises from **7.5%** among respondents with none of the eight measured chronic conditions to **16.4%** (one condition), **30.1%** (two), **45.9%** (three) and **66.2%** (four or more). It runs **45.0% → 5.3%** across the income bands (<$15k vs ≥$200k) and is **37.0% vs 17.1%** for respondents with vs without a cost barrier to care. A predictive model has real structure to find, and the descriptive story will be coherent with the model story (rubric: interpretation consistent with descriptives).

### 1.2 Why the secondary outcome: frequent physical distress (PHYSHLTH ≥ 14 days)

- Frequent physical distress (FPD; ≥14 physically unhealthy days in the past 30) is the second Healthy Days measure, used by the CDC and by Jiang & Hesser as a companion indicator to poor/fair health. Measured: **14.3%** event rate among valid responses; 88 ("none") → 0 covers most respondents; only **2.4%** are 77/99 (don't know/refused) and blanks are negligible (5 rows). It reuses the same predictor set and pipeline — one extra model run, zero extra data engineering, satisfying the "1 primary + 1 secondary outcome" scope with minimal cost.
- **Why NOT POORHLTH (activity limitation) — explicit justification.** POORHLTH is only asked of respondents who reported ≥1 unhealthy day (physical or mental), so **41.4% of the file is blank by skip logic** (measured: 189,428 blanks). Using it as an outcome would either discard 4 in 10 respondents non-randomly (the healthiest ones, by design) or require an assumption-laden imputation of zeros. It fails the brief's feasibility test and we say so in the proposal — demonstrating exactly the codebook awareness the 3% data-feasibility criterion rewards.

### 1.3 Why these predictors (19, from a candidate pool of 24 in topic_a.csv)

The general question form requires demographic, behavioural, healthcare-access and chronic-condition domains. We audited all 24 plausible candidates in `topic_a.csv` and kept 19 — deliberately below the 20-predictor cap — cutting 5 with documented reasons:

| Domain | Kept (measured DK/refused/blank) | Rationale |
|---|---|---|
| Demographics / SES (5) | SEXVAR (0%), _AGE80 (0%), _RACEGR3 (2.0%), _EDUCAG (0.5%), _INCOMG1 (19.1% code 9) | Core equity axes in Jiang 2006 and Clark 2021. Income's 19.1% non-report is too informative to drop — we keep code 9 as an explicit "Not reported" level (non-response to income is itself socially patterned). |
| Health behaviours (4) | _SMOKER3 (7.0%), _RFBING6 (10.4%), _TOTINDA (0.3%), _BMI5CAT (9.4% blank) | The four classic modifiable risk factors; all are CDC calculated variables with clean documented codings. Smoker/binge/BMI unknowns ≥5% are retained as explicit "Not reported/Unknown" levels rather than deleted. |
| Chronic conditions (8) | DIABETE4 (0.2%), _MICHD (1.1% blank), CVDSTRK3 (0.3%), CHCCOPD3 (0.5%), ADDEPEV3 (0.6%), _CASTHM1 (0.9%), _DRDXAR2 (0.6% blank), CHCKDNY2 (0.4%) | The dominant drivers of self-rated health (see the 7.5%→66.2% gradient). All are simple yes/no recodes with <1.2% loss. |
| Healthcare access (2) | PERSDOC3 (1.0%), MEDCOST1 (0.4%) | Personal provider and cost barrier capture realised access; both nearly complete. |

**Cuts and why (this is the judgement the rubric wants):**
1. **EMPLOY1** — response 8 is "unable to work", which is close to a restatement of the outcome (reverse-causation / label-leakage risk). Cut to keep the model honest.
2. **MARITAL** — weak incremental signal once age, income and education are in; parsimony.
3. **CHECKUP1** — a care-seeking behaviour that is itself a Topic C outcome ("no routine check-up"), with an awkward extra code (8 = never); access is already represented.
4. **_HLTHPL2** — insurance status; conceptually overlaps PERSDOC3 + MEDCOST1 (which measure *realised* access), has 4.1% code 9, and in 2024 derives from the *renamed* PRIMINS2 item (the 2024 naming trap: older papers cite PRIMINSR/HLTHPN1) — one more reason to prefer the two clean direct items.
5. **_URBSTAT** — 3.2% blank and modest expected signal; retained as a *descriptive/subgroup* variable only (urban–rural comparison in Table 1), not as a model predictor.

**Chronic count vs individual flags — the trade-off, decided.** Collapsing the 8 condition flags into a single 0–8 count would save 7 predictor slots but destroy the most interesting interpretive question: *which* conditions carry the prediction (depression? COPD? arthritis?). We keep the 8 individual flags for feature-importance interpretability and compute the count only as a descriptive variable (the 7.5%→66.2% gradient chart). We have slack under the cap, so the slots cost nothing.

### 1.4 Missingness policy and data flow (all measured)

- **Rule:** DK/refused/blank ≥5% of the file → keep as an explicit category ("Not reported"/"Unknown"): _INCOMG1 (19.1%), _RFBING6 (10.4%), _BMI5CAT (9.4%), _SMOKER3 (7.0%). Below 5% → recode to missing and complete-case exclude. Simple, transparent, defensible — the brief explicitly says a sophisticated method is not needed, only intentional handling.
- **Data flow (measured):** 457,670 → drop 1,310 with unusable outcome (GENHLTH 7/9/blank) → 456,360 → drop 32,220 with any missing value on the fifteen low-missingness predictors → **final analytic sample 424,140 (92.7% retained)**. Primary event rate in the final sample: 18.9%. Secondary outcome loses a further 8,364 (2.0%) to PHYSHLTH 77/99 in its own model only.

### 1.5 Why these models

- **Baseline logistic regression** — mandated, and the right benchmark: odds ratios for 19 predictors give the interpretable population-health story (direction, significance, practical importance) the brief's §6.4 requires.
- **ML model 1: decision tree (CART)** — the brief *recommends* it for interpretability; a depth-limited tree gives a human-readable segmentation of high-burden respondents (e.g., "≥2 chronic conditions AND cost barrier"), which is exactly the surveillance framing we want.
- **ML model 2: random forest** — the brief's recommended "stronger nonlinear model"; tests whether interactions among conditions, behaviours and SES add predictive value over the additive baseline. Basic tuning only (n_trees, max depth, min leaf) per §6.5.
- **Optional third: gradient boosting** if time permits — still within the recommended list. **No SVM** (adds little interpretability at this n), **no neural nets** (brief: not recommended).
- With n = 424k and 19 predictors, overfitting risk is low and training cost trivial — no need for anything exotic, and using an unexplainable model is a listed common mistake.

### 1.6 Trade-offs vs Topics B and C — honest assessment (why this is NOT our recommended pick)

- **For A:** safest feasibility profile of the three (outcome missingness 0.29%; every predictor verified in the file); the strongest and most direct literature anchor; the cleanest motivation; the healthiest event rate (19.7%).
- **Against A:** it is also the most *predictable* choice — self-rated health is the textbook BRFSS exercise, and in a relatively-graded cohort several groups may pick it, making differentiation harder. Its "insight ceiling" is lower: everyone already believes chronic disease and poverty predict poor health; our contribution is confirmatory rather than surprising.
- **vs Topic B (mental health):** B's frequent mental distress (MENTHLTH ≥ 14, measured 13.9%) plus the six-item disability battery in `topic_b.csv` (DEAF…DIFFALON, 3.3–4.5% blank) offers a fresher equity angle (Cree 2020 MMWR) and a true ML precedent (Parekh & Fahim 2021). But note `topic_b.csv` *lacks* GENHLTH/ADDEPEV3, and marijuana outcomes are unusable (state-optional module, absent from all three files) — B must be the FMD variant, not the brief's literal daily-marijuana example.
- **vs Topic C (preventive care gap):** C's no-routine-check-up outcome (18.2%) or no-dental-visit (~31% via _DENVST3) is the most actionable for health-system planning, but FLUSHOT7 is absent from `topic_c.csv` (flu-shot variant infeasible) and insurance requires the PRIMINS2 naming-trap handling (~4% DK/refused; 25,406 code-88 uninsured).
- **Risk register for A:** (i) class imbalance ~19% — mitigate with class weights and AUPRC reporting; (ii) _AGE80 top-coding at 80 flattens the oldest gradient — note as limitation; (iii) income "Not reported" level may absorb SES signal — check coefficient sanity; (iv) reverse causation everywhere (conditions ↔ perceived health) — we make no causal claims; (v) commonness of topic — mitigate with the surveillance framing and the chronic-count descriptive narrative.

### 1.7 Fit to course themes

Population-health surveillance framing: identifying which respondent profiles concentrate high health burden is a population-health management task (quadruple aim: improving population health while informing where care resources and prevention outreach yield most value). The unweighted analytic-sample caveat is itself a teachable point about the difference between surveillance estimates and predictive modelling.

---

## PART 2 — What we will learn & what is required

### 2.1 Mapping to the seven learning outcomes

| # | Learning outcome (brief §1) | How Topic A delivers it |
|---|---|---|
| 1 | Formulate a feasible healthcare analytics problem | Outcome verified in-file (0.29% missing, 19.7% event rate); POORHLTH rejected on measured 41.4% skip-logic blankness — feasibility argued with numbers, not hope. |
| 2 | Data cleaning & feature engineering; handle missing/refused/inapplicable responsibly | 19 predictors × documented recodes; the ≥5% explicit-category / <5% complete-case rule; 88→0 and 77/99→missing on PHYSHLTH; skip-logic recognition. |
| 3 | Descriptive analysis of the analytic sample | Table 1 by outcome status; chronic-count gradient chart (7.5%→66.2%); income-band gradient chart (45.0%→5.3%). |
| 4 | Baseline regression + build and compare ML models | Logistic regression ORs vs decision tree vs random forest on identical train/test splits. |
| 5 | Interpret findings in public health terms | Which conditions/behaviours/SES markers dominate importance; consistency with descriptives; no causal language. |
| 6 | Communicate professionally | Two-page proposal now; data-flow diagram, comparison table and limitation statement in the final report. |
| 7 | Work effectively in a team | Five defined roles with a shared-review rule (below). |

### 2.2 Required artefact checklist (brief §6) — all pre-scoped

- **Variable dictionary** (§6.1): 21 rows (2 outcomes + 19 predictors) with name / concept / original coding / recode / role — drafted in Part 3 annex table; expands 1:1 into the required dictionary.
- **Missingness table** (§6.2): per-variable counts and % already measured (income 19.1%, binge 10.4%, BMI 9.4%, smoker 7.0%, all others <2.1%) with a per-variable Keep-as-category / Exclude decision column.
- **Data-flow diagram** (§6.3): 457,670 → −1,310 outcome-unusable → 456,360 → −32,220 predictor-missing → **424,140**; separate branch −8,364 for the secondary outcome.
- **Table 1 + ≥2 charts** (§6.3): descriptives by outcome status; chart 1 = poor/fair % by chronic-condition count; chart 2 = poor/fair % by income band (both already computed).
- **Baseline regression interpretation** (§6.4): direction / significance / practical importance / limitations on 19 predictors.
- **Model comparison table** (§6.6): Accuracy, Sensitivity, Specificity, AUROC, AUPRC for LR / DT / RF on a held-out test set, plus a "is the difference meaningful?" paragraph.
- **Interpretation answering the five §6.7 questions**: most important variables; public-health sense; consistency with descriptives; limitations (top-coded age, self-report, cross-sectional reverse causation, unweighted sample); what extra data would help (clinical measures, utilisation, social needs).
- **Required limitation statement**: quoted verbatim in Part 3 §7.

### 2.3 Common mistakes (brief) — how this proposal avoids each

Outcome checked in the real file ✓ · 19 ≤ 20 predictors with justified cuts ✓ · codebook codes (7/9/77/99/88/blank) handled per variable ✓ · only recommended, explainable models ✓ · no causal language ✓ · interpretation plan tied to descriptives ✓ · limitations pre-listed ✓.

---

## PART 3 — THE SUBMITTABLE PROPOSAL

# Predicting High Health Burden Among 2024 BRFSS Respondents

*ISSS623 Applied Data Science in Healthcare — Group Project Proposal*

## 1. Project question

Among respondents in the 2024 BRFSS analytic sample, can selected demographic, behavioural, healthcare access, and chronic condition variables predict **high health burden**, defined primarily as **poor or fair self-rated general health** (GENHLTH), with **frequent physical distress** (PHYSHLTH ≥ 14 days) as a secondary outcome?

## 2. Public health motivation

Self-rated general health is the Centers for Disease Control and Prevention's core health-related quality of life (HRQOL) indicator and a validated predictor of mortality, morbidity, and healthcare utilisation. In our preliminary audit of the 2024 file, 19.7% of respondents with a valid response reported fair or poor health, and the rate rose steeply from 7.5% among respondents with none of eight measured chronic conditions to 66.2% among those with four or more, and from 5.3% in the highest income band to 45.0% in the lowest. A model that identifies which respondent profiles concentrate high health burden supports population-health surveillance and prevention outreach — helping health systems understand where poor perceived health clusters across behavioural, access, chronic disease, and socioeconomic domains, in line with population-health management and quadruple-aim thinking.

## 3. Brief literature context

Jiang and Hesser (2006) used Rhode Island's 2002 BRFSS to show that fair/poor general health and the CDC Healthy Days measures are strongly associated with demographics, health behaviours, and health risks — the direct template for our outcome and predictor domains. Moriarty, Zack and Kobau (2003) establish GENHLTH and PHYSHLTH as the CDC's standard population HRQOL tracking measures, supporting our primary/secondary pairing. Clark et al. (2021) demonstrate that machine-learning models can predict self-rated health from BRFSS data and yield health-equity insights, providing methodological precedent for comparing a logistic baseline against tree-based learners on this outcome.

## 4. Outcome variable and proposed predictors

We verified every variable in the supplied 2024 extract (457,670 respondents). **Primary outcome:** GENHLTH recoded to poor/fair (4–5) vs excellent/very good/good (1–3); don't know/refused (7/9, 0.29%) excluded. **Secondary outcome:** PHYSHLTH recoded 88→0 days and 77/99 (2.4%)→missing, then dichotomised at ≥14 days (frequent physical distress, 14.3%). We deliberately rejected POORHLTH (activity limitation) as an outcome because 41.4% of records are blank through skip logic. We propose **19 predictors** across four domains (annex table): demographics/SES — sex, age, race/ethnicity group, education, income; behaviours — smoking status, binge drinking, leisure-time physical activity, BMI category; chronic conditions — diabetes, coronary heart disease/MI, stroke, COPD, depressive disorder, current asthma, arthritis, kidney disease; healthcare access — personal care provider, cost barrier to care. Missing/refused handling is rule-based: variables with ≥5% unknown (income 19.1%, binge drinking 10.4%, BMI 9.4%, smoking 7.0%) retain an explicit "Not reported" category; variables below 5% are recoded to missing with complete-case exclusion. The resulting analytic flow is 457,670 → 456,360 (outcome usable) → **424,140 (92.7% retained)**, with an 18.9% primary event rate. We excluded EMPLOY1 (its "unable to work" code risks outcome leakage), MARITAL and health-plan status (redundancy/parsimony), CHECKUP1 (a care-seeking outcome in its own right), and retained urban–rural status for descriptive comparison only.

## 5. Planned models

We will split the analytic sample into stratified training and test sets (70/30, fixed seed). The **baseline model** is a logistic regression on all 19 predictors, interpreted through odds ratios (direction, statistical significance, practical importance, limitations). We will then build **two machine-learning models**: a **decision tree** (depth and minimum-leaf tuned) for an interpretable segmentation of high-burden profiles, and a **random forest** (number of trees, depth, minimum-leaf tuned) as a stronger nonlinear comparator; a gradient-boosting model may be added if time permits. Class imbalance (~19% events) will be addressed with class weighting. All models will be compared on the held-out test set in a single table reporting **Accuracy, Sensitivity, Specificity, AUROC, and AUPRC**, followed by interpretation: variable importance (odds ratios, tree splits, permutation importance), whether findings make public-health sense, consistency with the descriptive analysis (Table 1, chronic-condition and income gradients), model limitations, and what additional data would improve prediction. No causal claims will be made.

## 6. Team role allocation

| Role | Member | Responsibilities |
|---|---|---|
| Data Preparation Lead | [You] | Codebook review; variable dictionary; recoding of all 21 variables; missingness table and rule-based handling; data-flow diagram; analytic-sample construction. |
| Project & Proposal Lead | [Member 2] | Timeline and coordination; proposal drafting; GitHub repo and notebook hygiene; scope control (≤20 predictors, 2024 only, no weights). |
| Descriptive Analysis Lead | [Member 3] | Table 1 by outcome status; chronic-count and income-gradient charts; outcome prevalence checks against the data dictionary. |
| Modelling Lead | [Member 4] | Train/test split; logistic baseline; decision tree and random forest with basic tuning; model comparison table (Accuracy/Sensitivity/Specificity/AUROC/AUPRC). |
| Interpretation & Report Lead | [Member 5] | Variable-importance analysis; public-health interpretation; limitations; final report and presentation assembly. |

All members review the full pipeline end-to-end; each deliverable is checked by at least one member outside the owning role.

## 7. Scope and limitations

This project uses the 2024 BRFSS only, without survey weights, per the project brief. *"This project is an unweighted respondent-level predictive modelling exercise using the 2024 BRFSS analytic sample. The results should not be interpreted as nationally representative prevalence estimates."* Further limitations: cross-sectional self-reported data (no causal inference), age top-coded at 80, and income non-response retained as an explicit category.

## References

1. Jiang Y, Hesser JE. Associations between health-related quality of life and demographics and health risks: results from Rhode Island's 2002 Behavioral Risk Factor Survey. *Health Qual Life Outcomes*. 2006;4:14.
2. Moriarty DG, Zack MM, Kobau R. The CDC's Healthy Days Measures — population tracking of perceived physical and mental health over time. *Health Qual Life Outcomes*. 2003;1:37.
3. Clark CR, Ommerborn MJ, Moran K, et al. Predicting self-rated health across the life course: health equity insights from machine learning models. *J Gen Intern Med*. 2021;36(5):1181–1188.

## Annex A — Variable table (2 outcomes + 19 predictors)

| Concept | BRFSS variable | Role | Original coding | Recoded version |
|---|---|---|---|---|
| Self-rated general health | GENHLTH | Primary outcome | 1 excellent … 5 poor; 7 DK; 9 refused | Poor/fair (4–5)=1 vs good+ (1–3)=0; 7/9/blank excluded (0.29%) |
| Physically unhealthy days | PHYSHLTH | Secondary outcome | 1–30 days; 88 none; 77 DK; 99 refused | 88→0; ≥14 days=1 vs <14=0; 77/99→excluded (2.4%) |
| Sex | SEXVAR | Predictor | 1 male; 2 female | Binary female vs male |
| Age | _AGE80 | Predictor | 18–80 (top-coded at 80) | Continuous years |
| Race/ethnicity group | _RACEGR3 | Predictor | 1 White NH; 2 Black NH; 3 Other NH; 4 Multiracial NH; 5 Hispanic; 9 DK/ref | 5 categories; 9→missing (2.0%) |
| Education level | _EDUCAG | Predictor | 1 <HS; 2 HS grad; 3 some college; 4 college grad; 9 | 4 categories; 9→missing (0.5%) |
| Household income | _INCOMG1 | Predictor | 1 <$15k … 7 ≥$200k; 9 DK/ref | 7 bands + explicit "Not reported" (19.1%) |
| Smoking status | _SMOKER3 | Predictor | 1 daily; 2 some days; 3 former; 4 never; 9 | 4 categories + "Not reported" (7.0%) |
| Binge drinking | _RFBING6 | Predictor | 1 no; 2 yes; 9 | Yes vs no + "Not reported" (10.4%) |
| Leisure-time physical activity | _TOTINDA | Predictor | 1 had activity; 2 none; 9 | Active vs inactive; 9→missing (0.3%) |
| BMI category | _BMI5CAT | Predictor | 1 under; 2 normal; 3 over; 4 obese; blank | 4 categories + "Unknown" (9.4% blank) |
| Diabetes | DIABETE4 | Predictor | 1 yes; 2 gestational; 3 no; 4 pre-diabetes; 7/9 | Yes (1) vs no (2–4); 7/9→missing (0.2%) |
| CHD or MI | _MICHD | Predictor | 1 yes; 2 no; blank | Yes vs no; blank→missing (1.1%) |
| Stroke | CVDSTRK3 | Predictor | 1 yes; 2 no; 7/9 | Yes vs no; 7/9→missing (0.3%) |
| COPD | CHCCOPD3 | Predictor | 1 yes; 2 no; 7/9 | Yes vs no; 7/9→missing (0.5%) |
| Depressive disorder | ADDEPEV3 | Predictor | 1 yes; 2 no; 7/9 | Yes vs no; 7/9→missing (0.6%) |
| Current asthma | _CASTHM1 | Predictor | 1 no; 2 yes; 9 | Yes vs no; 9→missing (0.9%) |
| Arthritis | _DRDXAR2 | Predictor | 1 yes; 2 no; blank | Yes vs no; blank→missing (0.6%) |
| Kidney disease | CHCKDNY2 | Predictor | 1 yes; 2 no; 7/9 | Yes vs no; 7/9→missing (0.4%) |
| Personal care provider | PERSDOC3 | Predictor | 1 one; 2 more than one; 3 none; 7/9 | Has provider (1–2) vs none (3); 7/9→missing (1.0%) |
| Cost barrier to care | MEDCOST1 | Predictor | 1 yes; 2 no; 7/9 | Yes vs no; 7/9→missing (0.4%) |
| Urban/rural status | _URBSTAT | Subgroup (descriptive only) | 1 urban; 2 rural; blank | Urban vs rural; blank→missing (3.2%); not used in models |
