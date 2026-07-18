# Cheat Sheet 2: The Data (BRFSS 2024 - real numbers from the actual files)

Everything below was computed from the files in `data/` on 18 Jul 2026 - quote these numbers in the proposal and you will sound like the group already did the work (because we did).

## The dataset in one paragraph

BRFSS (Behavioral Risk Factor Surveillance System) is the CDC's annual telephone survey of US adults: health behaviours, chronic conditions, healthcare access, preventive services. The 2024 combined landline+cell file has **457,670 respondents**. Raw format is fixed-width ASCII (922 MB, column positions defined in the codebook); the prof already extracted per-topic CSVs so you never need to touch the raw file unless you want extra variables.

## Universal BRFSS coding rules (memorize - this is the 6% criterion)

| Code | Meaning | Action |
|---|---|---|
| 7 / 77 | Don't know / Not sure | -> missing (never a real value) |
| 9 / 99 | Refused | -> missing |
| 88 | "None" (in day-count variables like PHYSHLTH/MENTHLTH) | -> recode to **0**, NOT missing |
| BLANK / NaN | Not asked (skip logic) or missing | -> missing; explain skip logic if large |
| 8 (in CHECKUP1-type items) | Never | keep as valid category |
| `_` prefix (e.g. _AGEG5YR) | CDC calculated variable, partially pre-cleaned | prefer these for demographics |

Trap: in PHYSHLTH/MENTHLTH the valid range is 1-30 days + 88=none. If you forget 88->0 your "mean unhealthy days" explodes. If you treat 7/9 as values in GENHLTH your ordinal scale breaks.

## Verified per-topic files (headers read from the actual CSVs)

### Topic A - High Health Burden (29 cols)
- Outcomes available: GENHLTH (1 Excellent..5 Poor), PHYSHLTH, MENTHLTH, POORHLTH
- Predictors: PERSDOC3, MEDCOST1, CHECKUP1 · chronic: CVDSTRK3, CHCCOPD3, ADDEPEV3, CHCKDNY2, DIABETE4, _MICHD, _CASTHM1, _DRDXAR2 · demo/SES: SEXVAR, MARITAL, EMPLOY1, _RACEGR3, **_AGE80**, _EDUCAG, _INCOMG1 · behaviour: _TOTINDA, _SMOKER3, _RFBING6 · _HLTHPL2, _URBSTAT, _BMI5CAT
- **Measured**: poor/fair health prevalence **19.7%** (nice balanced-ish binary); FMD (MENTHLTH>=14) **13.9%**
- Missingness: POORHLTH **41.4% blank** (skip logic - only asked if PHYSHLTH or MENTHLTH > 0) -> avoid as primary outcome; _BMI5CAT 9.4%; everything else <4%

### Topic B - Mental Health (33 cols)
- Outcome: MENTHLTH (FMD >=14 days = 13.9%)
- Extras vs A: disability battery DEAF, BLIND, DECIDE, DIFFWALK, DIFFDRES, DIFFALON (3.3-4.5% missing), _CURECI3 (e-cig), _RFDRHV9 (heavy drinking), _AGEG5YR (5-yr bands, not _AGE80)
- NOTE: marijuana variables are NOT in this file (state-optional module) - despite the topic B description mentioning them. Use FMD as outcome, or extract module fields yourself.

### Topic C - Preventive Care Gap (37 cols)
- Outcomes: CHECKUP1 (**18.2% had no routine check-up within 1 year** - excellent binary), MEDCOST1 (9.5% cost barrier), _DENVST3 (dental visit: 1=yes 311k, 2=no 141k, 9=missing - ~31% no visit)
- FLUSHOT7 is NOT in the file - extract it via the notebook if you want the flu outcome
- Insurance: **PRIMINS2** (2024 name! older papers say PRIMINSR/HLTHPLN1) - 10 insurance-type categories + 88=none (25,406 uninsured), 77/99 missing
- Superset file: contains GENHLTH/PHYSHLTH/MENTHLTH + disability battery too - topic C's file can actually serve topics A-lite analyses

## Recode recipes (copy-paste level)

```python
import pandas as pd, numpy as np
df = pd.read_csv('brfss2024_topic_a.csv')

# outcome: poor/fair health
g = df['GENHLTH'].replace({7: np.nan, 9: np.nan})
df['poor_fair'] = np.where(g.isna(), np.nan, (g >= 4).astype(float))

# day-counts: 88 -> 0, 77/99 -> missing
for c in ['PHYSHLTH', 'MENTHLTH']:
    d = df[c].replace({88: 0, 77: np.nan, 99: np.nan})
    df[c + '_days'] = d
df['fmd'] = np.where(df['MENTHLTH_days'].isna(), np.nan, (df['MENTHLTH_days'] >= 14).astype(float))

# yes/no items (1=yes 2=no): MEDCOST1, chronic flags
df['cost_barrier'] = df['MEDCOST1'].replace({2: 0, 7: np.nan, 9: np.nan})

# missingness table (required deliverable)
miss = df.isna().mean().mul(100).round(1).rename('pct_missing')
```

## Data hygiene rules for the repo

- Never commit LLCP2024.ASC (922 MB) or the topic CSVs (44-57 MB) - .gitignore them; share via Drive
- Document every exclusion: 457,670 -> (drop missing outcome) -> ... -> final analytic N, as a data-flow diagram
- Keep one canonical recode notebook the whole group imports - divergent recodes between members is the classic group-project failure
