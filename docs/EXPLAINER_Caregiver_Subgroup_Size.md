# Why the Caregivers-Only Analysis Drops to ~20k Rows - A Plain-English Briefing

**ISSS623 Group Project - Topic C · For the team discussion of the within-caregiver analysis idea**
*All numbers below computed from our merged working dataset (`brfss2024_topic_c_caregiver.csv`).*

---

## 1. The one-sentence answer

The caregiver-profile questions (relationship to the care recipient, hours of care, duration of care) were only ever asked of people who first said "yes, I am a caregiver" - so any analysis that uses those questions can only include caregivers, and there are only **21,137 caregivers** in the entire 2024 BRFSS. Everything after that is small subtractions for missing values.

## 2. The Russian-doll structure of the data

Think of the dataset as dolls nested inside each other. Each analysis can only use the doll it fits inside:

```
457,670  everyone in the 2024 BRFSS (50 states + territories)
   |
   |  only 16 states asked the caregiver question at all
   v
 96,287  people who were ASKED "are you a caregiver?"     <- module sample (21.0%)
   |
   |  keep those who answered it validly AND have a valid check-up answer
   v
 94,670  valid caregiver status + valid outcome
   |
   |  drop rows missing any predictor (our missingness rules)
   v
 87,098  OUR MAIN ANALYTIC SAMPLE (both caregivers and non-caregivers)
   |
   |  teammate's idea: analyse caregiver PROFILE items
   |  -> only caregivers were ever asked those items
   v
 21,137  caregivers (CAREGIV1 = yes)
   |
   |  keep those with a valid check-up outcome
   v
 20,976  caregivers with valid outcome
   |
   |  drop rows missing any predictor (same rules as main model)
   v
 19,263  CAREGIVERS-ONLY COMPLETE CASES  (~2,890 with a care gap, 15.0%)
```

So "down to ~20k" is not a punishment or a mistake - it is simply that the questions the teammate wants to use only exist for the 22.2% of the module sample who are caregivers.

## 3. Why can't we use the profile items on everyone?

Because of **skip logic** - the survey's built-in routing. The interviewer only asks "how many hours a week do you provide care?" (CRGVHRS2) if the person just said they ARE a caregiver. For the other 74,592 non-caregivers in the module sample, the answer isn't "zero hours" - the question was never asked, and the field is blank.

This has a sneaky consequence called **leakage**: if we put CRGVHRS2 into the main model (caregivers + non-caregivers together), the model doesn't need to learn anything about caregiving - it can just notice "this field is blank -> non-caregiver, this field has a value -> caregiver" and perfectly reconstruct CAREGIV1 from the blank pattern alone. Any "importance" the model assigns to hours-of-care would really just be the survey's routing, dressed up as a finding. That's why our design quarantines the three profile items (CRGVREL5 relationship, CRGVHRS2 hours, CRGVLNG2 duration) away from every predictive model.

**Inside the caregivers-only doll, the problem disappears.** When everyone in the analysis is a caregiver, the profile items are no longer a giveaway - they're fully answered (98.5-99.9% valid among caregivers) and genuinely informative: does caring 40+ hours a week predict skipping your own check-up more than caring 8 hours does? That's a real dose-response question, and it's only askable at N ~= 19-21k.

## 4. Is ~20k rows a problem? Mostly no, with two cautions

**Not a problem for basic power.** 19,263 caregivers with ~2,890 gap events is still large by any classroom standard - a logistic regression with ~19 predictors wants roughly 10-20 events per predictor, and we'd have ~150 events per predictor. The models will fit fine.

**Caution 1 - the estimates get noisier.** With 87k rows, a subgroup like "uninsured caregivers under 35 who smoke" still has hundreds of people; with 19k rows it may have a few dozen. Rare categories (Medigap, CHIP) get very thin. Confidence intervals widen; the tree's deeper splits become less trustworthy.

**Caution 2 - the scope rules.** The brief allows 1 baseline + minimum 2 ML models around ONE primary outcome (plus one secondary). A full third modelling exercise on the caregivers-only subset is beyond that scope. The safe framing is:
- **In scope, free:** descriptive profile of caregivers - gap rates by hours-of-care band, by relationship, by duration. Tables and charts, no models. This delivers most of the teammate's idea at zero scope cost.
- **Optional annex, clearly labelled exploratory:** one caregivers-only logistic regression using the profile items, presented as "beyond the scoped analysis". Ask the prof or flag it as optional - do not silently promise it in the proposal.

## 5. The takeaway for the meeting

The teammate's instinct (dig into which KIND of caregiver misses check-ups) is good - it is just an analysis that structurally can only happen inside the 21,137-caregiver doll, because the survey never asked anyone else those questions. We get ~92% of the value by doing it descriptively (allowed, zero risk), and we keep the option of one exploratory caregivers-only model as an annex if time permits and the prof is fine with it. What we must NOT do is drop the profile items into the main 87k-row models - the skip logic would leak caregiver status and quietly invalidate the headline result.
