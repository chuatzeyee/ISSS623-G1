# Topic C — Predicting Preventive Care Gaps: Who Misses the Annual Check-Up?

**ISSS623 Applied Data Science in Healthcare · Group Project Proposal Pack · Recommended pitch (Topic C)**

---

## PART 1 — Decision rationale (read this before the group meeting)

### 1.1 Why this outcome, in numbers

Our primary outcome is **no routine check-up within the past year**, built from `CHECKUP1` in `brfss2024_topic_c.csv` (N = 457,670). Every number below was computed directly from the professor-supplied CSV, not quoted from a paper:

- **Event rate 18.2%** — 82,313 of 452,336 respondents with a valid response report their last routine check-up was more than a year ago (codes 2/3/4) or never (code 8). This is close to the "ideal" zone for classification: rare enough to be a meaningful public-health gap, common enough that models can learn it without heavy imbalance surgery (contrast: outcomes below ~5% force SMOTE/threshold gymnastics we would then have to defend).
- **Cleanest outcome in any of the three files: only 1.17% of rows are unusable** (4,615 "don't know" (7), 717 "refused" (9), 2 blank). Compare the traps in the alternatives: `POORHLTH` is 41.4% blank due to skip logic (a known common mistake per the project brief — an outcome not checked in the codebook), and `_BMI5CAT` is 9.4% missing.
- **It is a behaviour, not a self-perception.** Poor/fair self-rated health (Topic A, 19.7%) and frequent mental distress (Topic B, 13.9%) are subjective states. A missed check-up is a concrete service-utilisation event that a health system can *act on* — you can call, remind, and enrol people; you cannot "outreach" someone into feeling healthier.
- **The bivariate signal is already strong and coherent** (verified on the projected analytic sample of 381,759): gap rate is **57.8% among the uninsured vs 7.6% among Medicare respondents**; **62.5% among those without a personal healthcare provider vs 12.1% with one**; **38.0% among those reporting a cost barrier vs 16.0% without**. A predictive model has real structure to find, and the descriptives will visibly agree with the models — one of the interpretation questions the brief requires us to answer.

**Secondary outcome:** cost-related barrier to seeing a doctor, `MEDCOST1` (9.5% "yes", only 0.37% invalid). It is the natural companion question — the check-up gap is "did care happen?", the cost barrier is "why not?". One primary + one secondary satisfies the brief's rule against model proliferation.

### 1.2 Why Topic C beats Topics A and B (the pitch)

| Consideration | Topic A (High health burden) | Topic B (Mental distress / disability) | **Topic C (Preventive care gap)** |
|---|---|---|---|
| Outcome cleanliness | GENHLTH fine (19.7%), but composite HRQOL outcomes drag in POORHLTH (41.4% blank) | FMD 13.9%, disability items 3.3–4.5% blank | **CHECKUP1: <1.2% unusable** |
| Predictor–outcome separation | Health status predicting health status — partly circular | Mental-health predictors of mental distress — same issue | **Access/SES/health predictors → utilisation behaviour: clean causal-free predictive framing** |
| Actionability | "These people feel unwell" → diffuse response | Stigma-sensitive; interventions clinical | **Direct outreach targeting: remind, enrol, subsidise — a left-shift lever** |
| File flexibility | 29 columns | 33 columns, *no* GENHLTH/PHYSHLTH | **topic_c.csv is a 37-column superset — we can pivot to a Topic A question without re-extraction if the prof pushes back** |
| Distinctiveness | Likely the most-picked "default" topic in class | Crowded mental-health space | **Less contested; access-equity angle is fresh** |
| Lecture 1 hook | — | — | **"The cheapest hospital bed is the one never needed" — the left-shift argument** |
| Singapore resonance | General | General | **Healthier SG runs on exactly this logic: enrol residents with one regular family doctor and fund preventive visits. Our PERSDOC3 finding (62.5% vs 12.1% gap) is a direct empirical echo** |

### 1.3 Codebook diligence we can showcase (the 3% feasibility criterion)

The proposal rubric's single biggest component (3%) is data feasibility and variable selection, with explicit credit for "awareness of coding, missingness, and recoding needs". We have concrete evidence:

