# Topic C - Caregiving and the Annual Check-Up: Do Caregivers Miss Their Own Preventive Care?

**ISSS623 Applied Data Science in Healthcare · Group Project Proposal Pack · GROUP-CHOSEN ANGLE (caregiver vs non-caregiver)**

---

## PART 1 - Decision rationale (read this before finalising)

### 1.0 What changed from the generic Topic C draft

The group decided to centre the question on **caregiver status** - whether providing regular care to a family member or friend is associated with missing versus attending one's own annual check-up. This is a sharper, more distinctive question than the generic care-gap model, and it is feasible - but it comes with one material trade-off that we verified directly in the raw data and must state honestly:

- `CAREGIV1` ("During the past 30 days, did you provide regular care or assistance to a friend or family member?") lives in the **Caregiver optional module**, which only **16 of 53 states/territories** fielded in 2024. It is NOT in the prof's `brfss2024_topic_c.csv` extract - we extracted it ourselves from the raw `LLCP2024.ASC` using the prof's `BRFSS_Extraction.ipynb` logic (exactly the "you may go beyond these suggestions" invitation in the Project Guidance slide).
- Analytic sample shrinks from ~382k to **87,098 complete cases** - still far more than enough for our three models, but the 16-state footprint adds a second generalisability caveat on top of the unweighted-sample one.

### 1.1 Why this outcome and exposure, in numbers (all computed from the raw 2024 file)

- **Module coverage:** 96,287 respondents (21.0% of 457,670) were asked `CAREGIV1`; 94,670 have a valid caregiver status AND a valid check-up outcome; **87,098 complete cases** after our per-variable missingness rules (92.0% retention within the module sample).
- **Caregiver prevalence: 22.2%** (20,976 caregivers) - matching CDC's published ~1-in-5 figure, which validates our extraction.
- **Outcome event rate preserved: 18.2%** no check-up in the past year in the final sample - the same near-ideal classification rate as the full file.
- **The motivating puzzle - our data contradicts the popular narrative.** The caregiver-strain literature says caregivers postpone their own care. Our bivariate check says the opposite: caregivers have a LOWER gap rate (**15.0% vs 19.1%** for non-caregivers), and the difference persists within age bands (under-65: 21.1% vs 26.7%; 65+: 7.0% vs 7.4%). Caregivers in the module states skew female (60.0%) and slightly older - exactly the profile with higher baseline check-up attendance. Whether the caregiver "advantage" survives adjustment for age, sex, health status and access is precisely what the models will answer. Either result is interpretable and interesting: a persistent negative association suggests caregiving connects people to the health system (they are already in clinics with their care recipient); attenuation or reversal after adjustment would recover the neglect hypothesis for specific subgroups.
- **Coding trap handled:** `CAREGIV1` code 8 is NOT "none" - it means "expect to provide care within two years" (215 respondents). We recode 1→caregiver, 2→non-caregiver, and 7/8/9→missing, documenting the choice.

**Secondary outcome:** cost-related barrier to seeing a doctor, `MEDCOST1` (9.5% "yes") - "did care happen?" vs "why not?", now with the caregiver lens: do caregivers report more cost barriers even while attending more check-ups?

### 1.2 Why the caregiver angle strengthens the pitch

