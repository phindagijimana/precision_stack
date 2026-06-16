# Survival analysis

> *Time-to-event modelling for clinical data. The right machinery for almost every prognostic question in precision medicine, and a prerequisite for nearly every clinical-trial endpoint.*

## Why a separate methodology

Most clinical outcomes have two features that ordinary regression cannot handle gracefully:

1. **Censoring.** Some patients have not experienced the event by the end of follow-up. They are not "no-event" — they are "no-event-yet".
2. **Time matters.** The clinical question is rarely "did the event happen by month 12?" — it is "at what rate does the event occur over follow-up?"

Survival analysis is the family of techniques designed around these features.

## Core objects

For a patient with event time $T$ and survival function $S(t) = \Pr(T > t)$, the related quantities are:

- **Cumulative distribution** $F(t) = 1 - S(t)$.
- **Hazard rate** $\lambda(t) = -\frac{d}{dt} \log S(t)$. Instantaneous event rate given survival up to $t$.
- **Cumulative hazard** $\Lambda(t) = \int_0^t \lambda(u)\,du = -\log S(t)$.
- **Restricted mean survival time** $\text{RMST}(\tau) = \int_0^\tau S(t)\,dt$. Often a more clinically interpretable endpoint than median survival.

Censoring is *non-informative* if the censoring process is independent of the event time given covariates. Most methods assume this; the assumption fails when withdrawal from follow-up correlates with health status.

## Non-parametric estimation

The **Kaplan-Meier** estimator gives $\hat{S}(t)$ stepwise without distributional assumptions. The **Nelson-Aalen** estimator does the same for $\hat{\Lambda}(t)$. Use them for:

- Overall survival curves.
- Subgroup survival curves with log-rank tests.
- Visualising whether proportional-hazards is plausible.

The log-rank test is the standard for two-group comparison. Limitations: low power against late-emerging differences; not informative about effect size.

## Semi-parametric — Cox proportional hazards

The workhorse covariate-adjusted survival model:

$$
\lambda(t \mid X) = \lambda_0(t) \exp(X^\top \beta)
$$

Strengths: no parametric assumption on $\lambda_0(t)$; $\exp(\beta_j)$ is the hazard ratio; estimation by partial likelihood is well-developed.

The proportional-hazards (PH) assumption — covariate effects are constant in time — is the model's load-bearing wall. Check via:

- Schoenfeld residuals.
- Log-log survival plots.
- Stratification by covariate-time interactions.

When PH fails, switch to time-varying coefficients, accelerated failure time models, or non-parametric alternatives.

## Parametric — Accelerated Failure Time

The AFT family parameterises log-event-times directly:

$$
\log T_i = X_i^\top \beta + \sigma W_i
$$

where $W_i$ has a chosen distribution (Weibull, lognormal, log-logistic, generalised gamma). Strengths: interpretable on the time scale, well-suited to extrapolation, often the right choice for engineering tasks like simulating clinical-trial follow-up.

## Competing risks

If a patient can experience non-target events that prevent the target event from being observed (e.g. death-from-other-causes in cancer), naive censoring is biased. The right tools:

- **Cause-specific hazards.** Model each cause separately; uses Cox.
- **Sub-distribution hazards** (Fine–Gray). Model the cumulative incidence of each cause; uses a modified Cox-style partial likelihood.

