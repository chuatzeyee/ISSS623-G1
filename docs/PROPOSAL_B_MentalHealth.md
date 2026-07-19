# Proposal B — Mental Health: Predicting Frequent Mental Distress Among 2024 BRFSS Respondents

> Topic B option for the ISSS623 group project. All statistics below were computed directly from
> `data/brfss2024_topic_b.csv` (457,670 respondents, 33 columns) — nothing is quoted from memory.
> **Status: fully feasible, but not our first recommendation** (see trade-offs in Part 1.6).

---

## PART 1 — Decision rationale (read this before the group meeting)

### 1.1 The critical judgement call: the brief's example outcome does not exist in our data

The project brief lists "daily marijuana use" as an example Topic B outcome. **We checked: no marijuana
variable (MARIJAN1/MJUSE, etc.) exists in any of the three supplied extracts**, because marijuana
questions sit in *state-optional* BRFSS modules that the professor's extraction did not include (and which
only a subset of states administer, so they would also have severe structural missingness). The brief's own
"common mistakes" list puts *"choosing an outcome without checking the codebook"* at #1. Pivoting away
from the example outcome — with documented evidence — is therefore not a compromise; it is exactly the
masters-level judgement the rubric's largest proposal criterion (data feasibility, 3%) is designed to reward,
and we say so explicitly in the proposal.

### 1.2 The replacement outcome: frequent mental distress (FMD), and why it is the right one

We pivot to **frequent mental distress (FMD)**: `MENTHLTH ≥ 14` days of poor mental health in the past
30 days. Verified facts:

- `MENTHLTH` is asked of **all** respondents in topic_b.csv: only **3 blanks** in 457,670 rows. Codes:
  88 = "none" (269,909 rows → recode to 0), 77 = don't know (5,594; 1.2%), 99 = refused (2,559; 0.6%).
- After dropping 77/99/blank, **449,514 valid responses** remain and the FMD rate is **13.86%** —
  a healthy event rate for classification (not so rare that models collapse, not so common that the
  problem is trivial).
- The ≥14-day cut-off is the **standard CDC surveillance definition** used in Cree et al. (MMWR 2020)
  and Miyakado-Steger & Seidel (Prev Chronic Dis 2019) — both are direct FMD-on-BRFSS precedents,
  so our literature section writes itself and our results are directly comparable to published baselines.
- Contrast with the tempting alternative `POORHLTH`: in topic_a it is 41.4% blank due to skip logic —
  a trap outcome. `MENTHLTH` has no skip logic. This comparison goes in the proposal to show codebook
  awareness.

### 1.3 The angle: mental health burden + disability equity

Topic B's extract is the **only one containing the full 6-item ACS disability battery**
(DEAF, BLIND, DECIDE, DIFFWALK, DIFFDRES, DIFFALON). This lets us replicate the core finding of
Cree et al. 2020 (FMD is several times higher among adults with disabilities) inside a predictive-modelling
frame. Measured in our own analytic sample:

- **30.8%** of complete-case respondents report at least one disability.
- FMD is **26.4%** with any disability vs **7.8%** without — a **3.4×** gap. This is our headline
  descriptive chart and gives the interpretation section a genuine public-health story (equity in mental
  health burden), not just a metrics table.

### 1.4 Predictor selection: 27 candidates trimmed to exactly 20, with reasons

topic_b.csv offers 27 plausible predictors; the brief caps us at 20. Our cuts and keeps (all codings verified):

**Keep (20):**
| Domain | Variables | Why |
|---|---|---|
| Disability (6) | DEAF, BLIND, DECIDE, DIFFWALK, DIFFDRES, DIFFALON | The Cree equity angle; blanks only 3.3–4.5% each |
| Behaviour (2) | _SMOKER3, _TOTINDA | Strongest literature links to mental distress; low missingness |
| Access (3) | PERSDOC3, MEDCOST1, _HLTHPL2 | Cost barrier (MEDCOST1 yes = 9.5%) is a classic distress correlate |
| Chronic (3) | _MICHD, DIABETE4, CHCCOPD3 | Major comorbidity axes; each ≤1.2% missing |
| Demographic/SES (6) | SEXVAR, _AGEG5YR, _RACEGR3, _EDUCAG, _INCOMG1, EMPLOY1 | Core Table-1 stratifiers; EMPLOY1 "unable to work" is a known strong FMD signal |

