# Predicting Preventive Care Gaps Among Caregivers: Evidence from the 2024 Behavioral Risk Factor Surveillance System

**ISSS623 Applied Healthcare Analytics — Group 1 Final Report**

| Member | Role |
|---|---|
| Cheong Hui Ying | Project lead |
| Chua Tze Yee | Data preparation |
| Chan Guang Hua | Descriptive analysis |
| Ziying Zhou | Modelling |
| Lee Yu Ting Tamara | Interpretation and report |

## 1. Problem Statement and Research Questions

We ask whether routinely collected survey characteristics can predict which adults miss their own routine preventive check-up, and whether informal caregivers differ from non-caregivers in this respect. We frame the prediction problem with the five-element checklist below.

> **Problem framing**
>
> - **Target population:** US adults in the 16 states that fielded the 2024 BRFSS Caregiver optional module (analytic sample n = 86,676, of whom 19,263 form the caregivers-only subset).
> - **Exposure of interest:** informal caregiver status (CAREGIV1: provided regular care or assistance to a family member or friend with a health problem or disability in the past 30 days).
> - **Primary outcome:** no routine check-up within the past year (CHECKUP1, recoded binary; justification in §7).
> - **Predictors:** 19 variables spanning demographics, socioeconomic position, health status, chronic conditions, disability, health behaviours, and healthcare access (Table 1; full dictionary in Annex A).
> - **Time frame:** cross-sectional — all predictors and outcomes refer to the 2024 survey reference periods (check-up: past 12 months; caregiving: past 30 days). There is no longitudinal prediction horizon; the models estimate current-state association, not future risk.

Three numbered research questions follow from this framing:

- **RQ1.** Among adults in the module-state analytic sample, can demographic, socioeconomic, health, disability and access variables predict not having attended a routine check-up in the past year?
- **RQ2.** Does caregiver status remain associated with the check-up gap after adjustment for these characteristics (connection versus neglect hypotheses — direction not presumed)?
- **RQ3.** Within caregivers, which characteristics — including caregiving relationship, intensity and duration — identify subgroups with a higher likelihood of a gap?

A proposed secondary analysis with a cost-barrier outcome (MEDCOST1) was descoped in response to feedback to use fewer, better-evaluated models; MEDCOST1 now enters the primary model as a predictor only (§3).

## 2. Background and Literature Context

Informal caregiving is increasingly recognised as a global, lifespan public health issue: population ageing and rising chronic disability are expanding the pool of unpaid family caregivers on whom health systems quietly depend (Haley & Elayoubi, 2024). In our 16-state sample, 22.2% of adults report caregiving in the past 30 days, consistent with national estimates that roughly one in five US adults is an informal caregiver (Edwards et al., 2020). Concern for caregivers' own health is long-standing: strained spousal caregiving has been reported as a mortality risk factor (Schulz & Beach, 1999), family caregiving carries documented physiological and psychological costs (Bevans & Sternberg, 2012), and caregivers in an earlier BRFSS cycle showed elevated risk of lacking healthcare coverage and of forgoing care because of cost (Tingey et al., 2020). The question also resonates in Singapore, where family members are the primary source of eldercare for a rapidly ageing population (Azman et al., 2024).

