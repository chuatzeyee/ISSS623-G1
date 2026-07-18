# ISSS623-G1 - Group Project Workspace

SMU ISSS623 Applied Data Science in Healthcare (Jul-Aug 2026). Predictive modelling on the 2024 BRFSS dataset.

## Layout

| Folder | Contents |
|---|---|
| `data/` | BRFSS 2024 dataset (zipped ASCII), codebook (HTML + parsed CSV dictionary), prof's pre-extracted topic A/B/C CSVs (457,670 respondents each), extraction notebook |
| `docs/` | Week 1 prep guide + project cheat sheets (mechanics, data profile, pitch/proposal skeleton) |
| `debate/` | Session 1 ageing-debate prep (all three roles) |

## Data quick facts

- `LLCP2024ASC.zip` extracts to `LLCP2024.ASC` (922 MB fixed-width ASCII) - **do not commit the extracted file**; it exceeds GitHub's 100 MB limit. Extract locally and parse with `data/BRFSS_Extraction.ipynb` + `brfss2024_variable_dictionary.csv`.
- Topic extracts: A = 29 cols, B = 33, C = 37; all rows, unweighted, still contain 7/77 (don't know), 9/99 (refused), 88 (none) codes - see `docs/CHEATSHEET_2_DATA.md` for recode rules.
- Key deadlines (corrected): proposal 25 Jul · Quiz 1 1 Aug · Quiz 2 + presentation 15 Aug · final + peer eval 21 Aug.

## Study site

https://chuatzeyee.github.io/ISSS623/ - topics, glossary, debate trainer, project playbook, practice quiz.