**Cut (7), with documented reasons — this is rubric gold:**
- `_RFDRHV9` and (as predictor) alcohol generally: our **secondary outcome is _RFBING6** (binge drinking);
  keeping alcohol variables as predictors of FMD while binge drinking is itself an outcome muddles the design.
  `_RFDRHV9` also has 10.2% code-9 missingness.
- `_CURECI3` (e-cigarettes): construct overlap with _SMOKER3; keeps the behaviour domain parsimonious.
- `_BMI5CAT`: **9.4% missing** — the single biggest complete-case cost on offer (~40k rows) for a weak
  theoretical link to FMD. Cutting it is a deliberate, documented missingness decision.
- `CHECKUP1`: access already covered by three cleaner variables; CHECKUP1 adds messy code 8 ("never") handling.
- `CVDSTRK3`, `CHCKDNY2`: low prevalence (4.5%, 5.2%) and overlap the cardiometabolic axis already
  captured by _MICHD and DIABETE4.
- `MARITAL`: plausible but expendable; noted as a sensitivity-analysis candidate if the group wants it back
  (swap for a weaker predictor, staying ≤20).

### 1.5 The DECIDE problem: a leakage-adjacent variable we handle both ways

`DECIDE` asks about *"serious difficulty concentrating, remembering, or making decisions because of a
physical, mental, or emotional condition."* That is construct-adjacent to the outcome: measured
FMD is **47.4%** among DECIDE = yes vs **9.1%** among DECIDE = no (prevalence 11.7%). It is not literal
leakage — it is a distinct ACS disability item, and Cree et al. treat cognitive disability as a legitimate
FMD correlate — but a naive model will rank it #1 in importance and a careless write-up would over-read that.
**Decision: keep DECIDE in the primary specification (faithful to the disability-equity literature) and
re-fit all models without it as a pre-registered sensitivity analysis, reporting both.** Discussing this
trade-off openly is precisely the "cautious interpretation" the rubric asks for; hiding it would be the
"unexplainable model" mistake.

### 1.6 Why these models

- **Baseline: logistic regression** — required, interpretable odds ratios, direct comparability to
  Miyakado-Steger & Seidel's regression approach.
- **ML model 1: decision tree** — the brief explicitly recommends it for interpretability; the tree's first
  splits (likely DECIDE/DIFFALON/EMPLOY1) will visualise the equity story.
- **ML model 2: random forest** — the stronger nonlinear ensemble, and the exact LR→DT→RF ladder used by
  Parekh & Fahim (2021) on BRFSS substance-use outcomes, giving us a methodological citation for the whole
  design. Gradient boosting only as an optional annex if time allows; no neural nets, per the brief.
- **Class imbalance (13.6% positive)**: stratified 70/30 split, report AUPRC alongside AUROC, discuss
  threshold choice rather than tuning to accuracy (accuracy alone would reward predicting "no FMD" for everyone).

### 1.7 Honest trade-offs vs Topics A and C (why this is NOT our top recommendation)

- **Against B:** (i) the marijuana pivot must be explained carefully or a rushed reader may think we ignored
  the brief; (ii) topic_b.csv lacks GENHLTH/PHYSHLTH/ADDEPEV3, so we cannot use general health or a
  depression diagnosis as predictors — ADDEPEV3 in particular would have been a strong (if leakage-adjacent)
  covariate that Topic A groups will have; (iii) the DECIDE overlap needs the two-specification handling above.
- **For B:** the disability battery is unique to this extract, the FMD literature is the most directly
  on-point of all three topics (two anchor papers use our exact outcome definition), and the outcome is nearly
  missingness-free.
- **Net:** fully feasible and defensible, but Topic A (poor/fair general health, 19.7% event rate, no pivot
  story needed) is the safer path. If the group prefers a distinctive equity narrative and is comfortable
  defending the pivot, B is a strong choice.

### 1.8 Risks and mitigations