Our design has direct precedents: Edwards et al. (2020) profiled informal caregivers using the same BRFSS Caregiver module we analyse, shaping our adjustment set and validation benchmarks, and Tingey et al. (2020) used BRFSS to compare coverage and utilisation between caregivers and non-caregivers, anchoring our access-oriented predictors. A bivariate check of the 2024 data motivates a genuinely open question: caregivers record a *lower* unadjusted check-up gap rate than non-caregivers (15.0% versus 19.1%), but caregivers in the module states skew female and older — the profile with higher baseline attendance — so the unadjusted contrast cannot be taken at face value. Persistence of the difference after adjustment would support a *connection* hypothesis (caregiving keeps people engaged with the health system through the care recipient's appointments); attenuation or reversal would recover the *neglect* hypothesis. RQ2 adjudicates between these without presuming either. All analyses use the 2024 BRFSS public-use file and codebook (CDC, 2025).

## 3. Dataset and Variables

Our data source is the 2024 BRFSS annual file (LLCP2024.ASC; 457,670 respondents; CDC, 2025). The focal variable CAREGIV1 sits in the optional Caregiver module, fielded by 16 states in 2024 and absent from the course-supplied Topic C extract; we extracted it, with three caregiver-detail items, directly from the raw fixed-width file using the codebook column positions. The dataset combines the primary outcome (CHECKUP1) with 19 predictors. Table 1 condenses the variable dictionary to the decision-critical rows; the full dictionary with per-variable justifications is in Annex A.

**Table 1. Variable dictionary (key rows; full version in Annex A)**

| Variable | Description | Role | Original coding | Recode | Missing %* | Decision |
|---|---|---|---|---|---:|---|
| CHECKUP1 | Time since last routine check-up | Primary outcome | 1 ≤1 yr; 2 ≤2 yr; 3 ≤5 yr; 4 >5 yr; 8 never; 7/9 DK/refused | **1 → 0 (attended within past year)**; 2/3/4/8 → 1 (gap); 7/9 → missing | 1.12 | Complete-case |
| CAREGIV1 | Provided regular care to family/friend, past 30 days | Focal predictor | 1 yes; 2 no; 8 expects within 2 yrs; 7/9 | 1 → 1; 2 → 0; 7/8/9 → missing | 0.58 | Complete-case |
| MEDCOST1 | Could not see doctor due to cost, past 12 months | Predictor | 1 yes; 2 no; 7/9 | 1 → 1; 2 → 0; 7/9 → missing | 0.28 | Complete-case |
| PRIMINS2 | Primary insurance source | Predictor | 1–10 types; 88 none; 77/99 | 8 grouped categories; 88 → "uninsured"; 77/99 → "not reported" | 3.37 | Kept as category |
| PERSDOC3 | Has personal doctor/provider | Predictor | 1 one; 2 more than one; 3 none; 7/9 | 1/2 → 1; 3 → 0; 7/9 → missing | 0.87 | Complete-case |
| _INCOMG1 | Household income band | Predictor | 1–7 bands; 9 not reported | Bands 1–7 + explicit "not reported" category | 16.65 | Kept as category |
| _AGEG5YR | Age in 5-year bands | Predictor | 1–13 bands; 14 unknown | Ordinal 1–13; 14 → missing | 1.33 | Complete-case |
| SEXVAR | Sex | Predictor | 1 M; 2 F | As is | 0.00 | — |
| CRGVREL5 / CRGVHRS2 / CRGVLNG2 | Care recipient, weekly hours, duration | Descriptive only — never predictors | Asked only if CAREGIV1 = 1 | Caregiver profile table only | — | Leakage quarantine |

\* Item-level invalid share within the 96,287-respondent module sample.

**MEDCOST1 role clarification.** The proposal dictionary's "secondary outcome & predictor" label invited the reasonable objection that one variable sat on both sides of a model. Our design assigns MEDCOST1 one role per model, never both: a predictor in the primary (CHECKUP1) model, where affordability is a core access mechanism, and the target of the proposed *separate* secondary model from whose predictor set it would have been removed. Having descoped that secondary analysis (§1), MEDCOST1 carries exactly one role — predictor — and the dictionary label is corrected.

## 4. Data Preparation and Cohort Construction

Column positions for the caregiver fields were taken from the parsed codebook (CAREGIV1 at column 325) and read with a fixed-width parser; before positionally concatenating the extracted columns onto the Topic C extract, we asserted that the state identifier (_STATE) matched exactly, row for row, across all 457,670 records in both files — the appropriate alignment check, since BRFSS's SEQNO is not unique across states. Figure 1 presents the cohort cascade in the CONSORT-style format of clinical prediction studies (Lim et al., 2024).

**Figure 1. Data-flow diagram (2024 BRFSS → analytic samples)**

![Figure 1. CONSORT-style cohort cascade from the full 2024 BRFSS file to the main analytic sample (n = 86,676) and the caregivers-only subset (n = 19,263). Exclusion reasons and counts shown at right; format follows Lim et al. (2024).](results/data_flow_diagram.png)

All recodes are codebook-driven fixed mappings (CDC, 2025), so applying them before the stratified train/test split introduces no leakage; the full leakage-prevention design (intensity-item quarantine, no target-derived features, split-before-fit, single test-set evaluation) is described with the modelling methods (§8).

## 5. Missing-Data Strategy

Our decision rule: variables with item-level invalidity **below 5%** within the module sample are handled by complete-case exclusion, where deletion bias is negligible and the design remains simple and reproducible; variables **at or above 5%** retain invalid responses as an explicit "not reported" category, because deletion at that scale both shrinks and biases the sample — non-response correlates with the construct itself (income refusal correlates with socioeconomic position). Only _INCOMG1 (16.65%) crossed the threshold; PRIMINS2 (3.37%) was borderline and also kept as a category because insurance refusers adjoin the uninsured group. We considered and rejected imputation: single mean/median imputation distorts variance, and multiple imputation adds complexity disproportionate to item missingness below 2% on most variables. These conventions follow the BRFSS codebook's value schema (7/77 don't know, 9/99 refused, 88 "none" in count items, BLANK not asked), and income non-response is a documented BRFSS phenomenon, supporting categorical retention over deletion (CDC, 2025).