1. **The 2024 insurance rename trap.** Health insurance in 2024 is `PRIMINS2`. Older BRFSS papers and code snippets use `PRIMINSR` (2021–2023) or `HLTHPLN1` (pre-2021). Groups that copy legacy code will silently fail; we caught it in the codebook. `PRIMINS2` also gives insurance **type** (employer / private / Medicare / Medicaid / military / IHS / uninsured), which is far richer than the yes/no legacy variable.
2. **A verified redundancy.** The calculated variable `_HLTHPL2` (any coverage) is logically implied by `PRIMINS2`: 25,405 of the 25,406 respondents with `_HLTHPL2 = 2` (no coverage) are exactly the `PRIMINS2 = 88` group. We therefore **drop `_HLTHPL2`** — keeping both would be double-counting and would look like variable-list padding.
3. **The 88 = "none" convention.** In day-count variables (`MENTHLTH`, `PHYSHLTH`) 88 means "zero days" and must be recoded to 0, not treated as 88 days or as missing. In `PRIMINS2`, 88 means "no coverage" — same code, different meaning. We handle each per codebook.
4. **What is *not* in the extract.** `FLUSHOT7` (flu vaccination) is not in any of the three files, so an immunisation-gap outcome is infeasible — we say so rather than promise it. Marijuana variables are state-optional modules and absent. `_DENVST3` (no dental visit in past year, 31.1%) *is* present but its missing/refused respondents are folded into code 9, so we keep it as a **descriptive comparison** of a second preventive behaviour, not as an outcome.
5. **`_INCOMG1` refusals are too big to drop.** 19.1% of respondents have income "not reported" (code 9). Dropping them would delete ~73k rows *and* bias the sample (income refusal correlates with SES). We keep "not reported" as an explicit category — a defensible, documented decision the rubric rewards.

### 1.4 Why each predictor domain (19 predictors, deliberately under the 20 cap)

- **Healthcare access (3): `PRIMINS2`, `PERSDOC3`, `MEDCOST1`.** The core of the story — insurance type, usual source of care, affordability. These are the levers policy can pull. Verified gap-rate spreads of 50, 50 and 22 percentage points respectively.
- **Health status (2): `GENHLTH`, `MENTHLTH`.** Sicker people have more scheduled contact (reverse gradient expected — interesting to interpret); mental distress is a known barrier to help-seeking. We **exclude `PHYSHLTH`** as near-collinear with `GENHLTH` — restraint the rubric rewards.
- **Chronic conditions (2): `DIABETE4`, `_MICHD`.** Diagnosed chronic disease anchors people into recall systems; their *absence* plus other risk factors flags the "healthy-feeling avoider" the outreach story targets. We excluded `CHCCOPD3` and `CHCKDNY2` (lower prevalence, same mechanism) to stay lean.
- **Disability (3): `DIFFWALK`, `DECIDE`, `DIFFALON`.** Mobility, cognition, and errand-independence are three distinct mechanisms for missing appointments (getting there, remembering, needing help). Cree et al. (2020) show disability stratifies distress and access; 3.8–4.5% blanks handled explicitly.
- **Behaviours (2): `_SMOKER3`, `_TOTINDA`.** Health-behaviour clustering: smokers and the inactive underuse preventive care. `_SMOKER3` code 9 (7.0%) kept as "not reported".
- **Demographics/SES (7): `SEXVAR`, `_AGEG5YR`, `_RACEGR3`, `_EDUCAG`, `_INCOMG1`, `EMPLOY1`, `_URBSTAT`.** The equity axis (Clark et al. 2021 framing). `EMPLOY1` matters independently of insurance (retirees on Medicare show only 7.6% gap); `_URBSTAT` tests rural access disadvantage.
- **Dropped besides the above:** `MARITAL` (weakest theoretical link, keeps table lean), `_HLTHPL2` (redundant, see 1.3).

### 1.5 Why these models

- **Baseline logistic regression** — required, and genuinely the right baseline: odds ratios per predictor give the interpretable access-equity story, and it sets the AUROC bar the ML models must beat to justify their complexity.
- **Decision tree (CART, depth-limited)** — recommended by the brief for interpretability; the tree's top splits (we expect PERSDOC3 / insurance / age) *are* the outreach-targeting rules a programme manager could use.
- **Random forest** — the stronger nonlinear learner; captures interactions (e.g., uninsured × young × healthy-feeling) and yields permutation importances to cross-check the regression. Gradient boosting held as optional third if time permits.
- **No neural networks, no SVM on 380k rows** — the brief flags "unexplainable advanced models" as a common mistake, and SVMs scale poorly at this N for little interpretability payoff.

### 1.6 Risks and mitigations

