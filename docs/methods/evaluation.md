# Evaluation and calibration

> *Discrimination is the metric that gets a paper published. Calibration is the metric that keeps a deployed model safe.*

## The four-quadrant model

A complete evaluation of a precision-medicine model spans four quadrants:

|                  | Discrimination               | Calibration                 |
|------------------|-------------------------------|------------------------------|
| **Aggregate**    | AUROC, C-index, AUC-PRC      | Calibration intercept + slope, reliability diagram |
| **Subgroup**     | Per-subgroup AUROC            | Per-subgroup reliability diagram |

For decisions (not just predictions), add a fifth element: **net benefit / decision-curve analysis** across plausible thresholds, both aggregate and per-subgroup.

A model that performs well in the top-left quadrant only is a research artefact. A model that performs in all five is a candidate for deployment.

## Discrimination

The standard discrimination metric for binary outcomes is **AUROC**, the area under the receiver operating characteristic curve. Intuition: $\Pr(\text{score}_+ > \text{score}_-)$ for a random positive and a random negative.

- AUROC = 0.5 is no better than chance.
- AUROC = 1.0 is perfect ranking — almost always a sign of label leakage.
- For survival, the analogue is the **concordance index** ($C$-index).

What AUROC misses:

- Calibration. A model that ranks perfectly but is off by 10× on probability is useless for decisions.
- Class imbalance. AUROC overstates performance when the positive rate is low. Use AUC-PRC (precision-recall) for low-prevalence outcomes.
- Subgroup performance. A model with AUROC = 0.85 overall can have AUROC = 0.65 in a subgroup.

## Calibration

A model is *calibrated* if among patients predicted to have probability $p$, the observed rate is $p$.

- **Calibration intercept and slope.** Fit a logistic regression of the binary outcome on the model's log-odds prediction; intercept $\alpha$ measures bias, slope $\beta$ measures overall calibration. Perfect: $\alpha = 0$, $\beta = 1$.
- **Reliability diagram.** Bin patients by predicted probability decile; plot mean predicted vs. observed rate.
- **Brier score.** $\frac{1}{n} \sum (p_i - y_i)^2$. Decomposes into refinement + reliability.

For survival, calibration plots are constructed at fixed horizons (3, 6, 12 months).

When calibration breaks at deployment:

- **Platt scaling.** Logistic regression on the calibration set.
- **Isotonic regression.** Non-parametric monotone mapping.
- **Temperature scaling.** Single-parameter softmax temperature; common for neural-network outputs.

Calibration *must* be checked separately for each subgroup. Aggregate calibration can hide opposite mis-calibrations cancelling out.

## Decision-curve analysis