For the focal variable: of 96,287 module-sample respondents asked CAREGIV1, 21,137 answered yes, 74,592 no, 241 don't know, 215 "expects to provide care within two years" (code 8, recoded to missing — not "none", despite that digit's meaning elsewhere), and 102 refused — item invalidity of 0.58%. This is distinct from *structural* missingness: the 361,383 respondents (79.0%) never asked CAREGIV1 reflect a sampling-frame restriction, not non-response, and are reported separately (Annex A).

**Outcome recode and cut-point.** Correcting a typo the feedback caught, the mapping is **1 → 0** (attended within the past year) — there is no code 0 in CHECKUP1 — and codes 2/3/4/8 → 1 (gap). The one-year cut-point rests on three grounds: (i) it is the interval built into BRFSS's own question structure and used with this instrument in the caregiver literature (Tingey et al., 2020); (ii) an annual routine visit operationalises engagement with preventive care in practice guidance (e.g., the Medicare Annual Wellness Visit); and (iii) code 8 ("never") is a valid substantive value — the most severe form of the gap — not missing data. Because codes 2/3/4/8 form an ordinal gradient, we pre-specified a robustness threshold (gap = more than two years, codes 3/4/8; event rate 9.4%) to confirm findings are not artefacts of the one-year cut (§9.6).

## 6. Data Validation

Four systematic checks support the pipeline's correctness. **(1) External prevalence benchmark:** the caregiver share is 22.2% (19,263 of 86,676), consistent with the CDC's estimate that roughly one in five US adults is an informal caregiver — implausible had the fixed-width column offsets been wrong. **(2) Event-rate preservation:** the outcome event rate is 18.2% in the analytic sample, essentially identical to the full national file, so restriction to the 16 module states did not distort the outcome distribution. **(3) Codebook value-set check:** the extracted CAREGIV1 distribution (21,137 / 74,592 / 241 / 215 / 102 / 361,383 blank) matches the codebook's legal code set exactly, and the pre-merge _STATE assertion passed on all 457,670 rows. **(4) External profile consistency:** caregivers in our sample skew female (60.0%) and older, matching the published profile from the same BRFSS module (Edwards et al., 2020).

## 7. Outcome Justification and Limitations

CHECKUP1 operationalises the preventive care gap on three grounds: routine check-ups are the delivery vehicle through which preventive services are actually received, with visit attendance associated with receipt of screening and counselling services (Viera et al., 2006); the item is asked of all respondents, unlike age- or sex-gated screening items; and it is nearly complete (1.12% invalid). Its limitations are equally explicit: (i) it is self-reported and subject to recall bias; (ii) a check-up is not the totality of preventive care — dental visits, vaccinations, and specific screenings are distinct services; (iii) annual frequency is a convention rather than a universal clinical requirement, so a "gap" denotes deviation from the annual-visit norm, not clinical harm per se; and (iv) the item carries no information on visit quality or content. As a companion indicator, 31.7% of the analytic sample reported no dental visit within the past year (caregivers 31.2% vs non-caregivers 31.8%), a similar caregiver-neutral pattern that supports the check-up measure not being idiosyncratic.

## 8. Methods

### 8.1 Model line-up

Following the feedback to "use fewer models but evaluate them well," we restricted the final line-up to three models and dropped the XGBoost and soft-voting ensemble candidates from the proposal: (i) **logistic regression** as the interpretable baseline, whose adjusted odds ratios directly answer RQ2; (ii) a **depth-limited decision tree**, whose explicit rule structure shows how access, condition, and demographic variables partition risk; and (iii) a **random forest** to capture non-linearity and interactions the linear baseline cannot. A majority-class dummy classifier anchors the comparison floor. The 19 predictors were encoded into 38 model features (one-hot for nominal variables; ordered integers for ordinal bands).

### 8.2 Validation design