| Risk | Mitigation |
|---|---|
| Pivot from brief's example outcome questioned | One paragraph in proposal documenting the codebook check; cite "common mistake #1" |
| DECIDE dominates importance rankings | Dual specification (with/without); interpret as construct overlap, not discovery |
| Complete-case drop looks large (457,670 → ~374,388; 81.8% retained) | Data-flow diagram with per-step counts; missingness table showing every variable's 7/9/blank rates and decision |
| Income refusals 19% raw | Keep code 9 as an explicit "not reported" category (13.9% of final sample) — refusal itself may predict distress |
| Imbalance (13.6%) inflates accuracy | Report Sensitivity/Specificity/AUROC/AUPRC; never lead with accuracy |
| Over-claiming | Required limitation statement verbatim; "analytic sample" language throughout |

---

## PART 2 — What we will learn & what is required

### Mapping to the seven course learning outcomes

1. **Healthcare data landscape** — BRFSS as the archetypal population telephone survey: skip logic,
   proxy codes (7/9/77/88/99), calculated variables (underscore-prefixed), state-optional modules
   (the marijuana lesson).
2. **Formulating a feasible analytics question** — the marijuana→FMD pivot is a live case study in
   verifying outcome availability *before* committing.
3. **Data preparation** — recoding 88→0, dropping 77/99, handling blank-by-design, building the
   recode dictionary; the deepest-weighted skill in both rubrics (3% proposal + 6% final).
4. **Descriptive analysis** — Table 1 stratified by FMD, outcome distribution, disability-gap chart.
5. **Predictive modelling** — one interpretable baseline (logistic) plus two ML models (tree, forest)
   on an imbalanced binary outcome.
6. **Model evaluation** — Accuracy/Sensitivity/Specificity/AUROC/AUPRC comparison; why AUPRC matters
   at a 13.6% event rate.
7. **Cautious interpretation & communication** — variable importance vs causation, the DECIDE overlap
   discussion, unweighted-sample limitation, equity framing.

### Required artefacts and how this topic satisfies each

| Required artefact | Our plan |
|---|---|
| Variable dictionary (name/concept/original coding/recode/role) | 22-row table drafted in Part 3 §4; expanded in annex |
| Missingness table with per-variable decisions | 7/9/blank rates already measured for all 20 predictors; decisions documented (drop, recode-to-0, keep-as-category) |
| Data-flow diagram | 457,670 → 449,514 (valid MENTHLTH; −8,156) → ≈374,388 complete-case (−75,126); 81.8% retained |
| Table 1 + ≥2 charts | Descriptives by FMD status; Chart 1: FMD by disability count (0/1/2+); Chart 2: FMD by income group |
| Model comparison table | LR vs DT vs RF × {Accuracy, Sensitivity, Specificity, AUROC, AUPRC} on held-out 30% |
| Interpretation questions | Importance rankings (both DECIDE specifications); public-health sense-check vs Cree 2020; consistency with descriptives; limitations; "what extra data": depression diagnosis (ADDEPEV3), social support, adverse childhood experiences modules |
| Limitation statement | Quoted verbatim in Part 3 |

---

## PART 3 — THE SUBMITTABLE PROPOSAL

# Predicting Frequent Mental Distress Among 2024 BRFSS Respondents

**ISSS623 Applied Data Science in Healthcare — Group Project Proposal (Topic B: Mental Health)**

## 1. Project question

Among respondents in the 2024 BRFSS analytic sample, can selected demographic, behavioural, healthcare
access, chronic condition, and disability variables predict **frequent mental distress** (FMD), defined by
the CDC surveillance standard of 14 or more poor-mental-health days in the past 30 days (`MENTHLTH ≥ 14`)?
As a secondary outcome, we will apply the same pipeline to the calculated **binge-drinking indicator**
(`_RFBING6`), a closely linked behavioural-health outcome.

*A note on outcome selection:* the brief's example Topic B outcome (daily marijuana use) sits in
state-optional BRFSS modules that are **not present in the supplied 2024 extract**; we verified this against
the extract columns and the 2024 codebook before selecting FMD, which is asked of all respondents (only 3
blank values in 457,670 records). We chose FMD deliberately: it is the standard CDC definition used in prior
BRFSS mental-health surveillance research, ensuring both feasibility and comparability.

