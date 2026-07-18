# Cheat Sheet 1: The Project (mechanics, deliverables, grading)

One-line summary: groups of ~6 build an end-to-end predictive-modelling pipeline on the 2024 BRFSS survey - one outcome, max 20 predictors, 1 logistic baseline + 2 or more ML models - and are graded far more on data handling and interpretation than on model accuracy.

## Corrected timeline (all dates shifted +1 week; Session 1 = Sat 18 Jul)

| Date | Deliverable | Weight |
|---|---|---|
| 18 Jul (today) | Session 1: groups form, topic selection starts | participation 15% runs Weeks 1-4 |
| **25 Jul** | **Proposal (2 pages max excl. annexes)** | **10%** |
| 1 Aug | Quiz 1 (landscape, data ecosystem, readings) | 20% |
| 15 Aug | Quiz 2 + final presentation (20 min + 10 min Q&A, 8-10 slides) | 25% + part of 20% |
| 21 Aug | Final report (2,500-3,500 words) + notebook + slides + SPOT peer eval | 20% + 10% |

## Hard scope rules (do not violate any)

1. ONE primary outcome (optional: one secondary, or a justified composite)
2. Max 20 predictors
3. Exactly 1 baseline regression (logistic unless outcome is continuous)
4. Minimum 2 ML models on top (recommended: decision tree + random forest or gradient boosting; skip neural nets)
5. 2024 data only
6. Survey weights NOT used - and therefore NEVER claim "US adults..."; always say "in our analytic sample..."

## Proposal rubric (10% total - due in ONE WEEK)

| Criterion | Weight | Win condition |
|---|---|---|
| Problem framing & motivation | 2% | Why it matters + why BRFSS fits |
| **Data feasibility & variable selection** | **3% (biggest)** | Outcome + predictors named with 2024 codebook awareness (codings, missingness, recodes) |
| Literature / background | 2% | 2-3 anchors (provided: Jiang 2006, Parekh & Fahim 2021, Clark 2021) |
| Analysis plan | 2% | cleaning -> missing handling -> descriptives -> baseline -> 2 ML -> evaluation -> interpretation |
| Team roles | 1% | 5 named roles with owners |

## Final rubric (20%)

- Data prep & variable handling 6% (LARGEST single criterion - recode table + missingness table + data-flow diagram)
- Descriptive analysis 4% (Table 1, outcome distribution, 2+ charts)
- Model development/evaluation/interpretation 4% (baseline vs ML comparison, metrics, cautious language)
- Reporting quality 4% (organised, limitations acknowledged)
- Completeness of collaterals 2% (report + notebook + slides + peer eval all consistent; do NOT submit the raw dataset)

## What the prof supplies (v3 Project Guidance - all already in `data/`)

- `LLCP2024ASC.zip` -> 922 MB fixed-width ASCII raw file (source of truth)
- `USCODE24_LLCP_082125.HTML` codebook + `brfss2024_variable_dictionary.csv` (parsed: variable, label, column positions)
- `BRFSS_Extraction.ipynb` - fixed-width extraction logic; reusable to pull EXTRA fields
- `brfss2024_topic_a.csv` (29 cols) / `_b.csv` (33) / `_c.csv` (37) - all 457,670 respondents, ready to model

## Grading psychology (read the rubric like a grader)

- 6% for recoding/missingness vs 4% for models: a boring model with immaculate data handling beats a fancy model with sloppy recodes.
- "Interpretation should be clear, cautious, and connected to the problem statement" - hedge properly: association not causation, sample not population.
- Peer eval (10%) = SPOT: Supportive, Proactive, Openness, Thorough. Translation: answer the group chat fast, hit internal deadlines, volunteer for the unglamorous data-prep role.