Our **internal validation** is a stratified 70/30 train/test split (n = 60,673 train; n = 26,003 test; event rate 18.2% in both partitions), with 5-fold stratified cross-validation on the training partition to assess stability. The test set was evaluated exactly once, for the final comparison table. Because true external validation would require a different survey year or dataset (out of scope), we probe **geographic transportability** with a leave-one-region-out design: for each Census region, we trained on the other three regions and evaluated on the held-out region — an adaptation of the leave-site-out logic recommended for health prediction models (Simon & Aliferis, 2024). We present this honestly as a transportability probe within the 16 module states, not as external validation.

### 8.3 Class imbalance

The primary outcome is 18.2% positive — mild imbalance, well above the severity (roughly below 5%) at which resampling earns its distortions. We therefore used **no resampling**; instead we set `class_weight='balanced'` in all three models and emphasised threshold-aware metrics over accuracy. One known consequence, assessed explicitly below, is that class weighting inflates the mean predicted probability, which degrades calibration even when discrimination is good.

### 8.4 Evaluation metrics

Each model is evaluated on the held-out test set with accuracy, sensitivity, specificity, AUROC, and AUPRC (baseline = the 18.2% event rate), plus two calibration measures added in response to feedback: the **Brier score** for all models and **10-bin reliability curves** (predicted versus observed event fraction) for the logistic and forest models.

## 9. Results

### 9.1 Descriptive highlights

All rates are unweighted analytic-sample proportions, not population prevalence estimates. Access variables show the steepest gradients: the gap rate is 58.4% among uninsured respondents (n = 4,879) versus 7.5% among Medicare enrolees (n = 28,751); 62.6% among respondents without a personal doctor versus 11.9% with one; and 37.4% among those reporting a cost barrier versus 16.2% without. Geographically, the gap ranges from 14.0% (Northeast) to 23.3% (West), with the Midwest at 18.2% and South at 16.2%; the raw caregiver advantage appears within every region (e.g., West: 19.1% caregivers vs 24.5% non-caregivers), so it is not one region's artefact. Table 2 breaks the gap rate down by the 16 module states: rates span a near-twofold range, from 12.2% (NH) to 24.2% (UT), with the four Western states (NV, OR, WY, UT) clustering at the high end.

**Table 2. Gap rate by module state (sorted ascending)**

| State | Gap rate | State | Gap rate |
|---|---:|---|---:|
| NH | 12.2% | DE | 16.9% |
| CT | 14.2% | WI | 18.0% |
| KY | 14.4% | IL | 18.9% |
| VA | 15.2% | SD | 19.2% |
| LA | 16.0% | NV | 22.0% |
| MS | 16.7% | OR | 22.8% |
| TX | 16.7% | WY | 23.1% |
| NC | 16.8% | UT | 24.2% |

### 9.2 Model comparison

**Table 3. Held-out test-set performance**

| Model | Accuracy | Sensitivity | Specificity | AUROC | AUPRC | Brier |
|---|---:|---:|---:|---:|---:|---:|
| Dummy (majority) | 0.818 | 0.000 | 1.000 | 0.500 | 0.182 | 0.149 |
| Logistic regression | 0.771 | 0.677 | 0.791 | **0.814** | 0.541 | 0.168 |
| Decision tree | 0.781 | 0.643 | 0.812 | 0.800 | 0.505 | 0.172 |
| Random forest | 0.768 | 0.680 | 0.787 | 0.813 | **0.553** | 0.167 |

The dummy's 81.8% accuracy — highest in the table while detecting zero gaps — illustrates why accuracy alone is misleading here. The logistic baseline and the forest are effectively tied on discrimination (AUROC 0.814 vs 0.813); the forest leads on AUPRC (0.553, three times the 0.182 baseline). Five-fold CV AUROCs for the logistic model (0.795, 0.804, 0.806, 0.822, 0.823; mean 0.810) bracket the test-set value, indicating no overfitting.

### 9.3 Calibration

Figure 2 presents the 10-bin reliability curves for the logistic and forest models against the perfect-calibration diagonal. Both models discriminate well but **systematically over-predict risk in every one of the ten bins** — both curves sit entirely below the diagonal: where the logistic model predicts a mean probability of 0.40, the observed event fraction is 12.9% (forest: 0.40 predicted, 13.6% observed), and in the top decile 0.91 predicted corresponds to 67.3% observed (forest: 0.86 vs 67.0%). This uniform upward offset is the expected artefact of `class_weight='balanced'`, which shifts the mean predicted probability above the 18.2% base rate; it also explains why the models' Brier scores (0.167–0.172) sit above the dummy's base-rate-constant 0.149 despite far superior discrimination. For any deployment that reads the probabilities literally, Platt scaling or isotonic recalibration is the standard remedy; rank-based uses (e.g., targeting the highest-risk decile for outreach) are unaffected.