## 2. Public health motivation

Frequent mental distress marks the population whose mental-health burden is persistent rather than episodic,
and it predicts unmet treatment need, health-risk behaviours, and downstream chronic disease. In our
2024 analytic sample, 13.9% of respondents meet the FMD threshold — but the burden is starkly unequal:
among respondents reporting at least one functional disability (30.8% of the sample), FMD prevalence is
26.4%, versus 7.8% among those reporting none, a 3.4-fold gap. Identifying which routinely collected
survey characteristics best discriminate FMD can help target screening and community mental-health outreach
toward the highest-burden groups, particularly people with disabilities — an explicit health-equity priority.

## 3. Brief literature context

Cree et al. (MMWR 2020) documented, using BRFSS and this same ≥14-day definition, that FMD among adults with
disabilities is several times that of adults without, establishing our equity framing. Miyakado-Steger &
Seidel (Prev Chronic Dis 2019) modelled FMD risk factors in BRFSS data for an Austin-area sample, finding
income, employment, and chronic conditions to be dominant correlates — informing our predictor domains.
Methodologically, Parekh & Fahim (Drug Alcohol Depend 2021) applied logistic regression, decision tree, and
random forest classifiers to BRFSS behavioural-health outcomes; we adopt their model ladder and evaluation
approach. Our contribution is applying this design to the 2024 cycle with the full six-item disability battery.

## 4. Outcome variable and proposed predictors (20 predictors)

