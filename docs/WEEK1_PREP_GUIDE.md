# ISSS623 Week 1 - Detailed Step-by-Step Preparation Plan

**Session 1: Sat 18 Jul 2026, 8:15–11:30am, SOE/SCIS2 Seminar Room 5-2. Instructor: Dr Sean Lam (seanlam@smu.edu.sg).**
(Dates corrected: the prof's original materials were one week early. All deadlines below are shifted +7 days.)
The mini-debate happens in Session 1 today. Total prep time budget below: ~8–10 hours.

Companion files in this folder:
- `DEBATE_00_Overview_and_Tactics.md` - debate format, timing, tactics
- `DEBATE_GroupA_Proposition.md` - argument-by-argument case FOR the motion
- `DEBATE_GroupB_Opposition.md` - argument-by-argument case AGAINST the motion
- `DEBATE_GroupC_PolicyJury.md` - evaluation criteria + synthesis script
- `data/` - the BRFSS 2024 dataset, codebook, variable dictionary, topic CSVs and extraction notebook (see Step 5)

---

# WHAT CHANGED IN THE v3 SLIDES (`2_Lecture 1_2026_v3.pdf`, 78 pp, replaces the 81-pp deck)

**Added (study these):**
1. **Project Guidance slide (new, near the end)** - the prof now supplies everything for the project:
   - Sample extraction code: `Lecture 1/BRFSS_Extraction.ipynb` in the course repo
   - Dataset: `LLCP2024ASC.zip` (41.5 MB zip → extracts to a **922 MB ASCII file**)
   - Codebook (HTML) zip + an extracted CSV: `brfss2024_variable_dictionary.csv`
   - Pre-extracted per-topic data files: `brfss2024_topic_a.csv`, `brfss2024_topic_b.csv`, `brfss2024_topic_c.csv`
   - All of these are already downloaded into `data/` in this folder.
2. **Key Takeaways slide (new closer)** - memorize the 3 lines: (1) healthcare data science must start with the **healthcare problem, not the dataset**; (2) **learning health systems** use data to continuously improve care; (3) the **data science value chain is an end-to-end process**. Perfect for wrap-up comments and quiz stems.
3. **Simplified Drive-mount flow** with a snippet that saves notebook outputs into your Drive (`os.makedirs('/content/drive/My Drive/ISSS623/Lecture_1_Outputs', exist_ok=True)` then `!mv brfss2024_variable_dictionary.csv "{drive_path}"`).

**Removed (deprioritize for Quiz 1, keep as background):**
- "Challenges in ASEAN" slide
- Both "Sources of Healthcare Data" description tables (EHR/registries/open data; IoT/claims/trials/genomics/sentiment)
- The Open Data slide (data.gov.sg / MIMIC-III & MIMIC-IV)
- One Learning Health System slide (the enablers wheel adapted from IOM 2012)

**Retitled:** "Singapore Healthcare Ecosystem **Across the Care Continuum**" and "Coordination of Care **across the Continuum**" - the continuum is now the explicit organizing idea of Segment 1.

---

# STEP 0 - Understand what you're being graded on (15 min)

**Where:** `0_CourseDog_Outline_2026.pdf` and `1_Course_Introduction.pdf` (this folder).

**What to learn:**
1. Assessment table (corrected +1 week): Participation 15% (Weeks 1–4, incl. debate 18 Jul) · Proposal 10% (25 Jul) · Quiz 1 20% (1 Aug) · Quiz 2 25% (15 Aug) · Presentation + Final 20% (15 & 21 Aug) · Peer eval 10% (21 Aug).
2. **Grading is relative/curved** ("Components B,B,B,B but final grade is A" is possible). Every mark of participation counts against classmates.
3. Participation rubric: "Very Good" = participates actively AND comes prepared, contributes to group learning. Attendance alone = pass only.
4. Session 1 agenda (3 segments): Healthcare Landscape → Health Data Ecosystem → Hands-on Lab + Group Project briefing.
5. Prerequisite expectations: Python useful but not required; groups should mix skills (Python, R, database, project management).

**Action items:**
- [ ] Put all 5 deadlines in your calendar with reminders 3 days before.
- [ ] Note the Wooclap code **FGTQDUH** - the very first participation activity ("Which organization are you from? Expectations in 3 words"). Prepare your two answers now.

---

# STEP 1 - Set up your tools (60–90 min, do this FIRST - everything else can be read on the train)

## 1.1 Google Colab (30 min)
**Where to go:** https://colab.research.google.com/ (needs a Google account).

**Step-by-step:**
1. Sign in → File → **Open Notebook** → **GitHub** tab.
2. Paste repo URL: `https://github.com/ISSS623-AHA/ISSS623_2024` (this is the actual course repo shown in the slides).
3. Open **`Lecture 1/Lecture_1.ipynb`**.
4. Click **Connect → Connect to hosted runtime** (top right; green tick + RAM/Disk bars = connected).
5. Run every cell top-to-bottom (click ▶ or Ctrl-Enter). The `[3]` style number shows execution order.
6. Try the **Gemini** button (✦ icon) - the instructor explicitly demos "learning in the era of GenAI"; ask it to explain one code block.
7. Practice **mounting Google Drive**: Files panel → Mount Drive icon → run the generated cell (`from google.colab import drive; drive.mount('/content/drive')`) → authorize → confirm `drive/MyDrive` appears. Output files land in `/content/`.

**What to learn:** how to open a notebook from GitHub, connect a runtime, run cells, find outputs, mount Drive. This is exactly what Segment 3A walks through - being fluent lets you help groupmates (participation).

## 1.2 GitHub + GitHub Desktop (30 min)
**Where to go:** https://github.com/ (create account) and https://desktop.github.com/ (install). Detailed walkthrough: **`5_Github Guide.pdf`** in this folder - read all 25 pages.

**Step-by-step:**
1. Create GitHub account (use a professional username; you'll share it with your group).
2. Install GitHub Desktop → sign in.
3. **Clone a repository** → URL tab → `https://github.com/ISSS623-AHA/ISSS623_2024` → choose an **empty** local folder → Clone.
4. Click "Show in Explorer" and verify folders: Lecture 1–4, Group_Project, plus the hidden `.git` folder (never delete `.git`).
5. Learn the collaboration loop the slides mandate: **start of session = Pull; end of session = Commit then Push.** Practice once: edit README locally → see it under "Changes" → write a commit message → Commit → Push (on your own test repo, not the course repo).

## 1.3 VS Code (optional, 15 min)
**Where:** https://code.visualstudio.com/Download. The instructor's preferred IDE; supports notebooks, Git integration, and AI assistants. Install + open the cloned repo folder. Optional but useful for the group project.

## 1.4 Python refresh (60–120 min depending on rustiness)
**Where:** `Lecture 1/Lecture_1.ipynb` itself is the syllabus. The 12 topics you must be comfortable with:

| # | Topic | Self-check: can you do this without looking it up? |
|---|---|---|
| 1–2 | Printing, variables | Store `patient_id`, `blood_pressure`, `has_diabetes`; print labelled output |
| 3 | Lists, tuples, dicts, pd.Series | `ages[-1]`, `ages.append()`, `patient["age"]`, `pd.Series(ages).mean()` |
| 4 | List comprehension | `[age for age in ages if age >= 60]`; conditional version `["Older adult" if a>=65 else "Adult" for a in ages]` |
| 5 | Conditionals & loops | BMI classifier with if/elif/else; for-loop with condition inside; while-loop counter |
| 6 | Functions | `def calculate_bmi(weight_kg, height_m): return weight_kg/height_m**2` + a classify function |
| 7–9 | File I/O | `with open(...,"w")`, `pd.read_csv/to_csv`, `pd.read_excel/to_excel(index=False)` |
| 10 | Data wrangling | `df.head() / .info() / .describe()`, `df.rename(columns={...})`, `df.isna().sum()`, `df["age"].fillna(df["age"].median())`, boolean filter `df[df["age"]>=65]`, column select `df[["a","b"]]` |
| 11–12 | Groupby | `df.groupby("gender")["age"].mean()`; multi-agg `df.groupby("g").agg(n=("id","count"), mean_bp=("bp","mean"))`; `.reset_index()` |

**Practice drill (30 min, mirrors the notebook's "Mini End-to-End Example"):**
1. Build a 5-row DataFrame with age, sex, exercise, general_health.
2. Create binary outcome: `df["poor_fair"] = df["general_health"].isin(["Poor","Fair"]).astype(int)`.
3. Create age groups with a conditional list comprehension.
4. `groupby("age_group").agg(...)` → compute a rate column. 
This exact pattern (recode → group → rate) is the core of the BRFSS group project.

**If Python is new to you:** work through the notebook with Gemini/ChatGPT as tutor, following the instructor's 8 rules (slide "How GenAI can be a good Python Tutor"): explain before generating; small examples; run line by line; paste errors and ask why; modify generated code yourself; ask "explain like I'm new"; keep a personal snippets notebook; always verify outputs.

---

# STEP 2 - Learn the Healthcare Landscape (Segment 1 content, ~2 hrs)

**Where:** `2_Lecture 1_2026_v3.pdf` (current deck) pages 4–22. Read in this order, making one flashcard per bolded item. Note the ASEAN challenges slide was dropped in v3.

## 2.1 Singapore public healthcare system (slides 6–8)
**Learn the 4-stage historical evolution:**
1. Government-owned hospitals under MOH →
2. **Corporatization** (NUH first, 1985; each hospital gets its own Board; autonomy + flexibility) →
3. **Healthcare clusters** (2001, started with 2 clusters; integrated hospitals + polyclinics + specialist centres around patients) →
4. **Regional Health Systems** - 2 → 6 → reorganized 2017 into today's **3 clusters**:
   - **SingHealth** (east; SGH, KKH, CGH, SKH, Duke-NUS AMC)
   - **NHG** (central; TTSH, KTPH, Yishun CH, LKC Medicine)
   - **NUHS** (west; NUH, NTFGH, Alexandra, NUS)
**Supporting agencies to recognize:** Synapxe (national health-tech), ALPS (healthcare supply chain), HSA (regulator), HPB (health promotion), AIC (Agency for Integrated Care - ILTC coordination), SCDF (emergency), MOHT.

## 2.2 Care continuum / ecosystem (slide 7 + "Health Outcomes vs Burden of Care" + "Coordination of Care")
**Learn the 5 care settings and examples in each:**
1. **Preventive** - school, community, workplace, home
2. **Primary** - polyclinics, GPs, family medicine clinics, dental
3. **Acute** - public/private hospitals, A&E, specialist outpatient clinics (SOC), community hospitals
4. **ILTC (Intermediate & Long-Term Care)** - nursing homes, hospices, home-based & centre-based care, mostly VWO-run
5. **Social & community** - senior activity centres, FSCs/SSOs, CSR
**Key insight to be able to articulate in class:** moving care left (prevention, home care, telemedicine, chronic disease management) = better outcomes AND lower cost; acute care = highest cost. This is *why* analytics that predicts and prevents is valuable.
**Coordination of care has 4 layers:** ancillary/support services; clinical care (care plans, information sharing, medication reconciliation); front-line patient care (appointments, referrals, handovers); **IT/information systems (integrated records, care pathways, data analytics & feedback)** - the last one is this course.

## 2.3 The 5 memorize-cold statistics (slide "Challenges in Singapore")
| Stat | Value | Source year |
|---|---|---|
| Population aged 65+ | 20.7% (2025) → **23.9% by 2030** (~1 in 4) | population.gov.sg |
| Life expectancy at birth | **83.9 years** (2025) | Straits Times 2026 |
| Health expenditure | **$22B (2018) → $59B (2030)** projected | MOH 2021 |
| Old-age support ratio (aged 20–64 per 65+) | **5.7 (2015) → 2.7 (2030)** | Population in Brief 2022 |
| Healthcare workforce | **129k (2024) → ~156k needed (2030)** (+20%); WHO global shortfall **18M by 2030** | MOH 2026 |

## 2.4 SG healthcare philosophy & financing (slides 12, 16–17)
**1993 MOH White Paper "Affordable Healthcare" - 5 principles (learn verbatim-ish):**
1. Nurture a healthy nation by promoting good health
2. Personal responsibility for one's health; avoid over-reliance on welfare/insurance
3. Good and affordable basic medical services for all
4. Rely on competition and market forces for efficiency
5. Government intervenes directly **when the market fails** to keep costs down

**Financing stack (S + 3M) - know what each layer is FOR:**
| Layer | Mechanism | Purpose |
|---|---|---|
| **Subsidies** | Up to 80% subvention in lower ward classes | Universal access with co-payment |
| **MediSave** | Compulsory 6–8% of income into personal account | Individual savings for smaller bills |
| **MediShield Life** (+ Integrated Shield Plans, ElderShield/CareShield) | National catastrophic insurance | Risk pooling for large bills |
| **Medifund** | Government endowment fund | Last-resort safety net for the indigent |
Plus non-government layer: private insurers, NTUC Income/Health, C3A, TOUCH, People's Association.

## 2.5 Policy strategy evolution (slide 13)
- **Healthcare 2020 (~2012):** Accessibility + Quality + Affordability.
- **Beyond Healthcare 2020 / "3 Beyonds" (~2016)** - learn all three and be able to give an analytics example for each:
  1. **Beyond healthcare to health** (prevention, Healthier SG → e.g., population risk stratification)
  2. **Beyond hospital to community** (care near home → e.g., telemonitoring analytics, demand forecasting for community care)
  3. **Beyond quality to value** (best value, sustainable → e.g., VDC outcome/cost benchmarking)

## 2.6 Value in Health & VDC (slides 18–22)
- **Definition (Porter):** Value = health outcomes that matter to patients ÷ total resources/cost across the full cycle of care. Cost-cutting without outcomes can be harmful; outcomes are condition-specific and multidimensional.
- **3 key shifts:** supply-driven → patient-centred; volume & profitability → outcomes achieved (**PROMs** = patient-reported outcome measures, **PREMs** = patient-reported experience measures); fragmented → integrated.
- **AVBC** (Appropriate and Value-Based Care) FOCUS pillars: Financing models, Outcome measurement, Cost-effective interventions, Unwarranted variation & waste reduction, Skills/culture.
- **VDC (Value Driven Care, 2017, part of 3 Beyonds):** MOH picked **17 high-impact conditions** (e.g., cataract, total knee replacement, stroke, pneumonia, CHF); standardized clinical outcome indicators for like-for-like benchmarking across public healthcare institutions; total cost benchmarking enables **bundled payments**; **Clinical Quality Index** = % of cases where ALL indicators met (all-or-none composite). Timeline: 2017 3 Beyonds+VDC → 2018 fee benchmarks+ALPS → 2022 cancer drug list → 2023/24 Healthier SG, capitation funding for PHIs, implant subsidy list.

## 2.7 WHO Health System Framework (slide 10) - likely quiz material and the answer scaffold for Class Exercise 1
**6 building blocks:** 1 Service delivery · 2 Health workforce · 3 Information · 4 Medical products, vaccines & technologies · 5 Financing · 6 Leadership/governance
→ mediated by **access, coverage, quality, safety** →
**4 goals:** improved health (level AND equity) · responsiveness · social & financial risk protection · improved efficiency.

**Class Exercise 1 will be: "Who are the stakeholders in a health system?" (10 min).** Prepare a structured answer, e.g. by category: patients & caregivers · providers (doctors, nurses, allied health, hospitals, GPs, VWOs) · payers (government, insurers, employers) · regulators & policy (MOH, HSA) · suppliers (pharma, medtech, IT vendors like Synapxe/Epic) · researchers & academia · community/social services · the healthy public (taxpayers, future patients). Volunteering a category others miss (e.g., informal caregivers, IT vendors, healthy citizens) = participation marks.

---

# STEP 3 - Learn the Health Data Ecosystem (Segment 2 content, ~2 hrs)

**Where:** `2_Lecture 1_2026.pdf` pages 28–46, plus the two book chapters and the Liu paper (Step 4).

## 3.1 Why data: uses of healthcare data (slide "Harnessing and Using Data")
Six core uses: disease/biological insights · hospital efficiency · new care tools · patient outcomes/experience · population health · lower cost. Broader: precision medicine, drug development, trials, public health surveillance, quality & safety monitoring, policy, predictive analytics, patient empowerment. Be ready to name 3–4 with an example each.

## 3.2 DIKW pyramid (slide "DIKW") - learn with one worked example
**Data → Information → Knowledge → Wisdom**, i.e., raw → meaning → context → action.
Worked example to memorize (from the slide): motion sensor `{14:00, 255}, {14:10, 0}` (data) → "no motion since 14:10" (information) → "senior motionless for x hours, may have fainted" (knowledge) → "activate caregiver" (wisdom/action). Alternative examples: glucose monitoring, hospital early-warning scores, medication adherence. Quiz-friendly: be able to classify a given statement into the correct DIKW layer.

## 3.3 Learning Health System (LHS) - two slides + Liu et al. paper
- **IOM 2008 definition (quote fragments worth memorizing):** a system where "**science, informatics, incentives, and culture** are aligned for **continuous improvement and innovation**, best practices **seamlessly embedded** in the delivery process, new knowledge captured as an **integral by-product** of the delivery experience."
- **4-step loop:** pick a high-priority clinical process → build evidence-based best-practice guideline → blend guideline into clinical workflow WITH a data system tracking it → feed data into a "lean learning loop" → repeat. Grounded in *measured reality* (data from practice, not assumption).
- **Enablers wheel:** Talent (data literacy, citizen data science) · Data (quality, integration, interoperability) · Technology (big data & AI/ML infra) · Governance (privacy, security, AI governance, ethics) · Implementation (implementation science, cost-effectiveness, sustainable scale) · **Collaboration** (across research/operational/IT domains - highlighted in red on the slide, i.e., the instructor's emphasis).
- **Quadruple Aims:** 1 enhance patient experience · 2 improve population health · 3 reduce cost · 4 improve provider work-life (the 4th was added to the Triple Aim due to clinician burnout - a likely quiz nuance).

## 3.4 Research IT vs Operational IT (two table slides - high quiz probability)
Learn the contrast dimensions: purpose (research/analytics vs clinical care & billing) · funding (grants vs operational budget) · data (multimodal incl. genomic/survey vs primarily EHR) · governance (IRB/ethics vs regulatory/privacy compliance) · ownership (PI vs institution) · sharing (de-identified encouraged vs restricted) · teams (small project-based vs large multidisciplinary) · design (flexible/experimental vs stable/reliable) · change (frequent vs controlled) · support (downtime tolerable vs mission-critical 24/7) · success (publications vs patient outcomes/uptime) · validation (methodological/retrospective vs rigorous clinical validation, UAT, security audits).
**The Liu et al. 2025 thesis:** a rapid & continuous LHS requires *socio-technical harmonization* of these two worlds - 3 phases: Learning (EHR snapshot → data lake → ETL → common data models **OMOP/i2b2** → research dataset → model dev) → Implementation (multi-stakeholder collaboration, hosting/execution/integration decisions, clinical workflow integration via alerts/dashboards/EHR notes, monitoring/version control/audit) → Assessment (define outcomes with frontline HCWs, frameworks like **RE-AIM**, extract, analyze, classify as QI or research) + a continuous learning cycle (retrain, refine, feed back).

## 3.5 Sources of health data (TRIMMED in v3 - the detailed source tables and open-data slide were removed; learn the framework, deprioritize the vendor lists)
- **Patient–provider interactions (health system):** demographic, clinical (vitals, meds, diagnostics), administrative (claims, reimbursements) - i.e., **EHR/EMR** (NEHR, Epic, eHintS), electronic data warehouses, **disease registries** (National Diabetes Registry, National Death Registry, PAROS resuscitation registry, SingCLOUD cardiac).
- **External:** consumer-generated (wearables, smart devices), environmental (air/water/food quality, transport).
- **Research databases:** genomics, clinical trials, observational studies.
- Additional table: IoT/medical devices · claims · clinical trials & health services research (PROMs, surveys) · genomics · patient behaviour/sentiment (social web).
- **Open data:** data.gov.sg (>70 agencies, >100 health datasets) · NIH · PhysioNet (free curated); Datarage.ai, Flatiron, Thomson Reuters (paid); scraping (non-curated).
- **NEHR = "One Patient, One Health Record"**: national record following the patient across GP → hospital → community hospital; benefits = coordination, safety, efficiency.

## 3.6 Data science landscape & value chain (3 slides)
- **Nesting: AI ⊃ ML ⊃ Deep Learning.** AI adds planning, expert systems, robotics, vision; ML = supervised/unsupervised/reinforcement; DL = multilayer neural nets (pathology/radiology images, notes, NLP, LLMs). Takeaway line: "AI in healthcare goes beyond deep learning."
- **Data science definition (Aliferis & Simon):** science & technology of (a) data measurement/collection design, (b) representation, management, harmonization, secure storage/transmission, (c) analysis & interpretation, (d) deployment of results - in applied problem-solving. Formula on slide: **Data Assets + Scientific Thinking**.
- **Gartner analytics value chain (course focus = first three):**
  | Stage | Question | Examples |
  |---|---|---|
  | Descriptive | What happened? | EDA, dashboards, reports |
  | Diagnostic | Why did it happen? | drill-down, association, causality |
  | Predictive | What will happen? | risk scores, forecasting, anomaly detection, DL |
  | Prescriptive | How can we make it happen? | simulation, optimization, reinforcement learning |
  | Cognitive | (extension) | adaptive/continual learning, LLMs/GenAI |
  Hindsight → Insight → Foresight; value and difficulty both increase rightward.
- **Clinical-grade AI/ML lifecycle (Aliferis & Simon ch. 6 figure):** establish performance & safety requirements → data design & collection → "first-pass" analysis & modeling → model optimization & validation → production models & delivery (health-economic + implementation science considerations) → **model monitoring & safeguards**; in parallel: regulatory/ethical/legal/societal considerations, IP, ancillary work products. Course focus box = data design → first-pass modeling → optimization/validation.

---

# STEP 4 - Readings (2–3 hrs skim now; deep-read before Quiz 1 on 1 Aug)

| Priority | File / Where | What to extract |
|---|---|---|
| 1 | `1. The Need for Best Practices Enabling Trust in AI and ML.pdf` (Aliferis & Simon ch.1, in folder) | Definitions of AI/ML/data science; why healthcare AI needs best practices; sources of failure/mistrust; the case for rigorous validation |
| 2 | `6. The Development Process and Lifecycle of Clinical Grade...pdf` (ch.6, in folder) | The lifecycle stages (matches slide in 3.6); what makes a model "clinical grade"; difference from ordinary ML projects |
| 3 | `Liu et al. - 2025 - ...socio-technical harmonization...pdf` (in folder) | Research vs operational IT table (3.4); 3-phase LHS pipeline; OMOP/i2b2; RE-AIM; why harmonization is the bottleneck |
| 4 | https://www.academia.sg/explainer/keywords/ageing-population/ | The **assigned debate quick-read**. Read twice; extract every number and every framing argument (feeds the debate files) |

Reading method: for each, write 5 bullet takeaways + 3 possible quiz questions in your own notes. The lecture only includes "relevant sections covered in the lecture notes," so prioritize topics that appeared in Step 2–3.

---

# STEP 5 - Group Project head start (1–2 hrs; proposal due 25 Jul = 1 week after Session 1)

**Where:** `6_Group Project Brief.pdf` (18 pages - read fully), `8_Group_Project_Assess_Rubric_2026.pdf`, `7_Group_Project_Peer_Evaluation_Form.pdf`, v3 slides "Project Guidance" + project section, and the **`data/` folder here (everything is already downloaded)**.

**Step-by-step (updated per v3 Project Guidance):**
1. **Use the prof's supplied assets in `data/`** (no need to hunt on the CDC site):
   - `LLCP2024ASC.zip` - the full 2024 dataset (fixed-width ASCII; unzips to 922 MB `LLCP2024.ASC` - do NOT commit or email it)
   - `USCODE24_LLCP_082125.HTML` - the official codebook (unzipped from `codebook24_llcp-v2-508.zip`)
   - `brfss2024_variable_dictionary.csv` - the prof's parsed codebook (variable, label, column positions, type + suggested Topic B/C fields)
   - `brfss2024_topic_a/b/c.csv` - **pre-extracted analysis files, 457,670 respondents each** (A: 29 cols, B: 33, C: 37) - your group can start modelling immediately
   - `BRFSS_Extraction.ipynb` - the prof's extraction logic (fixed-width parsing driven by the variable dictionary); study it so you can extract EXTRA fields beyond the suggestions ("You may go beyond these suggestions")
2. **Verified variable lists (from the actual topic CSV headers - no longer guesses):**
   - **Topic A High Health Burden (29 cols):** GENHLTH, PHYSHLTH, MENTHLTH, POORHLTH + PERSDOC3, MEDCOST1, CHECKUP1, chronic flags (CVDSTRK3, CHCCOPD3, ADDEPEV3, CHCKDNY2, DIABETE4, _MICHD, _CASTHM1, _DRDXAR2), demographics/SES (SEXVAR, MARITAL, EMPLOY1, _RACEGR3, _AGE80, _EDUCAG, _INCOMG1), behaviours (_TOTINDA, _SMOKER3, _RFBING6), access (_HLTHPL2), _URBSTAT, _BMI5CAT
   - **Topic B Mental Health (33 cols):** MENTHLTH outcome + disability battery (DEAF, BLIND, DECIDE, DIFFWALK, DIFFDRES, DIFFALON), _CURECI3 (e-cigs), _RFDRHV9 (heavy drinking), _AGEG5YR + same access/chronic/SES core
   - **Topic C Preventive Care Gap (37 cols):** CHECKUP1, MEDCOST1, PRIMINS2 (note: 2024 name is PRIMINS2, not PRIMINSR), _DENVST3 (dental), plus GENHLTH/PHYSHLTH/MENTHLTH, disability battery, and the same core (FLUSHOT7 is NOT in the pre-extracted file - extract it yourself with the notebook if you want the flu-shot outcome)
   - Calculated variables start with `_` (e.g., _AGEG5YR, _BMI5CAT, _INCOMG1) - they are CDC-derived and already partially cleaned.
3. **Learn BRFSS data quirks now:** 7/77 = "Don't know/Not sure", 9/99 = "Refused", BLANK = not asked/missing; optional modules vary by state (the brief warns about this). Recoding these responsibly is worth 6% of your grade.
4. **Know the required scope:** 1 primary outcome (+ optional secondary), **max 20 predictors**, **1 baseline regression** (linear/logistic), **≥2 ML models**, **2024 data only**, **survey weights NOT used**.
5. **Proposal contents (≤2 pages excl. annexes):** project question · public health motivation · brief literature context · chosen outcome + predictors (show codebook awareness) · planned models · team role allocation. Rubric: framing 2% · data feasibility & variables 3% (biggest) · literature 2% · analysis plan 2% · roles 1%.
6. **Roles to fill (~6 people):** project/proposal lead · data preparation lead · descriptive analysis lead · modelling lead · interpretation/report lead (+ presentation). Decide which role suits you and say so during group formation.
7. Peer evaluation (10%) uses **SPOT**: Supportive, Proactive, Openness, Thorough (15% each) + overall participation/quantity/quality/ideas (10% each). Translation: be responsive on the group chat and hit internal deadlines.
8. Literature anchors cited in the slides (use in the proposal): Jiang, Hesser & Williams 2006 (BRFSS HRQOL); Parekh & Fahim 2021 (BRFSS ML for marijuana use); Clark et al. 2021 (ML predicting self-rated health from BRFSS).

**Before Session 1:** form a view of which topic you'd prefer and why, so your group can converge fast and spend Week 1 on variable selection instead of debating topics.

---

# STEP 6 - Debate preparation (1.5–2 hrs)

**Where:** `3_Class Exercise - Mini Structured Debate on Aging.pdf`, `4_Guidance for Aging Debate.pdf`, the academia.sg explainer, and the four `DEBATE_*.md` files in this folder.

**Step-by-step:**
1. Read `DEBATE_00_Overview_and_Tactics.md` (format, timing, delivery tactics).
2. Read `DEBATE_GroupA_Proposition.md` and `DEBATE_GroupB_Opposition.md` fully - you don't know which group you'll be assigned, and knowing the other side's case is how you pre-empt rebuttals.
3. Read `DEBATE_GroupC_PolicyJury.md` - even if you're in A or B, this is what the jury will score you on.
4. Memorize the **stat card** (in the overview file): 23.9% · 83.9 yrs · $22B→$59B · 5.7→2.7 · 129k→156k.
5. Pick, for each group role, the 2 arguments YOU would personally deliver, and rehearse saying each in ≤60 seconds out loud.

---

# STEP 7 - Night-before checklist

- [ ] Laptop charged; Colab tested; `Lecture_1.ipynb` runs end-to-end
- [ ] GitHub Desktop installed; course repo cloned
- [ ] Wooclap answers ready (organization + 3-word expectations)
- [ ] Stakeholder answer ready for Class Exercise 1
- [ ] Debate stat card memorized; 2 arguments per role rehearsed
- [ ] Preferred BRFSS topic chosen + 3 candidate variables noted
- [ ] Know which project role you want; names of potential groupmates
- [ ] Skimmed all 3 readings; deadlines in calendar