**Figure 2. Reliability curves (10 bins, held-out test set)**

![Figure 2. Reliability curves for the logistic regression and random forest models. Both curves lie below the perfect-calibration diagonal, showing systematic over-prediction of risk.](results/calibration_plot.png)

### 9.4 What predicts the gap

Permutation importance for the forest and the logistic odds ratios converge on the same access-dominated story. The forest's top five — personal doctor (0.079), age band (0.028), diabetes (0.018), sex (0.006), uninsured status (0.005) — mirror the odds-ratio extremes: having a personal doctor OR 0.142, uninsured OR 1.841, cost barrier OR 1.758, diabetes OR 0.37, female OR 0.73. The tree tells the same story structurally: it splits first on personal doctor; among those without one, next on diabetes and then uninsured status; among those with one, on age band and then diabetes.

### 9.5 The caregiver estimate

Unadjusted, caregivers show a lower gap rate (15.0% vs 19.1%; OR 0.75, p < .001). After adjustment for the full predictor set, the association attenuates to **OR 0.957 (95% CI 0.901–1.017, p = .156)** — no longer statistically significant. The apparent "connection-hypothesis" advantage is therefore largely compositional: caregivers in these states skew older and female (Edwards et al., 2020), profiles already associated with higher attendance, and the raw advantage narrows within age strata (under 65: 20.5% vs 26.3%; 65+: 7.0% vs 7.3%). Caregiving status, per se, is not meaningfully associated with the check-up gap once who caregivers are is accounted for.

### 9.6 Robustness and transportability

Under the alternative more-than-two-years gap threshold, the event rate falls to 9.4% (caregivers 9.1% vs non-caregivers 9.5%) and the caregiver OR is 0.934 — the same attenuated, near-null conclusion, so the finding is not an artefact of the one-year cut-point. Leave-region-out AUROCs (Northeast 0.771, West 0.802, Midwest 0.807, South 0.826) closely bracket the internal CV range (0.795–0.823), with modest degradation in the Northeast (0.771) and marginally stronger performance in the South (0.826) — the model transports reasonably across the module footprint rather than perfectly.

### 9.7 Within caregivers (RQ3)

Among the 19,263 caregivers, the gap shows **no dose-response gradient by caregiving hours** (0–8 h/week: 14.8%; 9–19 h: 16.2%; 20–39 h: 15.3%). By care relationship, respondents caring for a father show the highest gap rate (28.6%, n = 829), against 16.8% for the most common group, caring for a mother (n = 6,051), and 10.7% for mother-in-law (n = 4,307) — suggesting that who is cared for differentiates risk more than caregiving intensity does.

## 10. Discussion and Interpretation

**Which variables mattered most, and do they make public-health sense?** Healthcare access dominated every view of the model: having a personal doctor was by far the most influential predictor and forms the tree's root split, with uninsured status and cost barriers the strongest risk-increasing factors (§9.4). This is expected — attachment to a usual source of care is the delivery mechanism for preventive services. The sex effect replicates the long-standing finding that men are less likely to receive routine check-ups (Viera et al., 2006), and the protective associations of diabetes and Medicare are consistent with chronic-disease management and age-based entitlement pulling people into regular contact with the system. The modelled odds ratios mirror the descriptive gradients (§9.1) with no direction reversals.

**The caregiver question (RQ2).** Unadjusted, caregivers appear advantaged; after adjustment the association is essentially null (§9.5): the raw "advantage" is explained by who caregivers are — predominantly female and older — rather than by caregiving itself. We interpret this fairly in both directions: at the population level we find evidence of neither systematic self-neglect (Schulz & Beach, 1999; Bevans & Sternberg, 2012) nor a strong connection effect whereby caregiving pulls people into care. Within caregivers (RQ3), gap rates varied modestly and non-monotonically by weekly hours and more widely by care relationship; these unweighted subgroup contrasts are hypothesis-generating only.

**Policy reading (stated cautiously).** Routine-check-up outreach is better targeted at access-poor profiles — adults with no personal doctor, no insurance, or cost barriers, particularly younger employed men — regardless of caregiver status. Caregiver-specific programmes remain justified on well-established stress and health-burden grounds (Haley & Elayoubi, 2024), but our results do not support check-up gaps as their rationale.

