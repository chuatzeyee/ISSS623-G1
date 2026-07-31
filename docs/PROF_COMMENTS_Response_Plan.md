# Responding to the Prof's Proposal Comments - Comprehensive Fix Plan

**ISSS623 Group 1 · Sources: `Group 1_Comments_1.docx` (overall feedback) + `Group 1- Comments 2.docx` (9 inline comments by Sean Lam) on `Project Proposal (For Submission).docx`**

The overall verdict was positive - "a strong and feasible proposal... particularly strong in data [feasibility]" - so these comments are the roadmap for the FINAL report, not a rejection. Every comment below is quoted, interpreted, and answered with a concrete fix, an owner, and where it lands in the final report.

---

## Part A - The 9 inline comments (Comments 2)

### A1. Problem statement lacks precise Outcomes / Predictors / Time frame
> **Anchored on:** "...identify predictors and conduct a predictive analysis on preventive care gaps among caregivers..."
> **Prof:** "Problem statement is not clear on the Outcomes, predictor variables and prediction time frame. Can be improved with more precise definitions as covered in Lecture 2. These can be hypothesis or Research Questions."

**What he means:** He wants the Lecture 2 five-element framing checklist applied verbatim - target population, exposure, outcome, predictors, time frame - ideally crystallised into numbered research questions or hypotheses. Our prose buried these.

**The fix - open the final report with a formal framing box:**
- **Target population:** US adults in the 16 states that fielded the 2024 BRFSS Caregiver module.
- **Exposure of interest:** informal caregiver status (CAREGIV1: provided regular care to a family member/friend in the past 30 days).
- **Primary outcome:** no routine check-up within the past year (CHECKUP1 recoded binary; see A8 for the justification he also asked for).
- **Secondary outcome:** cost-related barrier to seeing a doctor in the past 12 months (MEDCOST1).
- **Predictors:** 19 variables across demographics, socioeconomic position, health status, chronic conditions, disability and healthcare access (full dictionary in annex).
- **Time frame:** cross-sectional prediction - all predictors and outcomes refer to the 2024 survey reference periods (check-up: past 12 months; caregiving: past 30 days). State explicitly there is NO longitudinal prediction horizon; the model estimates current-state association, not future risk.
- **Research questions (numbered):**
  - RQ1: Among adults in the module-state analytic sample, can demographic, socioeconomic, health, disability and access variables predict not having attended a routine check-up in the past year?
  - RQ2: Does caregiver status remain associated with the check-up gap after adjustment for these characteristics (connection vs neglect hypotheses - direction not presumed)?
  - RQ3: Within caregivers, which characteristics - including caregiving relationship, intensity and duration - identify subgroups with higher gap likelihood?

**Owner:** Project lead (Hui Ying) drafts; all review. **Lands in:** Report §1, first page.

### A2. Data completeness/missingness for CAREGIV1
> **Anchored on:** "The key variable used to inform the predictive analysis is CAREGIV1..."
> **Prof:** "Good idea. Do check data completeness, missingness, etc."

**What he means:** Approval plus an instruction: show the completeness numbers for the caregiver variable itself, not just the pipeline totals.

**The fix:** One short subsection: of 96,287 asked, 21,137 yes / 74,592 no / 241 don't-know (7) / 215 expects-within-2-years (8) / 102 refused (9) → item-level invalidity 0.58%. Distinguish structural non-participation (79% of the full file never asked - a sampling-frame issue) from item missingness (0.58% - negligible). We already have these numbers computed in the prep notebook; surface them.

**Owner:** Data prep (Tze Yee). **Lands in:** Report §3 data preparation.

### A3. More detailed data validation in the final report
> **Anchored on:** "...86,676 cases. This is consistent with the CDC's estimate that around 1 in 5 US adults are caregivers..."
> **Prof:** "Good that you check as part of data validation. Final report can be more detailed in data validation."

**What he means:** The CDC 1-in-5 check earned credit; he wants a *systematic* validation section, not one sanity check.

**The fix - a "Data validation" subsection with at least 4 checks:**
1. Caregiver prevalence 22.2% vs CDC ~1-in-5 (external benchmark).
2. Outcome event rate 18.2% in the module sample vs 18.2% in the full national file (restriction did not distort the outcome).
3. CAREGIV1 value distribution matches the codebook's legal code set exactly (extraction correctness; also cite the row-alignment `_STATE` assertion before the merge).
4. Demographic profile of caregivers (60% female, older skew) consistent with published caregiver profiles (Edwards et al. 2020, MMWR - which used this same BRFSS module).
Optionally 5: per-state event rates within the 16 states to show no single state drives the outcome (feeds A4 too).

