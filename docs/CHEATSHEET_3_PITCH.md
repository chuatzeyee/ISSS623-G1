# Cheat Sheet 3: What to Produce & What to Pitch to Your Group

## The 30-second pitch (say this when groups form today)

> "I've already downloaded the dataset, codebook and the prof's topic extracts, and profiled them. I suggest **Topic C (preventive care gap, outcome CHECKUP1)** - 18.2% event rate, cleanest data, and the topic C file is a superset that lets us pivot to Topic A without re-extraction. I can walk us through the variables tonight, and I propose we lock outcome + predictors by Tuesday so the proposal (due 25 Jul, 10%) is just writing, not deciding."

Instant credibility: you arrive with data profiled, a defensible recommendation, and a schedule.

## Recommended pick, with fallback

**Primary recommendation - Topic C: "Can respondent characteristics predict having no routine check-up in the past year?"**
- Outcome CHECKUP1 != 1 within past year: 18.2% event rate (measured), nearly no missing (7/9 codes ~1%)
- Rich verified predictors: PRIMINS2 (insurance type incl. 88=uninsured), MEDCOST1, PERSDOC3 (personal doctor), _HLTHPL2, disability battery, SES set (_EDUCAG, _INCOMG1, EMPLOY1), _URBSTAT (urban/rural), GENHLTH
- Story writes itself: access/equity, "move care left", Healthier SG parallel (Singapore relevance = better motivation section)
- Literature anchor: Clark et al. 2021 (+ CDC preventive-services literature)

**Fallback - Topic A: poor/fair general health (GENHLTH >= 4), 19.7% event rate.** Strongest literature anchor (Jiang 2006, CDC Healthy Days). Avoid POORHLTH as outcome (41.4% skip-logic blanks).

**Avoid as primary - Topic B marijuana outcomes**: the variables are not in the supplied extract (state-optional modules). If the group wants mental health, use FMD (MENTHLTH >= 14, 13.9%) which IS fully available.

## Division of labour to propose (5 roles, ~6 people)

| Role | Who | Deliverable | When |
|---|---|---|---|
| Project/proposal lead | strongest writer | 2-page proposal | draft 22 Jul, submit 25 Jul |
| Data prep lead (+1 helper) | you? (you have the head start) | recode notebook, variable dictionary, missingness table, data-flow diagram | 27 Jul |
| Descriptive lead | | Table 1, outcome distribution, 2+ charts | 1 Aug |
| Modelling lead (+1 helper) | | baseline logistic + DT + RF/GBM, metrics table | 8 Aug |
| Interpretation/report lead | | 2,500-3,500-word report + slides | 15-21 Aug |

Claiming data-prep is strategic: it is the single biggest rubric criterion (6%), it front-loads your effort before quizzes, and everyone downstream depends on you (visibility for SPOT peer eval).

## Proposal skeleton (2 pages - fill in, don't reinvent)

1. **Project question** (1 sentence): "Among respondents in the 2024 BRFSS analytic sample, can demographic, socioeconomic, healthcare-access, health-status and disability variables predict not having a routine check-up in the past year?"
2. **Public health motivation** (short para): preventive-care gaps -> late diagnosis -> higher acute cost; access equity; mirror to Singapore's Healthier SG enrolment logic.
3. **Literature context** (3 sentences): Clark 2021 (BRFSS ML precedent), Jiang 2006 (HRQOL associations), Parekh & Fahim 2021 (BRFSS ML methodology).
4. **Data & variables** (the 3% criterion - table): outcome CHECKUP1 recode (1 = check-up >1 yr ago/never; 7/9 -> missing), ~14 predictors from the verified topic C list with original coding + planned recode + expected missingness (quote: PRIMINS2 77/99 ~4%, disability items 3-5% blank, _BMI5CAT 9.4% if used).
5. **Planned analysis**: train/test split -> descriptives -> baseline logistic (odds ratios) -> decision tree + random forest -> AUC, sensitivity/specificity, calibration -> compare feature importance vs coefficients -> cautious interpretation, no population claims (no survey weights).
6. **Team roles**: table above.

## First-week production checklist (before/right after groups form)

- [x] Dataset + codebook + topic CSVs downloaded (`data/`)
- [x] Missingness + prevalence profiling done (numbers in Cheat Sheet 2)
- [ ] Set up group GitHub repo (add .gitignore for *.csv, *.ASC, *.zip) + shared Drive for data
- [ ] Share Cheat Sheets 1-3 with the group after the first meeting
- [ ] Lock topic + outcome by ~21 Jul; predictors by 22 Jul; proposal draft 23-24 Jul; submit 25 Jul
- [ ] Everyone runs Lecture_1.ipynb + BRFSS_Extraction.ipynb in Colab once (tool fluency)

## Questions to settle in the first group meeting (bring answers, not questions)

1. Topic: C (recommended) or A? - show the prevalence/missingness numbers
2. Secondary outcome: add MEDCOST1 (cost barrier) as secondary? (cheap, same file)
3. Tooling: Colab + GitHub (course default) - one canonical recode notebook everyone imports
4. Meeting cadence: 2x/week short check-ins beat 1x/week marathons (SPOT optics too)
5. Who writes what section of the proposal TONIGHT