**Validation mapped to the research questions.** RQ1 is answered by internal validation (held-out AUROC 0.814; 5-fold CV 0.795–0.823) plus the geographic transportability probe (leave-one-region-out AUROC 0.771–0.826). RQ2 rests on the adjusted coefficient with its confidence interval, robust to the alternative >2-year outcome threshold. RQ3 is descriptive only and was never modelled.

**What additional data would help?** Longitudinal follow-up to establish whether gaps precede or follow caregiving onset; visit content and quality; care-recipient characteristics for all respondents; a broader preventive-care bundle (screenings, vaccinations, dental); linkage to claims data; and caregiver-module fielding in all states with design weights, enabling representative estimation.

## 11. Limitations

This project is an unweighted respondent-level predictive modelling exercise using the 2024 BRFSS analytic sample. The results should not be interpreted as nationally representative prevalence estimates. The Caregiver module was fielded by only 16 states in 2024, so all findings are scoped to the module-state analytic sample (n = 86,676) and may not transport to non-participating states.

The design is cross-sectional: predictors and outcome refer to overlapping 2024 reference periods, there is no prediction horizon, and no association reported here should be read causally. All measures are self-reported and subject to recall and social-desirability bias, and the outcome captures one indicator of preventive engagement, not its totality (§7). Finally, a calibration caveat: with balanced class weights, predicted probabilities systematically over-state observed risk (§9.3); discrimination is honest, and the scores are suitable for ranking outreach lists, but they would require recalibration (Platt scaling or isotonic regression) before use as literal probabilities.

## 12. Conclusion and Learning Points

Demographic, socioeconomic, health, and access variables predict the routine check-up gap well (AUROC of approximately 0.81), access measures dominate, and caregiver status adds essentially nothing after adjustment. Four course lessons shaped the work. First, the five-element framing checklist converted a vague concern into three answerable research questions before any code was written. Second, codebook-driven preparation mattered: the same digit means different things across BRFSS items, and structural missingness had to be separated from item non-response (CDC, 2025). Third, leakage discipline — quarantining skip-logic variables, one declared role per variable, splitting before fitting — protected the headline null from being an artefact. Fourth, evaluating fewer models well (three models, full metric battery including calibration, plus a geographic hold-out modelled on published validation practice; Lim et al., 2024; Simon & Aliferis, 2024) produced conclusions we can defend, including knowing precisely what the model can and cannot be used for.

## 13. References

Azman, N. D., Visaria, A., Goh, V. S., Østbye, T., Matchar, D., & Malhotra, R. (2024). Informal caregiving time and its monetary value in the context of older adults in Singapore. *Aging and Health Research, 4*(2), Article 100193. https://doi.org/10.1016/j.ahr.2024.100193

Bevans, M., & Sternberg, E. M. (2012). Caregiving burden, stress, and health effects among family caregivers of adult cancer patients. *JAMA, 307*(4), 398–403. https://doi.org/10.1001/jama.2012.29

Centers for Disease Control and Prevention. (2025). *Behavioral Risk Factor Surveillance System: 2024 codebook report and overview*. U.S. Department of Health and Human Services. https://www.cdc.gov/brfss/annual_data/annual_2024.html

Edwards, V. J., Bouldin, E. D., Taylor, C. A., Olivari, B. S., & McGuire, L. C. (2020). Characteristics and health status of informal unpaid caregivers — 44 states, District of Columbia, and Puerto Rico, 2015–2017. *MMWR Morbidity and Mortality Weekly Report, 69*(7), 183–188. https://doi.org/10.15585/mmwr.mm6907a2

Haley, W. E., & Elayoubi, J. (2024). Family caregiving as a global and lifespan public health issue. *The Lancet Public Health, 9*(1), e2–e3. https://doi.org/10.1016/S2468-2667(23)00227-X

Lim, S. L., Chan, S. P., Shahidah, N., Woo, K. L., Lam, S. S. W., Leong, B. S.-H., Lip, G. Y. H., & Ong, M. E. H. (2024). Validation of the NULL-EASE score for predicting survival in a multiethnic Asian cohort of out-of-hospital cardiac arrest. *Journal of the American Heart Association, 13*(16), e034133. https://doi.org/10.1161/JAHA.123.034133

Schulz, R., & Beach, S. R. (1999). Caregiving as a risk factor for mortality: The Caregiver Health Effects Study. *JAMA, 282*(23), 2215–2219. https://doi.org/10.1001/jama.282.23.2215