| Risk | Mitigation |
|---|---|
| MEDCOST1 doing double duty (predictor and secondary outcome) looks confused | Cleanly separated: it is a *predictor* in the primary model; the secondary-outcome run is a separate model where it becomes the target and is removed from the predictor set. One paragraph in the proposal states this explicitly. |
| 18% event rate → accuracy is misleading | Report Sensitivity/Specificity/AUROC/AUPRC; discuss threshold choice; a majority-class dummy scores 82% accuracy and 0 sensitivity — we say so up front. |
| Reverse gradient (sicker people have *fewer* gaps) misread as "check-ups cause illness" | Predictive framing only; the brief bans causal claims and we pre-commit to utilisation-behaviour language. |
| Complete-case deletion criticised | Deletion confined to variables with <5% invalid; large "not reported" groups (`_INCOMG1` 19.1%, `_SMOKER3` 7.0%, `PRIMINS2` 77/99 ≈ 4.1%) kept as categories; every decision in the missingness table. Projected final N = **381,759 (83.4% of 457,670)** — computed, not guessed. |
| Prof prefers a Topic A question | topic_c.csv is a superset containing GENHLTH/PHYSHLTH/MENTHLTH — we can pivot outcomes without re-extraction. |

---

## PART 2 — What we will learn & what is required

### 2.1 Mapping to the seven learning outcomes (from the project brief)

1. **Formulate a feasible healthcare analytics problem** — outcome chosen *after* verifying coding and missingness in the actual extract (18.2% event rate, <1.2% invalid), in the brief's prescribed question form.
2. **Data cleaning & feature engineering; handle missing/refused/inapplicable responsibly** — the 7/9/77/99/88/blank taxonomy is handled per variable with documented decisions; recodes include insurance-type grouping and day-count 88→0; this is Member 1's core deliverable and the final report's biggest single criterion (6%).
3. **Descriptive analysis of the analytic sample** — Table 1 stratified by outcome plus two charts (gap rate by insurance type; gap rate by age band and personal-provider status).
4. **Baseline regression + build and compare ML models** — logistic regression, then CART and random forest, compared on one held-out test set.
5. **Interpret findings in public health terms** — outreach-targeting story, Healthier SG parallel, reverse health-status gradient discussion.
6. **Communicate professionally** — 2-page proposal now; slides and final report follow the same variable dictionary so numbers never drift between artefacts.
7. **Work effectively in a team** — five named roles with concrete responsibilities and whole-pipeline review (Section 6 of the proposal).

### 2.2 Required analysis artefacts and how this proposal sets them up

| Required artefact | Our set-up |
|---|---|
| **Variable dictionary** (name / concept / original coding / recode / role) | Section 4 table is already in that exact format — the final dictionary is an extension, not a rewrite. |
| **Missingness table with per-variable decisions** | Decisions pre-made and quantified: drop-if-invalid for <5% variables; category-retention for `_INCOMG1` (19.1%), `_SMOKER3` (7.0%), `PRIMINS2` 77/99 (4.1%); skip-logic awareness (POORHLTH counter-example). |
| **Data-flow diagram (457,670 → final N)** | 457,670 → 452,336 (valid `CHECKUP1`) → **381,759** (complete cases on low-missingness predictors; verified in pandas). |
| **Table 1 descriptives + 2 charts** | Stratified by outcome; charts chosen to prefigure model findings (insurance type; provider × age). |
| **Model comparison table** (Accuracy / Sensitivity / Specificity / AUROC / AUPRC) | Three models × five metrics on one stratified held-out test set; dummy-classifier row included for honesty. |
| **Interpretation questions** (important variables? public-health sense? consistent with descriptives? limitations? extra data?) | Pre-answerable: access variables should dominate; descriptive gap rates (57.8% uninsured etc.) give the consistency check; extra data wish-list = state-level access measures, `FLUSHOT7`, cost/utilisation linkage. |
| **Required limitation statement** | Quoted verbatim in Section 5 of the proposal. |

---

## PART 3 — THE SUBMITTABLE PROPOSAL

# Predicting Preventive Care Gaps: Who Misses the Annual Check-Up?

**ISSS623 Applied Data Science in Healthcare — Group Project Proposal (2024 BRFSS)**

## 1. Project question

Among respondents in the 2024 BRFSS analytic sample, can selected demographic, behavioural, healthcare access, chronic condition, and disability variables predict **not having had a routine check-up within the past year**? As a secondary question using the same analytic sample and predictor framework, we will examine whether the same variables predict **a cost-related barrier to seeing a doctor**.