**Owner:** Data prep (Tze Yee) + Descriptive (Guang Hua). **Lands in:** Report §3.

### A4. Geographic subgroup analysis + internal/external validation
> **Anchored on:** "...19,263 caregiver cases with around 2,890 check-up gaps (15%)."
> **Prof:** "Good that you checked the proportion of positive cases here. Anyway to do some subgroup analysis based on geographical regions and compare? Consider internal/external validation?"

**What he means:** Two distinct asks. (1) Exploit the 16-state structure descriptively - do gap rates differ by state/region? (2) Say explicitly what your validation design is, using the internal/external vocabulary.

**The fix:**
- **Geographic subgroup analysis (descriptive):** table/map of gap rate by state (we have _STATE for every row); optionally group into Census regions (South: KY LA MS NC TX VA · Midwest: IL SD WI · Northeast: CT NH · West: NV OR UT WY · DE South). Report caregiver vs non-caregiver gaps within regions to show the association is not one region's artefact.
- **Internal validation:** our stratified 70/30 held-out test set + cross-validation on the training partition = internal validation; label it as such in the report.
- **External-flavoured validation (the clever move):** LEAVE-ONE-STATE-OUT or region-based split - e.g., train on 12 states, test on the 4 held-out states - to approximate external/geographic transportability. Frame honestly: true external validation would need a different year or dataset (out of scope), so we offer geographic hold-out as a transportability probe. This directly echoes the leave-site-out concept from the readings.

**Owner:** Modelling (Ziying) for splits; Descriptive (Guang Hua) for the state table. **Lands in:** Report §4 methods + §5 results.

### A5. "Good" (leakage quarantine)
> **Anchored on:** the CRGV* exclusion paragraph. **Prof:** "Good"

**Fix:** None - keep the paragraph essentially verbatim in the final report. It scored.

### A6. Calibration plot / Brier score
> **Anchored on:** "...greater emphasis will be placed on sensitivity and AUPRC rather than accuracy alone."
> **Prof:** "Consider calibration plot or Brier score."

**What he means:** Discrimination metrics (AUROC/AUPRC) say who ranks higher; calibration says whether predicted probabilities are honest (a model saying "20% risk" should be right ~20% of the time). For policy targeting, calibration matters.

**The fix:** Add to the evaluation battery, on the held-out test set: (1) **calibration plot** (reliability curve, 10 probability bins, predicted vs observed) for the best model AND the logistic baseline; (2) **Brier score** for all models in the comparison table (note it rewards both discrimination and calibration; lower is better); (3) one interpretation sentence - e.g., tree ensembles often need recalibration; if miscalibrated, mention Platt scaling/isotonic as the remedy (doing it is optional; naming it shows understanding).
sklearn: `calibration_curve`, `brier_score_loss` - trivial additions to the notebook.

**Owner:** Modelling (Ziying). **Lands in:** §4 evaluation + comparison table + one figure.

### A7. Class imbalance - check and justify (probably) NOT treating it
> **Anchored on:** the MEDCOST1 row of the dictionary. **Prof:** "Check the proportion of positive cases in the outcome variables. Is there a need to do resampling in the case of imbalanced data? If imbalance is not too bad, there is no need for any treatment for imbalance."

**What he means:** He is TELLING us the answer: our imbalance is mild, so do not reach for SMOTE - but show the numbers and make the decision explicitly.

**The fix:** One paragraph: primary outcome 18.2% positive (86,676 cases; ~15,800 events), secondary MEDCOST1 ~9.5% positive. Both are mild-to-moderate; we therefore use NO resampling, and handle skew via `class_weight='balanced'` where beneficial plus threshold-aware metrics (sensitivity, AUPRC, calibration). Cite the decision rule: resampling is warranted for severe imbalance (~<5% or worse); ours is not. This converts his rhetorical question into a documented judgement call.

**Owner:** Modelling (Ziying). **Lands in:** §4 methods.