Reference: [Fine & Gray 1999](https://www.jstor.org/stable/2670170).

## Multi-state models

When the patient transitions between several clinical states (healthy → MCI → dementia, or progression-free → progression → death), multi-state survival models capture every transition explicitly. Each transition has its own hazard model and the overall picture is a Markov-on-states or semi-Markov framework.

## Time-dependent covariates

If a covariate $X(t)$ is time-varying, Cox regression with a counting-process formulation handles it. The interpretation is delicate: a time-varying covariate that depends on prior outcome can mediate or block the very effect being estimated. This is where the [G-methods from causal inference](causal-inference.md) become essential.

## Deep survival models

Several neural-network parameterisations:

- **DeepSurv** ([Katzman et al. 2018](https://bmcmedresmethodol.biomedcentral.com/articles/10.1186/s12874-018-0482-1)) — Cox-like risk function from a neural net.
- **DeepHit** ([Lee et al. 2018](https://aaai.org/ojs/index.php/AAAI/article/view/11842)) — directly estimates the joint distribution of competing-risk event times.
- **Nnet-Survival, PCHazard** — discrete-time hazard parameterisations that are easy to optimise.
- **Neural-ODE survival** — continuous-time, irregular-observation handling.

Trade-offs: more capacity to capture non-linear effects, harder calibration, harder to interpret, harder to validate externally. For most clinical-AI projects, a regularised Cox or gradient-boosted survival model (e.g. XGBoost-AFT) is a strong baseline.

## Evaluation

- **Discrimination.** Concordance index (C-index); time-dependent AUROC; Brier score.
- **Calibration.** Calibration plot at fixed horizons; calibration-in-the-large.
- **Net benefit.** Decision-curve analysis at each clinically relevant horizon.
- **Subgroup performance.** All metrics stratified by sex, age band, race, site.
- **External validation.** A different site / time / scanner.

Pitfalls:

- C-index near 0.5 doesn't always mean a useless model — the metric is insensitive to time-varying discrimination.
- Calibration drift is silent on AUROC and lethal on decision-curve analysis.
- Brier scores are sensitive to censoring distribution; report on a comparable test set.

## Sample size

A useful working rule: the number of *events* (not the number of patients) is what powers a survival analysis. Common thresholds:

- ≥ 10 events per candidate predictor for stable estimation (Peduzzi rule).
- ≥ 20 events per estimated effect for reliable hazard ratios.

These are minima; modern penalised methods relax them somewhat. See [Riley et al. 2020](https://www.bmj.com/content/368/bmj.m441) for modern minimum-sample-size derivations.

## A worked example — seizure-recurrence model after ASM monotherapy

The clinical question: among newly-diagnosed focal-epilepsy patients starting first ASM monotherapy, model the time to seizure recurrence.

1. **Eligibility.** Adult focal epilepsy, no prior ASM, no other CNS-active drug.
2. **Time-zero.** First dispensed dose of the ASM.
3. **Outcome.** Time to next clinically documented seizure or to second ASM start.
4. **Censoring.** End-of-follow-up, transfer of care, withdrawal of consent.
5. **Covariates.** MRI findings (epileptogenic lesion present/absent), baseline seizure frequency, EEG interictal abnormality, AED choice, age, sex.
6. **Model.** Penalised Cox + interaction term for MRI × AED choice; XGBoost-AFT for non-linearity check.
7. **Validation.** Held-out hospital; calibration at 3, 6, 12 months; concordance index; subgroup C-index.
8. **Reporting.** TRIPOD+AI checklist; pre-specified subgroup analyses.

The model becomes the prognostic input to the surgical decision-support analysis in [Case study → Epilepsy surgery](../case-studies/epilepsy.md).

## Exercises

1. **Implement a Kaplan-Meier estimator from scratch** and reproduce a published curve.
2. **Build a Cox model with non-PH violations.** Diagnose with Schoenfeld residuals and pick a remediation (stratification, AFT, time-varying coefficient).
3. **Compare a Cox model to DeepSurv** on a public dataset (METABRIC, NACC). Report calibration, C-index, and external-cohort generalisation.

## References

1. Klein JP, Moeschberger ML. *Survival Analysis: Techniques for Censored and Truncated Data.* 2nd ed. Springer, 2003.
2. Therneau TM, Grambsch PM. *Modeling Survival Data: Extending the Cox Model.* Springer, 2000.
3. Putter H, Fiocco M, Geskus RB. Tutorial in biostatistics: competing risks and multi-state models. *Stat Med.* 2007;26(11):2389–2430.
4. Fine JP, Gray RJ. A proportional hazards model for the subdistribution of a competing risk. *J Am Stat Assoc.* 1999;94(446):496–509.
5. Katzman JL, et al. DeepSurv. *BMC Med Res Methodol.* 2018;18:24.
6. Lee C, et al. DeepHit. *AAAI* 2018.
7. Riley RD, et al. Minimum sample size for developing a multivariable prediction model. *Stat Med.* 2019;38(7):1276–1296.

## Where to next

[Unsupervised subtyping](unsupervised-subtyping.md) — the methods behind the [stratification chapter](../tasks/stratification.md).
