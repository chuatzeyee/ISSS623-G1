# Predictor Selection: What Predictors Are, and How Ours Were Derived

**ISSS623 Group Project - Topic C (caregiver angle) · Companion to `PROPOSAL_C_DataPrep_Methodology.md`**
*Proposal-ready prose plus reference tables. All counts verified against the 2024 BRFSS files; see the methodology document for the measured missingness table.*

---

## 1. What a predictor is

A predictor - also called a feature, independent variable, or covariate - is any variable a model uses as **input** to estimate the **outcome**. In our design the outcome is the check-up gap (CHECKUP1, recoded to 0 = attended within a year, 1 = gap), and the predictors are the nineteen respondent characteristics from which the models estimate the probability of that gap: caregiver status, insurance type, presence of a personal doctor, cost barriers, health status, chronic conditions, disability, health behaviours, and demographic and socioeconomic position.

The direction of the relationship matters. Predictors must be things one would plausibly know about a respondent **before** knowing the outcome; the model's job is to map from those characteristics to a probability of the outcome. This is why the project brief phrases every suggested question in the same template - "can selected demographic, behavioural, healthcare access, chronic condition, and disability variables predict [outcome]?" - the five domains named in that template are the sanctioned predictor pool, and socioeconomic position rides along with demographics in the brief's topic descriptions.

A useful distinction for the report: the *human* decision is which variables are **eligible** to enter the model; the *algorithm* decides how **useful** each eligible variable is. Tree-based models in particular perform internal feature weighting and may effectively ignore weak predictors - that is expected, and comparing what the algorithms found useful (permutation importance) against the regression's odds ratios is precisely our interpretation section.

## 2. How our nineteen predictors were derived: a five-step funnel

**Step 1 - The brief defines the domains.** We did not invent predictor categories. The brief's question template names them: demographics, health behaviours, healthcare access, chronic conditions, disability (plus socioeconomic position in the topic descriptions). Our task was to populate prescribed domains, not to search an open space.

**Step 2 - Availability constrains the pool.** A predictor must exist, be usable, and be understood in the 2024 file. The course-supplied Topic C extract has 37 columns; removing the outcome variables, the state identifier, and verified redundancies leaves a realistic candidate pool of roughly two dozen variables, to which we added the caregiver fields by our own extraction from the raw file. Availability checking came first because the brief lists "choosing an outcome without checking whether it is available" as its number-one common mistake, and the same discipline applies to predictors - the clearest illustration being Topic B's suggested marijuana outcome, which simply does not exist in the supplied extracts because it lives in state-optional modules.

**Step 3 - Mechanism and literature select within the pool.** Every retained predictor has an articulable reason - a mechanism - to expect a relationship with missing check-ups, and where possible a literature anchor:

| Predictor(s) | Domain | Mechanism for missing/attending a check-up | Anchor |
|---|---|---|---|
| CAREGIV1 | Focal exposure | The group's research question: does caregiving connect a person to the health system (attending clinics with the care recipient) or crowd out their own care? | Schulz & Beach 1999; Bevans & Sternberg 2012; Edwards et al. 2020 |
| PRIMINS2, PERSDOC3, MEDCOST1 | Healthcare access | Can you afford care, and do you have a usual source of it? These are the levers policy can pull; verified bivariate gap-rate spreads of roughly 50, 50 and 22 percentage points respectively in the full file | Clark et al. 2021 (access-equity framing) |
| GENHLTH, MENTHLTH | Health status | Sicker people are *scheduled* into more contact (an expected reverse gradient worth interpreting); mental distress impedes help-seeking - and is elevated among caregivers, making MENTHLTH an essential adjustment for our focal comparison | Edwards et al. 2020 |
| DIABETE4, _MICHD | Chronic conditions | Diagnosed disease anchors respondents into recall and follow-up systems; their absence combined with other risk factors identifies the "healthy-feeling avoider" our outreach story targets | - |
| DIFFWALK, DECIDE, DIFFALON | Disability | Three distinct miss-a-visit mechanisms: mobility (getting there), cognition (remembering), independence (needing help to go) | Cree et al. 2020 |
| _SMOKER3, _TOTINDA | Health behaviours | Behaviour clustering: preventive-care avoidance travels with other health-risk behaviours | Jiang & Hesser 2006 |
| SEXVAR, _AGEG5YR, _RACEGR3, _EDUCAG, _INCOMG1, EMPLOY1 | Demographics / SES | The equity axis - and, for the caregiver question specifically, the **confounder set**: caregivers in the module states skew female (60.0%) and older, exactly the profile with higher baseline attendance, so age and sex must be adjusted for or the caregiver comparison is uninterpretable | Clark et al. 2021 |

**Step 4 - Explicit exclusion rules prune the pool.** What is left out, and why, is as much a part of variable selection as what is kept:

| Excluded variable | Rule applied |
|---|---|
| PHYSHLTH | Near-collinear with GENHLTH: two measures of one construct add instability, not information |
| _HLTHPL2 | Verified redundancy: 25,405 of the 25,406 no-coverage respondents are exactly the PRIMINS2 = 88 group; keeping both double-counts insurance |
| CRGVHRS2, CRGVREL5, CRGVLNG2 | Leakage: asked only of caregivers, so their presence/absence perfectly reconstructs the focal predictor; quarantined to the caregiver descriptive profile |
| _URBSTAT | The 16-state module footprint already constrains geographic composition; residual urban/rural variation is better acknowledged as a limitation than modelled |
| MARITAL, CHCCOPD3, CHCKDNY2 | Weakest marginal mechanism among their domains; parsimony under the cap |
| POORHLTH | 41.4% structurally blank from skip logic - unusable in any role |

**Step 5 - The cap forces parsimony.** The brief's maximum of 20 predictors exists to prevent kitchen-sink models that no group can interpret. We selected nineteen - deliberately one under the cap, so that a late, well-argued addition would not force an unplanned cut.

## 3. What we deliberately did NOT do

We did **not** select predictors by mining the data. There is no stepwise selection, no "include everything and keep what is significant", no univariate screening against the outcome, and no peeking at test-set performance to decide what stays. The predictor list was fixed *a priori* - from the brief's domains, variable availability, mechanism, and literature - before any model is fit. This choice has two justifications. Substantively, it is the masters-level judgement the brief demands ("justifying your variable choices"): a defensible reason per variable, stated before the results are known. Methodologically, it is leakage hygiene: data-driven selection performed on the full dataset lets information from the (future) test partition influence the model specification, quietly inflating apparent performance.

The one data-driven input we permitted ourselves was **feasibility checking** - measuring each candidate's missingness and verifying suspected redundancies (such as _HLTHPL2 against PRIMINS2). These checks use only the predictors' own distributions and their relationships to each other, never their relationship to the outcome, and are therefore selection on data *quality*, not on predictive signal.

## 4. Where the choices surface downstream

- The **variable dictionary** (methodology document, Section 4) records each predictor's original coding, recode, and role - the brief's required format.
- The **baseline logistic regression** reports an adjusted odds ratio per predictor; the caregiver coefficient is the headline estimand.
- The **tree and forest** are free to ignore, split on, or interact any eligible predictor; permutation importance is cross-checked against the odds ratios in the interpretation section, and divergence between the two (a variable strong in the forest but null in the regression) is read as evidence of non-linearity or interaction, not contradiction.
- The **limitations** section owns the exclusions' consequences: no geographic adjustment (by design), no caregiving-intensity dose-response in the models (quarantined), and a predictor set bounded by what BRFSS measures - unmeasured factors such as appointment availability, transport access, or health literacy remain candidates for the "what additional data would help" discussion the brief requires.