| Consideration | Generic Topic C | **Caregiver-focused Topic C (ours)** |
|---|---|---|
| Distinctiveness | Solid but likely picked by others | **A named exposure with a testable, counter-intuitive hypothesis - no other group will have it** |
| Codebook diligence evidence (3% criterion) | PRIMINS2 rename catch | **PRIMINS2 catch PLUS: optional-module detection, own extraction from the 922MB raw file, code-8 trap, skip-logic handling of CRGV* items** |
| Literature anchor | Access/equity generics | **Dedicated caregiver-health literature: Schulz & Beach 1999 (JAMA), Bevans & Sternberg 2012 (JAMA), Edwards et al. 2020 (MMWR - the BRFSS caregiver module itself)** |
| Public-health story | Outreach targeting | **Outreach targeting + caregiver-support policy (respite care, caregiver screening at the care-recipient's visits) - and a Singapore echo: AIC caregiver support grants** |
| Cost | - | 16-state subsample; N drops to 87k (still ample); one extra limitation paragraph |

### 1.3 Codebook diligence we can showcase (the 3% feasibility criterion)

1. **Optional-module awareness.** The brief itself warns that optional modules "vary by state and should be used with caution". We show exactly this caution: named the 16 participating states, quantified coverage (21.0%), and scoped claims to module states.
2. **Own extraction.** `CAREGIV1` (col 325) and companions were pulled from `LLCP2024.ASC` with the variable-dictionary column positions, then row-order-verified against the prof's extract via `_STATE` before merging - a documented, reproducible step in our notebook (`brfss2024_topic_c_caregiver.csv`, 41 columns).
3. **The code-8 trap.** In `CAREGIV1`, 8 = "expect to caregive in 2 years", not "none" - unlike day-count items where 88 = none → 0. Same digit, three different meanings across the codebook; we handle each per the codebook.
4. **Skip-logic discipline.** Care-intensity items (`CRGVHRS2` hours, `CRGVREL5` relationship, `CRGVLNG2` duration) are asked ONLY of caregivers - putting them in the main model would leak caregiver status. They appear only in a caregiver-side descriptive profile (the same reasoning that disqualified POORHLTH as a Topic A outcome).
5. **The 2024 insurance rename** (`PRIMINS2`, formerly `PRIMINSR`/`HLTHPLN1`) and **`_HLTHPL2` redundancy** (25,405 of 25,406 no-coverage rows are exactly `PRIMINS2`=88) - both retained from our original audit; `_HLTHPL2` stays dropped.
6. **`_INCOMG1` refusals kept as a category** (19.1% - too large and too non-random to drop).

### 1.4 Predictors (19, deliberately under the 20 cap)

We add **CAREGIV1 as the focal predictor** and drop `_URBSTAT` from the generic draft: the 16-state module footprint already distorts geographic composition, so individual urban/rural adds more noise than signal here and is better handled as a limitation. Everything else survives.

- **Focal exposure (1): `CAREGIV1`** - caregiver vs non-caregiver (22.2% caregivers).
- **Healthcare access (3): `PRIMINS2`, `PERSDOC3`, `MEDCOST1`** - insurance type, usual source of care, affordability. Verified gap-rate spreads of ~50, ~50 and ~22 percentage points in the full file.
- **Health status (2): `GENHLTH`, `MENTHLTH`** - sicker people have more scheduled contact (reverse gradient expected); mental distress is a help-seeking barrier and a documented caregiver-strain outcome (Edwards 2020: caregivers report more mentally-unhealthy days - making MENTHLTH an essential adjustment).
- **Chronic conditions (2): `DIABETE4`, `_MICHD`** - recall-system anchoring.
- **Disability (3): `DIFFWALK`, `DECIDE`, `DIFFALON`** - three distinct miss-a-visit mechanisms (mobility, cognition, independence).
- **Behaviours (2): `_SMOKER3`, `_TOTINDA`** - health-behaviour clustering.
- **Demographics/SES (6): `SEXVAR`, `_AGEG5YR`, `_RACEGR3`, `_EDUCAG`, `_INCOMG1`, `EMPLOY1`** - the confounding set that the caregiver puzzle makes decisive: caregivers are 60% female and older, both linked to attendance. Age × caregiver and sex × caregiver interactions are exactly what the random forest can surface.

### 1.5 Why these models (unchanged, now with a sharper interpretive target)

- **Baseline logistic regression** - the caregiver odds ratio, adjusted for the other 18 predictors, IS the headline result: does the raw caregiver advantage (15.0% vs 19.1%) survive adjustment?
- **Decision tree (CART, depth-limited)** - transparent segmentation; we will inspect where (if anywhere) CAREGIV1 appears in the tree relative to PERSDOC3/insurance/age.
- **Random forest** - interactions (caregiver × age, caregiver × sex, caregiver × distress) and permutation importance to cross-check the regression. Gradient boosting optional third.
- **No neural networks, no SVM** - per the brief's common-mistakes list.

### 1.6 Risks and mitigations

| Risk | Mitigation |
|---|---|
| 16-state module sample criticised as unrepresentative | Named up front; claims scoped to "module states analytic sample"; limitation statement extended accordingly. The brief's own warning about optional modules is quoted - caution demonstrated, not violated. |
| N drops to 87,098 | Still >4,000 events per predictor; power is a non-issue. We say so with numbers. |
| Caregiver result may be "null" after adjustment | Pre-framed as informative either way (connection hypothesis vs neglect hypothesis); the proposal never promises a direction. |
| CRGV* skip-logic leakage | Intensity items excluded from models by design; caregiver-only descriptive table instead. |
| MEDCOST1 double duty | As before: predictor in the primary model; target of the separate secondary run (removed from that predictor set). |
| Reverse gradient misread causally | Predictive framing; no causal language; cross-sectional caveat explicit. |

---

## PART 2 - What we will learn & what is required

### 2.1 Mapping to the seven learning outcomes

1. **Formulate a feasible healthcare analytics problem** - feasibility proven the hard way: exposure availability checked in the codebook, module coverage quantified (16 states, 21.0%), sample re-derived (87,098), event rate re-verified (18.2%).
2. **Data cleaning & feature engineering; handle missing/refused/inapplicable responsibly** - the full 7/9/77/99/8/88/blank taxonomy including the code-8 trap and skip-logic exclusion of CRGV* items; Member 1's core deliverable (6% final criterion).
3. **Descriptive analysis** - Table 1 stratified by outcome AND a caregiver-vs-non-caregiver profile; two charts: gap rate by caregiver status within age bands; gap rate by insurance type.
4. **Baseline regression + ML comparison** - logistic (adjusted caregiver OR), CART, random forest on one held-out test set.
5. **Interpret findings in public health terms** - connection vs neglect hypotheses; caregiver-support policy; Healthier SG / AIC caregiver-grant parallels.
6. **Communicate professionally** - one variable dictionary feeds proposal, notebook, report and slides.
7. **Teamwork** - five roles, whole-pipeline review.

### 2.2 Required artefacts and set-up

| Required artefact | Our set-up |
|---|---|
| Variable dictionary | Section 4 table, now including CAREGIV1 with the code-8 note |
| Missingness table | Per-variable decisions incl. module non-participation (79.0% structurally not asked - reported separately from item-missingness, a distinction the rubric will reward) |
| Data-flow diagram | 457,670 → 96,287 (module states) → 94,670 (valid outcome + caregiver status) → **87,098** complete cases |
| Table 1 + 2 charts | Outcome-stratified + caregiver profile; charts: caregiver × age-band gap rates; insurance-type gap rates |
| Model comparison table | 3 models × Accuracy/Sensitivity/Specificity/AUROC/AUPRC + dummy baseline |
| Interpretation questions | Pre-answerable with the adjusted caregiver OR as centrepiece |
| Limitation statement | Verbatim, extended with the module-subsample caveat |

---

## PART 3 - THE SUBMITTABLE PROPOSAL

# Caregiving and the Annual Check-Up: Do Caregivers Miss Their Own Preventive Care?

**ISSS623 Applied Data Science in Healthcare - Group Project Proposal (2024 BRFSS)**

## 1. Project question

Among respondents in the 2024 BRFSS analytic sample (states fielding the Caregiver module), can selected demographic, behavioural, healthcare access, chronic condition, disability, and **caregiving-status** variables predict **not having had a routine check-up within the past year**? Our focal question is whether informal caregivers differ from non-caregivers in attending their own preventive care once other characteristics are accounted for. As a secondary question on the same sample and framework, we examine whether the same variables predict **a cost-related barrier to seeing a doctor**.

## 2. Public health motivation

An estimated one in five US adults provides regular unpaid care to a family member or friend, and the caregiver-health literature has long warned that caregivers neglect their own health while caring for others (Schulz & Beach 1999; Bevans & Sternberg 2012). Routine check-ups are the entry point to preventive care, so a caregiver gap - if real - would compound into later diagnoses and higher downstream costs precisely in the population the health system depends on for informal care. Our verified extract of the 2024 BRFSS Caregiver module (16 states; 94,670 respondents with valid data; 22.2% caregivers) shows a counter-intuitive starting point: caregivers report FEWER check-up gaps than non-caregivers (15.0% vs 19.1%), a difference that persists within age bands. Whether this caregiver "advantage" survives adjustment for age, sex, health status, socioeconomic position and healthcare access - or conceals neglect within specific caregiver subgroups - is an open, actionable question: the answer informs caregiver-support policy such as respite provision and opportunistic caregiver screening during the care recipient's clinical visits. The question resonates in Singapore, where informal caregivers underpin ageing-in-place policy and are supported through AIC caregiver grants, and where Healthier SG anchors residents to one regular family doctor - the mechanism our strongest access predictor captures.

## 3. Brief literature context

Schulz and Beach (1999, JAMA) established caregiving as a risk factor for mortality; Bevans and Sternberg (2012, JAMA) reviewed the burden and health effects of family caregiving. Edwards et al. (2020, MMWR) profiled informal caregivers using this same BRFSS Caregiver module, documenting higher rates of mentally-unhealthy days among caregivers - motivating our inclusion of mental distress as an adjustment variable. For method, Clark et al. (2021) frame BRFSS machine-learning prediction as a health-equity targeting tool, and Parekh and Fahim (2021) provide the logistic-regression-versus-tree-ensemble recipe we adopt. We verified in the 2024 codebook that the insurance variable is now `PRIMINS2` (formerly `PRIMINSR`/`HLTHPLN1`) and that `CAREGIV1` sits in a state-optional module fielded by 16 states in 2024 - both checks material to feasibility.

## 4. Outcome variable and proposed predictors

All variables were checked in the 2024 BRFSS codebook. `CAREGIV1` and caregiver-detail items are not in the course-supplied extract; we extracted them from the raw `LLCP2024.ASC` using the course extraction notebook's fixed-width logic and merged them onto the supplied Topic C extract with row-order verification (working file `brfss2024_topic_c_caregiver.csv`, 41 columns). Throughout: 7/77 = don't know, 9/99 = refused, blank = not asked; in `CAREGIV1`, code 8 means "expects to provide care within two years" (recoded to missing, n = 215), not "none". We propose **19 predictors**, deliberately under the 20-variable cap; relative to a generic care-gap model we added `CAREGIV1` and removed `_URBSTAT` (the 16-state module footprint already constrains geographic generalisability), and we retain earlier exclusions: `PHYSHLTH` (near-collinear with `GENHLTH`), `_HLTHPL2` (verified redundant with `PRIMINS2` = 88), `MARITAL`, `CHCCOPD3`, `CHCKDNY2`. Caregiver-intensity items (`CRGVHRS2`, `CRGVREL5`, `CRGVLNG2`) are asked only of caregivers and are therefore used solely in a caregiver-profile descriptive table, never as model predictors.

| Concept | Variable | Role | Original coding | Recode |
|---|---|---|---|---|
| No routine check-up ≤1 yr | CHECKUP1 | **Primary outcome** | 1 ≤1yr; 2 ≤2yr; 3 ≤5yr; 4 >5yr; 8 never; 7/9 DK/ref | 1→0; 2/3/4/8→1; 7/9/blank→missing |
| Cost barrier to care | MEDCOST1 | **Secondary outcome** (also predictor in primary model) | 1 yes; 2 no; 7/9 | 1→1; 2→0; 7/9→missing |
| **Caregiver status (focal)** | **CAREGIV1** | **Predictor** | 1 yes; 2 no; 8 expects within 2 yrs; 7/9 DK/ref; blank = state did not field module | 1→1; 2→0; 7/8/9→missing; module non-participation reported separately |
| Insurance type | PRIMINS2 | Predictor | 1-10 types; 88 none; 77/99 | employer/private/Medicare/Medicaid-state/military-IHS/other-gov/uninsured(88)/not-reported(77,99) |
| Personal healthcare provider | PERSDOC3 | Predictor | 1 one; 2 more than one; 3 none; 7/9 | 1/2→1; 3→0; 7/9→missing |
| Self-rated general health | GENHLTH | Predictor | 1-5 excellent→poor; 7/9 | Ordinal 1-5; 7/9→missing |
| Mentally unhealthy days | MENTHLTH | Predictor | 1-30 days; 88 none; 77/99 | 88→0; keep 0-30; 77/99→missing |
| Diabetes | DIABETE4 | Predictor | 1 yes; 2 pregnancy-only; 3 no; 4 pre-diabetes; 7/9 | 1→1; 2/3/4→0; 7/9→missing |
| CHD or MI history | _MICHD | Predictor | 1 yes; 2 no; blank | 1→1; 2→0; blank→missing |
| Difficulty walking | DIFFWALK | Predictor | 1 yes; 2 no; 7/9; blank | 1→1; 2→0; else missing |
| Cognitive difficulty | DECIDE | Predictor | 1 yes; 2 no; 7/9; blank | 1→1; 2→0; else missing |
| Difficulty doing errands alone | DIFFALON | Predictor | 1 yes; 2 no; 7/9; blank | 1→1; 2→0; else missing |
| Smoking status | _SMOKER3 | Predictor | 1/2 current; 3 former; 4 never; 9 | Current/former/never/not-reported(9) |
| Any leisure physical activity | _TOTINDA | Predictor | 1 yes; 2 no; 9 | 1→1; 2→0; 9→missing |
| Sex | SEXVAR | Predictor | 1 M; 2 F | As is |
| Age group | _AGEG5YR | Predictor | 1-13 five-yr bands; 14 unknown | Bands 1-13; 14→missing |
| Race/ethnicity group | _RACEGR3 | Predictor | 1-5 groups; 9 | Groups 1-5; 9→missing |
| Education level | _EDUCAG | Predictor | 1-4; 9 | Levels 1-4; 9→missing |
| Income group | _INCOMG1 | Predictor | 1-7; 9 not reported | Levels 1-7 + not-reported(9) kept as category (19.1%) |
| Employment status | EMPLOY1 | Predictor | 1-8; 9 refused | Employed/unemployed/homemaker-student/retired/unable; 9/blank→missing |

**Data flow (verified in pandas on the raw file):** 457,670 respondents → 96,287 asked the Caregiver module (16 states, 21.0%) → 94,670 with valid caregiver status and check-up outcome → **87,098 complete cases (92.0% within-module retention)**. Event rate 18.2%; caregivers 22.2% (20,976). Bivariate preview: caregiver gap rate 15.0% vs non-caregiver 19.1%; under-65: 21.1% vs 26.7%; 65+: 7.0% vs 7.4%.

## 5. Planned models and analysis

We will (i) clean and recode per the table above, documenting every decision in a missingness table that separates structural module non-participation from item-level missingness, plus a data-flow diagram; (ii) produce Table 1 stratified by outcome, a caregiver-vs-non-caregiver profile (including the caregiver-only intensity items descriptively), and two charts (gap rate by caregiver status within age bands; gap rate by insurance type); (iii) fit a **baseline logistic regression** whose adjusted caregiver odds ratio is our headline estimand; (iv) fit **two machine-learning models - a depth-limited decision tree (CART)** for transparent decision rules and a **random forest** for nonlinear interactions (caregiver × age, caregiver × sex) and permutation importance (gradient boosting optional); (v) evaluate all models on a single stratified 70/30 held-out test set using **Accuracy, Sensitivity, Specificity, AUROC and AUPRC**, with a majority-class dummy baseline reported because the 18% event rate makes accuracy alone misleading; and (vi) interpret: does the caregiver association survive adjustment, does it make public-health sense, is it consistent with the descriptives, what are the limitations, and what additional data (caregiving intensity for all, state-level context, utilisation linkage) would help. No neural networks; no survey weights (_LLCPWT). *"This project is an unweighted respondent-level predictive modelling exercise using the 2024 BRFSS analytic sample. The results should not be interpreted as nationally representative prevalence estimates."* In addition, because the Caregiver module was fielded by 16 states in 2024, findings describe the module-state analytic sample and may not generalise to other states.

## 6. Team role allocation

| Role | Member | Responsibilities |
|---|---|---|
| Data Preparation Lead | [You] | Codebook verification (PRIMINS2 rename, CAREGIV1 code-8 trap, module coverage), raw-file extraction and merge of caregiver variables, recoding per variable dictionary, missingness table separating structural vs item missingness, data-flow diagram |
| Project & Proposal Lead | [Member 2] | Timeline and coordination, proposal drafting, GitHub repo and Colab management, submission logistics |
| Descriptive Analysis Lead | [Member 3] | Table 1, caregiver profile table (incl. CRGV* descriptive-only items), the two charts, bivariate checks anchoring interpretation |
| Modelling Lead | [Member 4] | Logistic baseline (adjusted caregiver OR), decision tree and random forest, interaction inspection, train/test protocol, five-metric comparison table |
| Interpretation & Report Lead | [Member 5] | Connection-vs-neglect interpretation, caregiver-policy framing, limitations, final report and presentation assembly |

All members review the full pipeline end-to-end; no artefact is owned by one person alone.

**References.** Schulz R, Beach SR. *JAMA* 1999;282(23):2215-2219. · Bevans M, Sternberg EM. *JAMA* 2012;307(4):398-403. · Edwards VJ, et al. *MMWR* 2020;69(7):183-188. · Clark CR, et al. *J Gen Intern Med* 2021;36(5):1181-1188. · Parekh T, Fahim F. *Drug Alcohol Depend* 2021;225:108789. · CDC, 2024 BRFSS Codebook (LLCP).
