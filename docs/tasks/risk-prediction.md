# C. Risk prediction

> *Where is this patient's trajectory heading if nothing changes?* Risk prediction is the prognostic counterpart of treatment-response prediction; it asks where the trajectory is going, not what to do about it.

## What we're estimating

A risk-prediction model estimates a *prognostic* quantity:

$$
\Pr(Y \in \mathcal{E} \mid X, T > t^*)
$$

— the probability that an outcome of interest $Y$ falls in some event set $\mathcal{E}$ during a follow-up window, conditional on features $X$ observed at time $t^*$ and on still being event-free at $t^*$.

Examples:

- Risk of hospital readmission within 30 days of discharge.
- Risk of stroke within 5 years.
- Risk of heart-failure decompensation within 90 days.
- Risk of seizure recurrence within 1 year of starting an ASM.
- Risk of cancer progression within 6 months of staging.
- Risk of Alzheimer's progression from MCI to dementia.
- Risk of ICU deterioration within 6 hours.

Common shapes:

- **Binary risk at a horizon** — logistic regression, gradient boosting, neural networks.
- **Time-to-event risk** — Cox proportional hazards, parametric survival, deep survival models. See [Methods → Survival analysis](../methods/survival-analysis.md).
- **Recurrent / cumulative events** — count models, gap-time survival, multi-state models.
- **Continuous-time hazard with covariate trajectories** — joint models, state-space models, neural ODEs.

## Why this is *not* the same task as treatment-response

It is essential to keep these straight.

| Risk prediction | Treatment-response prediction |
|---|---|
| Prognostic — what happens on the current path | Predictive — what happens under intervention |
| Identified from the joint distribution of $(X, Y)$ | Identified only under causal assumptions |
| Often safe to use observational data | Requires randomisation, IPTW, or DR-ML |
| Evaluation = calibration + discrimination | Evaluation = predicted-vs-observed *benefit* curve |
| Doesn't change with treatment policy | Embedded inside any decision-support model |

A risk-prediction model that says "this patient has a 20% chance of stroke" is a statement about *the world as it operates now*. Deploying the model to redirect care will change the world, which will change the underlying risk, which will violate the model's identifying distribution. This is the **prediction-as-intervention** problem; ignoring it is the most common cause of harm from deployed risk scores.

## Method families

### Tabular / EHR-based

- **Penalised logistic regression** — still the right default for a small, well-curated feature set. Easy to calibrate, easy to audit.
- **Gradient boosting** (XGBoost, LightGBM, CatBoost) — strong default for medium-to-large EHR cohorts.
- **Random forests** — comparable performance to GBDTs, sometimes easier to deploy.
- **Calibration layers** — Platt scaling, isotonic regression, or temperature scaling on a hold-out for any model whose outputs are not natively probabilistic.

### Sequence / longitudinal

- **Recurrent neural networks** (LSTM, GRU) — first wave of EHR-trajectory models (RETAIN, Dr.AI).
- **Transformers** — currently dominant for long EHR sequences (BEHRT, Med-BERT, ClinicalBERT, foundation-model variants).
- **State-space and ODE models** — for irregularly-sampled time series with strong physiology priors.

### Survival

- **Cox proportional hazards** — workhorse; covariate effects on a multiplicative hazard scale.
- **Accelerated failure time** — covariate effects on a time-scale.
- **Random survival forests** — non-parametric; useful when PH is violated.
- **DeepSurv, Nnet-Survival, DeepHit** — neural network parameterisations.
- **Cumulative incidence with competing risks** (Fine–Gray) — when the patient can experience non-target events that prevent observing the target.

### Foundation models for EHR

- Med-PaLM, Llama-3-Med, GatorTron, NYUTron — pretrained-on-EHR transformers fine-tuned for risk prediction. State of the art on several benchmarks; opaque to audit; require careful subgroup evaluation. See [Regulatory → FDA pathways](../regulatory/fda-pathways.md).

## Evaluation

A complete risk-prediction evaluation reports:

- **Discrimination.** AUROC for binary; concordance index (C-index) for survival.
- **Calibration.** Reliability diagram with isotonic / logistic regression fit; calibration intercept + slope.
- **Net benefit.** Decision-curve analysis across plausible threshold ranges.
- **Subgroup performance.** Discrimination + calibration stratified by sex, age band, race/ethnicity, site, scanner / device generation.
- **External validation.** A different site, a different time window, a different EHR vendor.
- **Robustness to distribution shift.** PSI / KL on input features against the training distribution.
- **Robustness to data drift.** Concept drift detection on the residuals.

Common evaluation traps:

- AUROC on a hold-out from the same hospital, same time period, no external validation.
- Mis-calibration ignored because AUROC was high.
- Subgroup metrics not reported.
- Implicit re-balancing of the training set, then evaluating on the re-balanced test set rather than the natural distribution.

See [Methods → Evaluation](../methods/evaluation.md) for the full rubric.

## The deployment loop

A deployed risk score reshapes the very risk it is trying to predict.

- Sepsis-alert systems lead to earlier antibiotics, which lower observed sepsis-mortality, which retrains the model.
- Readmission-risk models trigger discharge planning, which lowers observed readmission, which retrains the model.

This is the policy problem and the only correct framing is **off-policy evaluation** with explicit policy intervention. A bare "AUROC dropped 3 points after deployment" is *expected* — the model is partly successful — and must not be confused with concept drift.

References: [Lenert et al. 2019](https://academic.oup.com/jamia/article/26/10/957/5538650), [Finlayson et al. 2021 (NEJM)](https://www.nejm.org/doi/full/10.1056/NEJMc2104626) on the EHR-shift problem.

## A worked example — 30-day readmission

The clinical question: at hospital discharge, what is the probability of unplanned readmission within 30 days?

1. **Features.** Demographics, principal diagnosis, comorbidity index, prior 12-month admissions, discharge disposition, labs at last 24 hours.
2. **Outcome.** Unplanned readmission within 30 days, ascertained from EHR + state HIE.
3. **Method.** Gradient-boosted classifier with calibration on a temporal hold-out.
4. **Validation.** Three external sites; calibration plots per site; subgroup performance by race and primary insurance.
5. **Deployment plan.** Score posted in EHR at discharge; threshold triggers care-management referral; off-policy evaluation by comparing matched threshold-positive vs. threshold-negative patients.

Lessons from the famous failures of deployed readmission models:

- The model that worked at Hospital A failed at Hospital B because the EHR vendor coded discharge disposition differently.
- The model that worked at training time failed two years later because the post-pandemic care patterns shifted.
- The model that worked overall was biased against the patients with the worst social-determinants profile.

## Limitations

- A prognostic model is not a treatment-effect model. Do not derive treatment recommendations from a risk score alone.
- The model is only as honest as its labels. If outcomes are under-ascertained, risk is under-estimated and the harm is biased toward populations whose outcomes are under-recorded.
- Continuous monitoring is mandatory. A 2017 risk model deployed in 2024 has almost certainly drifted; you need a re-validation schedule and a halt condition.

## Exercises

1. **Build a 30-day readmission model on a public EHR dataset** (MIMIC-IV). Report discrimination, calibration, and at least three subgroup analyses.
2. **Implement decision-curve analysis** for your model. At what threshold range does the model help?
3. **Simulate a deployment loop.** Train, deploy a threshold-based policy, simulate the shifted outcome distribution, retrain. How fast does AUROC drift?

## References

1. Steyerberg EW. *Clinical Prediction Models.* 2nd ed. Springer, 2019.
2. Collins GS, Reitsma JB, Altman DG, Moons KGM. Transparent reporting of a multivariable prediction model for individual prognosis or diagnosis: the TRIPOD statement. *Ann Intern Med.* 2015;162(1):55–63.
3. Collins GS, Moons KGM, et al. TRIPOD+AI statement. *BMJ.* 2024;385:e078378.
4. Vickers AJ, Elkin EB. Decision curve analysis. *Med Decis Making.* 2006;26(6):565–574.
5. Lenert MC, et al. Prognostic models will be victims of their own success, unless …. *JAMIA.* 2019;26(12):1645–1650.
6. Finlayson SG, et al. The clinician and dataset shift in artificial intelligence. *NEJM.* 2021;385(3):283–286.

## Where to next

[D. Clinical decision support](clinical-decision-support.md) — what to do *with* a risk or treatment-response model once you have one.