Decision-curve analysis ([Vickers & Elkin 2006](https://journals.sagepub.com/doi/10.1177/0272989X06295361)) compares the *net benefit* of a model-based policy to baseline strategies across the threshold range that is clinically plausible.

$$
\text{Net benefit at threshold } p_t = \frac{\text{TP}}{n} - \frac{\text{FP}}{n} \cdot \frac{p_t}{1 - p_t}
$$

Plot net benefit vs. $p_t$ for:

- The model-based threshold policy.
- The "treat-all" policy.
- The "treat-none" policy.
- Any reference clinical rule.

The model is useful in the range where its curve is above both reference curves.

## Subgroup evaluation

The minimum stratification for clinical-AI evaluation:

- Sex / gender.
- Age band.
- Race / ethnicity (with appropriate caution about the social construct).
- Primary insurance / socioeconomic proxy.
- Site / scanner / device generation.
- Calendar-time stratum (training era vs. deployment era).

Each stratum gets its own discrimination, calibration, and decision-curve analysis. A model that is well-calibrated overall but badly mis-calibrated for one stratum can cause more harm than help.

## External validation

The single highest-information evaluation step. External validation should differ from the training population on at least one axis:

- Different hospital / health system.
- Different geographic region.
- Different temporal era.
- Different EHR vendor or scanner manufacturer.
- Different population sub-group.

If your model has only been validated on a held-out random split from the same hospital, you have not validated it.

Recommended: *temporal* external validation (next year's data) for the initial deployment decision, plus *site* external validation before scaling beyond the original institution.

## Off-policy evaluation

For decision-support models, the held-out set is also part of the historical policy. Evaluating *the model's policy* requires off-policy techniques:

- **Inverse-propensity weighted (IPW) evaluation.**
- **Doubly-robust off-policy evaluation.**
- **Direct method.**
- **Shadow-mode deployment.** Run the model silently for a period; compare to actual clinician decisions; estimate the value of the model's policy from the contrast.

See [Tasks → Clinical decision support](../tasks/clinical-decision-support.md) for the broader framing.

## Distribution-shift monitoring

A deployed model needs ongoing monitoring:

- **Input drift.** PSI / KL divergence on each input feature.
- **Output drift.** Histogram of predicted scores over time.
- **Outcome drift.** Predicted vs. observed reliability over rolling windows.
- **Subgroup drift.** Per-subgroup reliability over time.

Pre-specify thresholds that trigger re-validation or halt. The PCCP framework ([FDA 2024](https://www.fda.gov/regulatory-information/search-fda-guidance-documents/marketing-submission-recommendations-predetermined-change-control-plan-artificial)) is how the regulatory side handles this.

## Conformal prediction and uncertainty

For risk-prediction models, point estimates without uncertainty are dangerous. Two practical tools:

- **Conformal prediction.** Distribution-free prediction sets with marginal coverage guarantees.
- **Bayesian posterior sampling.** When the model has a usable posterior, propagate uncertainty into the decision step.

Reference: [Angelopoulos & Bates 2023](https://arxiv.org/abs/2107.07511), [Vovk et al. 2005](https://link.springer.com/book/10.1007/978-3-031-06649-8) for the foundational treatment.

## Reporting

The reporting standards in precision medicine:

- **TRIPOD+AI** ([Collins et al. 2024](https://doi.org/10.1136/bmj-2023-078378)) — clinical-prediction models.
- **CLAIM** ([Mongan, Moy, Kahn 2020](https://doi.org/10.1148/ryai.2020200029)) — medical imaging AI.
- **CONSORT-AI** ([Liu et al. 2020](https://www.nature.com/articles/s41591-020-1034-x)) — RCTs of AI interventions.
- **SPIRIT-AI** ([Cruz Rivera et al. 2020](https://www.nature.com/articles/s41591-020-1037-7)) — protocols for AI trials.
- **DECIDE-AI** ([Vasey et al. 2022](https://www.bmj.com/content/377/bmj-2022-070904)) — early clinical evaluation of AI decision support.
- **STARD-AI** ([Sounderajah et al. 2020](https://www.nature.com/articles/s41591-020-1041-y)) — diagnostic-accuracy studies.

A pre-submission checklist that ticks all of TRIPOD+AI is the operational minimum for a deployable model.

## The minimal evaluation checklist

- [ ] Discrimination + calibration on a temporal hold-out.
- [ ] Discrimination + calibration on at least one external site.
- [ ] Per-subgroup metrics for sex, age, race, site.
- [ ] Decision-curve analysis at deployment-relevant thresholds.
- [ ] Failure-case write-up (≥ 3 documented).
- [ ] Sensitivity analysis for unmeasured confounding (causal models only).
- [ ] Drift-monitoring plan with halt criteria.
- [ ] TRIPOD+AI checklist attached.
- [ ] Reproducibility artefacts pinned (commit SHA, container digest, lockfile, model weights hash).

If you can tick all nine, you're ready for a serious external review. If you cannot, the gaps are precisely what an auditor will ask about.

## Exercises

1. **Build a calibration audit pipeline** for a binary risk model: reliability diagram, intercept/slope, Brier score decomposition, by subgroup.
2. **Implement decision-curve analysis** for a published model. At what thresholds is it actually useful?
3. **Pick a published AI-for-medicine paper.** Score it against the TRIPOD+AI checklist. Where are the gaps?

## References

1. Steyerberg EW. *Clinical Prediction Models.* Springer, 2019.
2. Collins GS, Moons KGM, et al. TRIPOD+AI statement. *BMJ.* 2024;385:e078378.
3. Liu X, et al. Reporting guidelines for clinical trial reports for interventions involving artificial intelligence: CONSORT-AI extension. *Nat Med.* 2020;26(9):1364–1374.
4. Cruz Rivera S, et al. Guidelines for clinical trial protocols for interventions involving AI: SPIRIT-AI extension. *Nat Med.* 2020;26(9):1351–1363.
5. Vasey B, et al. Reporting guideline for the early-stage clinical evaluation of decision-support systems driven by AI: DECIDE-AI. *BMJ.* 2022;377:e070904.
6. Sounderajah V, et al. STARD-AI. *Nat Med.* 2020;26(6):807–808.
7. Vickers AJ, Elkin EB. Decision curve analysis. *Med Decis Making.* 2006;26(6):565–574.
8. Angelopoulos AN, Bates S. A gentle introduction to conformal prediction. *arXiv:2107.07511.*

## Where to next

[Case studies](../case-studies/index.md) — three long-form worked examples that put the techniques in this section to use.