## 2. Public health motivation

Routine check-ups are the entry point to preventive care: blood-pressure and diabetes screening, vaccination, and early detection all depend on periodic contact with primary care. In our verified extract of the 2024 BRFSS (N = 457,670), **18.2% of respondents with a valid response had no routine check-up in the past year**, and the gap is starkly unequal: 57.8% among uninsured respondents versus 7.6% among Medicare respondents, and 62.5% among those without a personal healthcare provider versus 12.1% among those with one. A model that identifies who misses check-ups supports the most actionable of public-health responses — targeted outreach and enrolment — and reflects the "left-shift" principle that resources spent keeping people connected to primary care are cheaper than the downstream admissions they prevent. The question also resonates beyond the US: Singapore's Healthier SG programme is built on precisely the mechanism our strongest predictor captures — enrolling every resident with one regular family doctor to anchor preventive care.

## 3. Brief literature context

Clark et al. (2021) used machine learning on BRFSS data to predict poor self-rated health and explicitly framed prediction as a tool for identifying underserved groups, motivating our access-equity angle and our choice of interpretable models. Parekh and Fahim (2021) demonstrated the standard BRFSS modelling recipe we adopt — logistic regression compared against decision tree and random forest classifiers on a binary behavioural outcome. Cree et al. (2020, MMWR) documented how disability status stratifies both distress and healthcare access in BRFSS, motivating our inclusion of the disability battery; the CDC Healthy Days literature (Moriarty, Zack & Kobau 2003) underpins our handling of the day-count measures. We note that 2024 BRFSS renamed the insurance variable to `PRIMINS2` (formerly `PRIMINSR`/`HLTHPLN1` in the studies above), which we verified in the 2024 codebook.

## 4. Outcome variable and proposed predictors

All variables below were checked in the 2024 BRFSS codebook and verified in the supplied extract (`brfss2024_topic_c.csv`, 37 columns). Throughout, 7/77 = don't know, 9/99 = refused, 88 = "none" in day-count items, blank = not asked. The primary outcome has only 1.17% unusable responses. We propose **19 predictors**, deliberately under the 20-variable cap; we excluded `PHYSHLTH` (near-collinear with `GENHLTH`), `_HLTHPL2` (verified redundant with `PRIMINS2` = 88), and `MARITAL`, `CHCCOPD3`, `CHCKDNY2` (weaker marginal contribution) to keep the model lean and interpretable.

| Concept | Variable | Role | Original coding | Recode |
|---|---|---|---|---|
| No routine check-up ≤1 yr | CHECKUP1 | **Primary outcome** | 1 ≤1yr; 2 ≤2yr; 3 ≤5yr; 4 >5yr; 8 never; 7/9 DK/ref | 1→0; 2/3/4/8→1; 7/9/blank→missing (1.17%) |
| Cost barrier to care | MEDCOST1 | **Secondary outcome** (also predictor in primary model) | 1 yes; 2 no; 7/9 | 1→1; 2→0; 7/9→missing (0.37%) |
| Insurance type | PRIMINS2 | Predictor | 1–10 types; 88 none; 77/99 | Groups: employer(1)/private(2)/Medicare(3,4)/Medicaid-state(5,6,9)/military-IHS(7,8)/other-gov(10)/uninsured(88)/not-reported(77,99; 4.1%) |
| Personal healthcare provider | PERSDOC3 | Predictor | 1 one; 2 more than one; 3 none; 7/9 | 1/2→1; 3→0; 7/9→missing |
| Self-rated general health | GENHLTH | Predictor | 1–5 excellent→poor; 7/9 | Ordinal 1–5; 7/9→missing |
| Mentally unhealthy days | MENTHLTH | Predictor | 1–30 days; 88 none; 77/99 | 88→0; keep 0–30; 77/99→missing |
| Diabetes | DIABETE4 | Predictor | 1 yes; 2 pregnancy-only; 3 no; 4 pre-diabetes; 7/9 | 1→1; 2/3/4→0; 7/9→missing |
| CHD or MI history | _MICHD | Predictor | 1 yes; 2 no; blank | 1→1; 2→0; blank→missing (1.1%) |
| Difficulty walking | DIFFWALK | Predictor | 1 yes; 2 no; 7/9; blank | 1→1; 2→0; else missing (≈4%) |
| Cognitive difficulty | DECIDE | Predictor | 1 yes; 2 no; 7/9; blank | 1→1; 2→0; else missing (≈4%) |
| Difficulty doing errands alone | DIFFALON | Predictor | 1 yes; 2 no; 7/9; blank | 1→1; 2→0; else missing (≈4.5%) |
| Smoking status | _SMOKER3 | Predictor | 1/2 current; 3 former; 4 never; 9 | Current/former/never/not-reported(9; 7.0%) |
| Any leisure physical activity | _TOTINDA | Predictor | 1 yes; 2 no; 9 | 1→1; 2→0; 9→missing (0.3%) |
| Sex | SEXVAR | Predictor | 1 M; 2 F | As is |
| Age group | _AGEG5YR | Predictor | 1–13 five-yr bands; 14 unknown | Bands 1–13; 14→missing (1.8%) |
| Race/ethnicity group | _RACEGR3 | Predictor | 1–5 groups; 9 | Groups 1–5; 9→missing (2.0%) |
| Education level | _EDUCAG | Predictor | 1–4; 9 | Levels 1–4; 9→missing (0.5%) |
| Income group | _INCOMG1 | Predictor | 1–7; 9 not reported | Levels 1–7 + not-reported(9) kept as category (19.1% — too large to drop) |
| Employment status | EMPLOY1 | Predictor | 1–8; 9 refused | Employed(1,2)/unemployed(3,4)/homemaker-student(5,6)/retired(7)/unable(8); 9/blank→missing |
| Urban/rural residence | _URBSTAT | Predictor | 1 urban; 2 rural; blank | 1→urban; 2→rural; blank→missing (3.2%) |

