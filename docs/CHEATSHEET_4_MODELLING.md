# Cheat Sheet 4: ML Analyses & Predictive Modelling Plan

Grounded in the scope rules (1 baseline + >=2 ML, <=20 predictors) and the actual data (recommended outcome CHECKUP1, 18.2% event rate, 457,670 rows).

## The model lineup (matches the rubric exactly)

| # | Model | Role | Notes |
|---|---|---|---|
| 1 | **Logistic regression** | Mandatory baseline | Report odds ratios with 95% CIs. Interpretability anchor: "uninsured respondents (PRIMINS2=88) have X times the odds of no check-up." |
| 2 | **Decision tree** | ML model 1 | Shallow (depth 4-5), PLOT it. The brief names it for interpretability; the tree diagram is presentation gold for non-technical audiences. |
| 3 | **Random forest or gradient boosting** | ML model 2 - performance ceiling | With ~450k rows use sklearn's `HistGradientBoostingClassifier` (fast, handles NaN natively) or `RandomForestClassifier(n_jobs=-1)`. Stop there - neural nets are explicitly discouraged. |

## Methodology choices that score marks

1. **Stratified 80/20 train/test split**, fixed `random_state`, split BEFORE fitting anything (leakage is the classic mistake graders look for). `train_test_split(X, y, test_size=0.2, stratify=y, random_state=42)`.
2. **Encoding**: one-hot the nominal vars (PRIMINS2, EMPLOY1, _RACEGR3, MARITAL); keep ordinal vars as ordered integers (_EDUCAG, _INCOMG1, _AGEG5YR). Binary flags stay 0/1 after the 7/9 -> missing recode.
3. **Class imbalance**: 18% is mild - do NOT reach for SMOTE. Set `class_weight='balanced'` in logistic/tree and justify in one sentence. Simple and defensible.
4. **Evaluation - one table, all models**: AUC-ROC (primary), sensitivity, specificity, precision, F1 at a stated threshold, plus a **calibration curve** for the best model. Rationale to state: in a screening/outreach framing, missing a person with a care gap (false negative) is the costly error, so sensitivity matters more than raw accuracy.
5. **Interpretation crosswalk** (where the 4% interpretation criterion is won): put logistic odds ratios side-by-side with RF/GBM **permutation importance** and check they tell a consistent story (insurance, income, personal doctor, cost barrier should dominate). A variable that ranks high in ML but is null in logistic hints at non-linearity or interaction - one paragraph on that reads as sophistication. SHAP summary plot = optional stretch goal, not required.

## What NOT to do

- **No survey weights** - state it as a limitation ("results describe our analytic sample, not the US population"), not a choice you forgot.
- **No heavy hyperparameter tuning** - defaults or a small grid + one sentence. The brief's own language: "emphasis is on understanding, not maximising accuracy."
- **No causal claims** - "predicts" / "is associated with", never "causes".
- **No neural networks** - explicitly not recommended without prior experience.
- **No test-set peeking** - the test set is touched once, at the end, for the final table.

## Skeleton code (the whole pipeline in ~30 lines)

```python
import pandas as pd, numpy as np
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LogisticRegression
from sklearn.tree import DecisionTreeClassifier, plot_tree
from sklearn.ensemble import HistGradientBoostingClassifier
from sklearn.metrics import roc_auc_score, classification_report
from sklearn.inspection import permutation_importance

df = pd.read_csv('brfss2024_topic_c.csv')
# ... recodes from CHEATSHEET_2 (7/9->NaN, 88 handling, outcome = CHECKUP1 != 1 in past yr)

X = pd.get_dummies(df[predictors], columns=nominal_vars, drop_first=True)
y = df['no_checkup']
mask = y.notna() & X.notna().all(axis=1)          # or let HistGB handle NaN
X_tr, X_te, y_tr, y_te = train_test_split(X[mask], y[mask], test_size=0.2,
                                          stratify=y[mask], random_state=42)

models = {
    'logistic': LogisticRegression(max_iter=2000, class_weight='balanced'),
    'tree':     DecisionTreeClassifier(max_depth=5, class_weight='balanced', random_state=42),
    'hgb':      HistGradientBoostingClassifier(random_state=42),
}
for name, m in models.items():
    m.fit(X_tr, y_tr)
    print(name, 'AUC:', roc_auc_score(y_te, m.predict_proba(X_te)[:, 1]).round(3))

# odds ratios (baseline) vs permutation importance (best ML)
ors = pd.Series(np.exp(models['logistic'].coef_[0]), index=X.columns).sort_values()
imp = permutation_importance(models['hgb'], X_te, y_te, n_repeats=5, random_state=42)
```

## Division of the modelling work

- Modelling lead: pipeline + models + metrics table
- Data-prep lead: hands over ONE canonical recoded DataFrame (never let two members recode independently)
- Interpretation lead: odds-ratio table, importance crosswalk, limitation paragraphs
- Everyone: agree the threshold and the metric story BEFORE the presentation is written