Simon, G. J., & Aliferis, C. (Eds.). (2024). *Artificial intelligence and machine learning in health care and medical sciences: Best practices and pitfalls*. Springer. https://doi.org/10.1007/978-3-031-39355-6

Tingey, J. L., Lum, J., Morean, W., Franklin, R., & Bentley, J. A. (2020). Healthcare coverage and utilization among caregivers in the United States: Findings from the 2015 Behavioral Risk Factor Surveillance System. *Rehabilitation Psychology, 65*(1), 63–71. https://doi.org/10.1037/rep0000307

Viera, A. J., Thorpe, J. M., & Garrett, J. M. (2006). Effects of sex, age, and visits on receipt of preventive healthcare services: A secondary analysis of national data. *BMC Health Services Research, 6*, Article 15. https://doi.org/10.1186/1472-6963-6-15

## Annex A — Variable Dictionary and Missingness

*This annex is supplementary reference material; its words do not count toward the 2,500–3,500-word main-text budget.*

**Table A1. Full variable dictionary (21 rows; invalid % measured within the 96,287-respondent module sample)**

| Variable | Description | Role | Original coding | Recode | Invalid % | Decision |
|---|---|---|---|---|---:|---|
| CHECKUP1 | Time since last routine check-up | Primary outcome | 1 ≤1 yr; 2 ≤2 yr; 3 ≤5 yr; 4 >5 yr; 8 never; 7/9 DK/refused | 1 → 0 (attended within past year); 2/3/4/8 → 1 (gap); 7/9 → missing | 1.12 | Complete-case |
| CAREGIV1 | Provided regular care to family/friend, past 30 days | Focal predictor | 1 yes; 2 no; 8 expects within 2 yrs; 7/9 | 1 → 1; 2 → 0; 7/8/9 → missing (8 is NOT "none") | 0.58 | Complete-case |
| MEDCOST1 | Could not see doctor due to cost, past 12 months | Predictor | 1 yes; 2 no; 7/9 | 1 → 1; 2 → 0; 7/9 → missing | 0.28 | Complete-case |
| PRIMINS2 | Primary insurance source | Predictor | 1–10 types; 88 none; 77/99 | 8 groups: employer(1)/private(2)/Medicare(3,4)/Medicaid-state(5,6,9)/military-IHS(7,8)/other-gov(10)/uninsured(88)/not reported(77,99) | 3.37 | Kept as category |
| PERSDOC3 | Has personal doctor/provider | Predictor | 1 one; 2 more than one; 3 none; 7/9 | 1/2 → 1; 3 → 0; 7/9 → missing | 0.87 | Complete-case |
| GENHLTH | Self-rated general health | Predictor | 1–5 excellent→poor; 7/9 | Ordinal 1–5; 7/9 → missing | 0.27 | Complete-case |
| MENTHLTH | Days of poor mental health, past 30 days | Predictor | 1–30 days; 88 none; 77/99 | 88 → 0; 1–30 kept; 77/99 → missing | 1.68 | Complete-case |
| DIABETE4 | Ever told had diabetes | Predictor | 1 yes; 2 pregnancy-only; 3 no; 4 pre-diabetes; 7/9 | 1 → 1; 2/3/4 → 0; 7/9 → missing | 0.19 | Complete-case |
| _MICHD | Coronary heart disease or myocardial infarction (calculated) | Predictor | 1 yes; 2 no; blank | 1 → 1; 2 → 0; blank → missing | 1.09 | Complete-case |
| DIFFWALK | Serious difficulty walking or climbing stairs | Predictor | 1 yes; 2 no; 7/9; blank | 1 → 1; 2 → 0; else missing | 0.42 | Complete-case |
| DECIDE | Difficulty concentrating, remembering, or deciding | Predictor | 1 yes; 2 no; 7/9; blank | 1 → 1; 2 → 0; else missing | 0.63 | Complete-case |
| DIFFALON | Difficulty doing errands alone | Predictor | 1 yes; 2 no; 7/9; blank | 1 → 1; 2 → 0; else missing | 0.33 | Complete-case |
| _SMOKER3 | Smoking status (calculated) | Predictor | 1/2 current; 3 former; 4 never; 9 | current/former/never; 9 → missing | 0.63 | Complete-case |
| _TOTINDA | Any leisure-time physical activity, past 30 days (calculated) | Predictor | 1 active; 2 inactive; 9 | 1 → 1; 2 → 0; 9 → missing | 0.24 | Complete-case |
| SEXVAR | Sex | Predictor | 1 M; 2 F | As is | 0.00 | — |
| _AGEG5YR | Age in 5-year bands (calculated) | Predictor | 1–13 bands; 14 unknown | Ordinal 1–13; 14 → missing | 1.33 | Complete-case |
| _RACEGR3 | Race/ethnicity group (calculated) | Predictor | 1–5 groups; 9 | Groups 1–5; 9 → missing | 1.66 | Complete-case |
| _EDUCAG | Education level (calculated) | Predictor | 1–4 levels; 9 | Ordinal 1–4; 9 → missing | 0.35 | Complete-case |
| _INCOMG1 | Household income band | Predictor | 1–7 bands; 9 not reported | Bands 1–7 + explicit "not reported" category | 16.65 | Kept as category |
| EMPLOY1 | Employment status | Predictor | 1–8 statuses; 9 refused | Grouped: employed(1,2)/unemployed(3,4)/homemaker-student(5,6)/retired(7)/unable(8); 9/blank → missing | 0.83 | Complete-case |
| CRGVREL5 / CRGVHRS2 / CRGVLNG2 | Care recipient relationship, weekly caregiving hours, caregiving duration | Descriptive only — never predictors | Asked only if CAREGIV1 = 1 (skip logic) | Caregiver profile table only | — | Leakage quarantine |