**Data flow:** 457,670 respondents → 452,336 with a valid primary outcome → projected **381,759 complete cases (83.4%)** after per-variable decisions above (verified in pandas). `MEDCOST1` serves as a predictor in the primary model; in the secondary analysis it becomes the target and is removed from the predictor set. `_DENVST3` (no dental visit, 31.1%) is retained for descriptive comparison only, since its refused/missing responses are folded into a single code; `FLUSHOT7` is not in our extract, so immunisation outcomes are out of scope.

## 5. Planned models and analysis

We will (i) clean and recode per the table above, documenting every missing/refused/inapplicable decision in a missingness table and data-flow diagram; (ii) produce Table 1 descriptives stratified by outcome plus two charts (gap rate by insurance type; gap rate by age band and personal-provider status); (iii) fit a **baseline logistic regression** for interpretable odds ratios; (iv) fit **two machine-learning models — a depth-limited decision tree (CART)** for transparent decision rules and a **random forest** for nonlinear interactions and permutation importance (gradient boosting optional); (v) evaluate all models on a single stratified held-out test set (70/30) using **Accuracy, Sensitivity, Specificity, AUROC and AUPRC**, with a majority-class dummy baseline reported because the 18% event rate makes accuracy alone misleading; and (vi) interpret: which variables matter most, whether they make public-health sense, whether they are consistent with the descriptives, limitations, and what additional data would help. No neural networks will be used; interpretability is prioritised throughout. Survey weights (_LLCPWT) will not be used. *"This project is an unweighted respondent-level predictive modelling exercise using the 2024 BRFSS analytic sample. The results should not be interpreted as nationally representative prevalence estimates."*

## 6. Team role allocation

| Role | Member | Responsibilities |
|---|---|---|
| Data Preparation Lead | [You] | Codebook verification (incl. 2024 renames such as PRIMINS2), recoding per variable dictionary, missingness table and per-variable decisions, data-flow diagram, analytic-sample construction |
| Project & Proposal Lead | [Member 2] | Timeline and coordination, proposal drafting, GitHub repo and Colab management, submission logistics |
| Descriptive Analysis Lead | [Member 3] | Table 1 stratified descriptives, the two charts, bivariate gap-rate checks that anchor model interpretation |
| Modelling Lead | [Member 4] | Logistic regression baseline, decision tree and random forest, train/test protocol, model comparison table across the five metrics |
| Interpretation & Report Lead | [Member 5] | Public-health interpretation, limitations, final report and presentation assembly |

All members review the full pipeline end-to-end; no artefact is owned by one person alone.

**References.** Clark CR, et al. *J Gen Intern Med* 2021;36(5):1181–1188. · Parekh T, Fahim F. *Drug Alcohol Depend* 2021;225:108789. · Cree RA, et al. *MMWR* 2020;69(36). · Moriarty DG, Zack MM, Kobau R. *Health Qual Life Outcomes* 2003;1:37. · CDC, 2024 BRFSS Codebook (LLCP).