### A8. "Why is this also a predictor here?" - MEDCOST1's dual role ⚠ most pointed comment
> **Anchored on:** CAREGIV1 dictionary row (visually adjacent to MEDCOST1's "Secondary outcome & predictor" label).
> **Prof:** "Why is this also a predictor here?"

**What he means:** The dictionary's terse "Secondary outcome & predictor" label makes it look like one model uses MEDCOST1 on both sides - a leakage/confusion red flag. Our design is actually clean (predictor in the primary model; target of a SEPARATE secondary model from whose predictor set it is removed), but the proposal never said so plainly in the table.

**The fix (do all three):**
1. **Answer the question head-on in the report:** "MEDCOST1 plays one role per model, never both: in the primary (CHECKUP1) model it is a predictor capturing affordability - a core access mechanism; in the separate secondary analysis it becomes the target and is removed from that model's predictor set. A variable never appears on both sides of the same model."
2. **Fix the dictionary label:** split into two rows or footnote it - "Predictor (primary model) / Outcome (secondary model only)".
3. **Consider descoping:** honestly evaluate whether the secondary outcome earns its complexity. Comments 1 says "use fewer models but evaluate them well" - if time is tight, demote the MEDCOST1 analysis to a brief annex or drop it, and say the primary analysis is the focus. Decide as a group BEFORE modelling begins.

**Owner:** Data prep (Tze Yee) for the dictionary fix; group decision on descoping. **Lands in:** §3 dictionary + §4 methods.

### A9. Missing-value strategy - justify, and cite BRFSS documentation
> **Anchored on:** "1.3 Missingness Table & Retention Rule"
> **Prof:** "Do think of possible missing value handling strategies and provide justifications for why those were used. Some of the missingness may have been reported in BRFSS data documentation. Reference them if needed."

**What he means:** The table shows WHAT we did, not WHY complete-case beats imputation for us, and it cites no BRFSS documentation.

**The fix - a missing-data strategy subsection:**
- **State the decision rule:** item-invalidity <5% → complete-case exclusion (bias negligible at these magnitudes; simple and reproducible); ≥5% → retain as explicit "not reported" category (deleting 16.7% income non-responders would both shrink and bias the sample, since income refusal correlates with socioeconomic position - the construct itself).
- **Name the road not taken and why:** single imputation (mean/median) distorts variance; multiple imputation (MICE) is defensible but adds complexity disproportionate to <2% item missingness on most variables - consistent with the brief's own note that a good project needs intentional handling, not sophisticated methods.
- **Reference BRFSS documentation:** cite the 2024 BRFSS Codebook (value conventions 7/9/77/99, BLANK = not asked), the BRFSS Overview/Data Quality Report (response rates; item non-response patterns - income non-response is a known BRFSS phenomenon), and the module-fielding list (16 states). Also reiterate the structural-vs-item missingness distinction.
- **Add "% retained" line:** 92.0% within-module retention → 86,676.

**Owner:** Data prep (Tze Yee). **Lands in:** §3 + expanded annex missingness table (add a "Justification" column).

---

## Part B - The overall feedback (Comments 1)

### B1. "Sharpen the problem framing" → same fix as A1 (the framing box + RQs).

### B2. "There seems to be a secondary outcome as 'Cost'. Consider this in your definition of the problem statement."
The secondary outcome appeared only in the annex table, never in the problem statement. Fix: if we KEEP it, one explicit sentence in §1 ("As a secondary question, we examine whether the same characteristics predict a cost-related barrier to care (MEDCOST1)") + the A8 role clarification. If we DROP it (see A8.3), remove it everywhere consistently. Either way the problem statement and annex must agree.

### B3. "What is the reason for this recode for CHECKUP1? 1, 0 = 0 (attended); 2/3/4/8 = 1 (gap) - Positive cases is 1 or more years? Why?"
Two things to fix:
- **A real typo he caught:** our table says "1, 0 = 0" - there is no code 0 in CHECKUP1. It should read "1 → 0 (attended within past year)". Correct the table.
- **Justify the cut-point:** positive = last check-up more than 1 year ago or never (codes 2/3/4/8). Rationale to write: (i) annual periodicity is the interval used by BRFSS's own question structure (code 1 = "within past year") and by preventive-visit measures in the caregiver literature we cite (Tingey et al. 2020 used the same instrument); (ii) an annual routine check-up operationalises "engaged with preventive care" in guidance (e.g., Medicare Annual Wellness Visit; USPSTF screening cadences anchored to periodic visits); (iii) sensitivity framing: codes 2/3/4/8 form an ordinal gradient - we can report a secondary threshold (>2 years, codes 3/4/8) as a robustness check to show findings are not artefacts of the 1-year cut. Also justify "never" (8) = gap: never attending is the most severe form of the gap, not missing data.

### B4. "Justify why the outcome can represent 'preventive care gap' but also highlight its limitations."
Write both halves:
- **Justification:** routine check-ups are the delivery vehicle for preventive services (BP/diabetes/cholesterol screening, vaccination review, counselling); the measure is asked of ALL respondents (unlike age/sex-gated screening items); it is nearly complete (1.1% invalid).
- **Limitations paragraph:** (i) self-reported, recall bias; (ii) a check-up is not the totality of preventive care - dental visits, vaccinations and screenings are distinct (we descriptively report _DENVST3 as a companion indicator); (iii) annual frequency is a convention, and some guidance questions fixed-interval general check-ups for low-risk adults - so "gap" means deviation from the annual-visit norm, not clinical harm per se; (iv) no information on visit QUALITY or content.

### B5. "Final report should show the variables and descriptions clearly in a table." → We have the dictionary; upgrade columns to: Variable · Concept/description (plain English) · Role · Original coding · Recode · Missingness % · Decision. One table, everything visible.

### B6. "Include a data flow diagram from the full BRFSS sample (Ref Lecture 4; Lim SL et al. 2024 JAHA - NULL-EASE validation)."
Draw a CONSORT-style cascade figure exactly like clinical prediction papers:
457,670 (full 2024 BRFSS) → 96,287 asked Caregiver module (16 states) → 94,670 valid caregiver status + outcome → 86,676 complete cases (main analytic sample) → branch: 19,263 caregivers-only subset; annotate each arrow with the exclusion reason and n lost. He even gave us the exemplar paper to imitate (Lim et al. 2024, JAHA 13(16):e034133) - cite it as the format reference. Put the figure in the main text, not the annex.

### B7. "Use fewer models but evaluate them well."
Direct instruction. Fix: cut the proposal's XGBoost + soft-voting-ensemble mentions. Final lineup: **logistic regression (baseline) + decision tree + random forest** - exactly the brief's recommended set - each evaluated thoroughly (5 metrics + calibration + Brier + the geographic hold-out). Depth over breadth is being graded.

### B8. "Interpret results cautiously, avoiding causal claims and consider internal/external validations based on what your problem statement is."
Standing discipline: "predicts/is associated with", never "causes"; carry the required unweighted-sample limitation statement plus the 16-state caveat; tie the internal (70/30 + CV) and geographic hold-out validation back to RQ1-RQ3 explicitly (validation design should follow from the problem statement - his exact phrasing).

---

## Part C - Priority checklist (what to actually do, in order)

| # | Action | Comment(s) | Owner | Effort |
|---|---|---|---|---|
| 1 | Framing box + 3 numbered RQs at the top of the report | A1, B1, B2 | Hui Ying | Low |
| 2 | Fix CHECKUP1 recode typo ("1, 0 = 0") + cut-point justification + robustness threshold | B3 | Tze Yee | Low |
| 3 | Outcome justification + limitations paragraph | B4 | Tamara | Low |
| 4 | MEDCOST1 role clarification in dictionary + report; group decision keep-vs-drop secondary outcome | A8, B2, B7 | All (decide), Tze Yee (edit) | Low |
| 5 | Missing-data strategy subsection with justifications + BRFSS documentation citations + "Justification" column in annex table | A9, A2 | Tze Yee | Medium |
| 6 | Data validation subsection (4-5 checks) | A3 | Tze Yee + Guang Hua | Medium |
| 7 | Data flow diagram (CONSORT-style, Lim 2024 format) in main text | B6 | Guang Hua | Low |
| 8 | Drop XGBoost + ensemble; lineup = LR + DT + RF, evaluated deeply | B7 | Ziying | Low (deletion!) |
| 9 | Add calibration plot + Brier score to evaluation | A6 | Ziying | Low |
| 10 | Imbalance paragraph: report 18.2%/9.5%, justify NO resampling | A7 | Ziying | Low |
| 11 | Geographic subgroup table + leave-states-out transportability validation | A4 | Ziying + Guang Hua | Medium-High |
| 12 | Cautious-language pass + limitation statements + validation-to-RQ mapping | B8 | Tamara | Low |

**Two group decisions needed before analysis starts:** (a) keep or drop the MEDCOST1 secondary outcome (recommend: keep but compress to a short annex - it answers B2 while respecting B7); (b) region grouping scheme for the geographic analysis (recommend: Census regions, since 16 states give thin single-state cells).

**The meta-message across all 11 comments:** the prof liked the data work and wants the *scientific packaging* to match - precise framing, justified decisions, honest validation, fewer-but-deeper models. Nothing requires re-doing the data pipeline; everything is writing, one figure, and three modelling additions (calibration, imbalance paragraph, geographic hold-out).