**Table A2. Missingness table with per-variable justification (invalid = don't know / refused / blank, per codebook conventions; CDC, 2025)**

| Variable | Invalid n | Invalid % | Decision | Justification |
|---|---:|---:|---|---|
| CHECKUP1 | 1,080 | 1.12 | Exclude row | Missing outcome cannot be modelled; <5% rule — deletion bias negligible |
| CAREGIV1 | 558 | 0.58 | Exclude row | Focal predictor; <5% rule |
| MEDCOST1 | 274 | 0.28 | Exclude row | <5% rule |
| PRIMINS2 | 3,245 | 3.37 | Keep 77/99 as "not reported" category | Borderline; insurance refusers adjoin the uninsured group — retaining them preserves that signal |
| PERSDOC3 | 838 | 0.87 | Exclude row | <5% rule |
| GENHLTH | 256 | 0.27 | Exclude row | <5% rule |
| MENTHLTH | 1,620 | 1.68 | Exclude row (after 88 → 0) | <5% rule; 88 is a valid zero, not missing |
| DIABETE4 | 186 | 0.19 | Exclude row | <5% rule |
| _MICHD | 1,049 | 1.09 | Exclude row | <5% rule |
| DIFFWALK | 401 | 0.42 | Exclude row | <5% rule |
| DECIDE | 605 | 0.63 | Exclude row | <5% rule |
| DIFFALON | 315 | 0.33 | Exclude row | <5% rule |
| _SMOKER3 | 611 | 0.63 | Exclude row | <5% rule (module-sample rate is low) |
| _TOTINDA | 233 | 0.24 | Exclude row | <5% rule |
| SEXVAR | 0 | 0.00 | — | No invalid values |
| _AGEG5YR | 1,282 | 1.33 | Exclude row | <5% rule |
| _RACEGR3 | 1,598 | 1.66 | Exclude row | <5% rule |
| _EDUCAG | 338 | 0.35 | Exclude row | <5% rule |
| _INCOMG1 | 16,035 | 16.65 | Keep code 9 as "not reported" category | ≥5% rule — deletion at this scale both shrinks and biases the sample; income refusal correlates with socioeconomic position, the construct itself |
| EMPLOY1 | 798 | 0.83 | Exclude row | <5% rule |

**Decision rule (restated).** Item-level invalidity **below 5%** within the module sample → complete-case exclusion (deletion bias negligible; simple and reproducible). Invalidity **at or above 5%** → invalid responses retained as an explicit "not reported" category, because deletion at that scale both shrinks and biases the sample — non-response correlates with the construct itself.

**Structural versus item missingness.** The 361,383 respondents (79.0% of the full 2024 file) who were never asked CAREGIV1 reflect a sampling-frame restriction (their state did not field the Caregiver module), not non-response; they are excluded by design and do not appear in Table A2, which measures item-level invalidity within the 96,287-respondent module sample only. Among the caregivers, the descriptive-only detail items are nearly complete: CRGVREL5 99.9% valid, CRGVHRS2 98.7%, CRGVLNG2 98.5%.

<!-- word count: 3360 (main text §1–§12, excluding title block, tables, figures and captions, reference list, and Annex A) -->