Outcome recoding: `MENTHLTH` 88→0 days; 77 (don't know, 1.2%) and 99 (refused, 0.6%) dropped; FMD = 1 if
days ≥ 14. Blank disability items (3.3–4.5%, skip-logic/module non-response) are dropped complete-case;
income code 9 ("not reported", 13.9% of the final sample) is retained as an explicit category because
refusal may itself carry signal. Estimated data flow: 457,670 → 449,514 valid outcome → ≈374,000
complete-case analytic sample (≈82% retained; FMD 13.6%).

| Concept | Variable | Role | Original coding | Recode |
|---|---|---|---|---|
| Poor mental health days | MENTHLTH | Primary outcome | 1–30 days; 88 none; 77 DK; 99 refused | 88→0; drop 77/99; FMD = days ≥ 14 |
| Binge drinking | _RFBING6 | Secondary outcome | 1 no; 2 yes; 9 missing | 2→1, 1→0; drop 9 |
| Deafness/hearing | DEAF | Predictor | 1 yes; 2 no; 7/9; blank | 1/0; drop 7/9/blank |
| Blind/vision difficulty | BLIND | Predictor | as DEAF | as DEAF |
| Cognitive difficulty | DECIDE | Predictor | as DEAF | as DEAF; sensitivity run without (construct overlap with outcome) |
| Difficulty walking | DIFFWALK | Predictor | as DEAF | as DEAF |
| Difficulty dressing | DIFFDRES | Predictor | as DEAF | as DEAF |
| Difficulty with errands | DIFFALON | Predictor | as DEAF | as DEAF |
| Smoking status | _SMOKER3 | Predictor | 1–4; 9 missing | 4 categories (current daily/some-day/former/never); drop 9 |
| Physical activity | _TOTINDA | Predictor | 1 active; 2 not; 9 | 1/0; drop 9 |
| Personal doctor | PERSDOC3 | Predictor | 1 yes-one; 2 yes-multiple; 3 no; 7/9 | 3 categories; drop 7/9 |
| Cost barrier to care | MEDCOST1 | Predictor | 1 yes; 2 no; 7/9 | 1/0; drop 7/9 |
| Health coverage | _HLTHPL2 | Predictor | 1 yes; 2 no; 9 | 1/0; drop 9 |
| CHD or MI | _MICHD | Predictor | 1 yes; 2 no; blank | 1/0; drop blank |
| Diabetes | DIABETE4 | Predictor | 1 yes; 2 gestational; 3 no; 4 borderline; 7/9 | yes vs not (2,3,4→0); drop 7/9 |
| COPD | CHCCOPD3 | Predictor | 1 yes; 2 no; 7/9 | 1/0; drop 7/9 |
| Sex | SEXVAR | Predictor | 1 male; 2 female | binary; no missing |
| Age group | _AGEG5YR | Predictor | 1–13 five-yr bands; 14 missing | ordinal 13 bands; drop 14 |
| Race/ethnicity | _RACEGR3 | Predictor | 1–5; 9 missing | 5 categories; drop 9 |
| Education | _EDUCAG | Predictor | 1–4; 9 | ordinal 4 levels; drop 9 |
| Income group | _INCOMG1 | Predictor | 1–7; 9 not reported | 7 levels + "not reported" category |
| Employment | EMPLOY1 | Predictor | 1–8; 9 refused; blank | 8 categories; drop 9/blank |
| Any disability (derived) | (from 6 items) | Subgroup | — | ≥1 of six items = 1; descriptive stratifier only |

We considered and excluded seven further candidates, documenting each decision: `_BMI5CAT` (9.4% missing —
the largest avoidable complete-case cost), `_RFDRHV9` and `_CURECI3` (domain overlap with the secondary
outcome and smoking respectively), `CHECKUP1` (access already covered; complex code-8 handling),
`CVDSTRK3`/`CHCKDNY2` (low prevalence, overlapping the cardiometabolic axis), and `MARITAL`
(held as a sensitivity swap candidate).

## 5. Planned models and analysis plan

Cleaning and recoding per the table above, with a missingness table (per-variable 7/9/blank rates and the
decision taken) and a data-flow diagram from 457,670 raw records to the final analytic sample. Descriptive
analysis: Table 1 of all predictors stratified by FMD status, outcome distribution, and two charts (FMD by
disability count; FMD by income group). Modelling on a stratified 70/30 train/test split: (i) **baseline
logistic regression** with odds ratios; (ii) **decision tree** (depth-limited for interpretability, as the
brief recommends); (iii) **random forest** as the stronger nonlinear learner — mirroring the
Parekh & Fahim BRFSS model ladder. Given the 13.6% event rate we will compare models on **Accuracy,
Sensitivity, Specificity, AUROC, and AUPRC**, emphasising AUPRC and threshold choice over raw accuracy.
Because `DECIDE` (cognitive difficulty) is construct-adjacent to FMD, all models will be fitted **with and
without DECIDE** and both importance rankings reported. Interpretation will compare variable importance
against the descriptive patterns and prior literature, answer whether findings make public-health sense, and
state limitations, including what additional data (e.g., a depression-diagnosis variable, social-support and
adverse-childhood-experience modules) would strengthen the work. The same pipeline is then applied to the
secondary outcome `_RFBING6`.

**Required limitation statement:** *"This project is an unweighted respondent-level predictive modelling
exercise using the 2024 BRFSS analytic sample. The results should not be interpreted as nationally
representative prevalence estimates."*

## 6. Team role allocation

| Role | Member | Responsibilities |
|---|---|---|
| Data Preparation Lead | [You] | Codebook verification (incl. the outcome-availability check), recode dictionary, missingness table and decisions, data-flow diagram, analytic-sample construction |
| Project & Proposal Lead | [Member 2] | Timeline and coordination, proposal drafting, GitHub repo management, submission logistics |
| Descriptive Analysis Lead | [Member 3] | Table 1, outcome distribution, disability-gap and income charts, descriptive narrative |
| Modelling Lead | [Member 4] | Logistic baseline, decision tree, random forest, DECIDE sensitivity runs, evaluation metrics and comparison table |
| Interpretation & Report Lead | [Member 5] | Variable-importance interpretation, equity framing, limitations, final report and presentation slides |

All members review the full pipeline end-to-end; no artefact is submitted on a single member's checking alone.

**References.** Cree RA, et al. Frequent mental distress among adults, by disability status, disability type,
and selected characteristics — United States, 2018. MMWR 2020;69(36):1238–1243. · Miyakado-Steger H,
Seidel S. Using BRFSS data to assess mental health, Travis County, Texas, 2011–2016. Prev Chronic Dis
2019;16:180449. · Parekh T, Fahim F. Building risk prediction models for daily use of marijuana using
machine learning techniques. Drug Alcohol Depend 2021;225:108789.
