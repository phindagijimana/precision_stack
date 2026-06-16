# F. Synthetic control arms

> *A synthetic control arm uses historical patient data to construct the comparison group that a single-arm trial cannot recruit. Done well, it can shorten trials and reduce the number of patients exposed to placebo. Done badly, it bakes confounding by indication into the regulatory submission.*

## When synthetic controls are reasonable

There are three settings where regulators have signalled openness to external / synthetic control arms:

1. **Rare diseases.** Randomising a placebo arm is ethically and practically infeasible because the disease is rare and standard of care is not the natural comparator.
2. **Catastrophic, untreated disease.** Withholding the experimental therapy would deny a likely-beneficial treatment.
3. **Cell / gene therapies and devices.** Sham-control or no-treatment arms are not ethical or technically feasible.

In each, a real placebo arm is impractical; a *contemporary external control* drawn from registry / EHR / prior-trial data offers a feasible comparator.

The FDA's [framework on RWE for regulatory decision making](https://www.fda.gov/science-research/science-and-research-special-topics/real-world-evidence) and the EMA's [Reflection Paper on the Use of Real-World Data](https://www.ema.europa.eu/en/documents/regulatory-procedural-guidelines/reflection-paper-use-real-world-data-rwd-non-interventional-studies-generate-real-world-evidence_en.pdf) are the starting points.

## The estimation target

A synthetic-control analysis estimates the **counterfactual outcome distribution under the control** that the trial population *would have experienced* had they not received the experimental therapy:

$$
\hat{F}_{Y(0)}(\cdot \mid X = x_i),\quad i \in \text{trial}
$$

The treatment effect is then the contrast between the observed treated outcomes and the predicted control outcomes:

$$
\hat{\tau}_i = Y_i^{\text{treated}} - \hat{Y}_i^{(0)}
$$

The success of the inference depends on whether the *external* control population can stand in for the trial's *would-have-been* control — i.e., whether the no-unmeasured-confounding assumption from [Foundations → Bias and confounding](../foundations/bias-and-confounding.md) holds across the boundary between the trial cohort and the external cohort.

This is a strong assumption. Several methodological structures help.

## Method families

### Propensity-score-based matching

For each trial patient, find one or more external-cohort patients with similar covariates. Match on the estimated propensity score $\Pr(\text{in trial} \mid X)$. Standard methods:

- 1:1 nearest-neighbour matching.
- $k$-NN matching with caliper.
- Optimal matching (network flow).
- Coarsened exact matching for low-dimensional $X$.

Strengths: intuitive, well-tested, transparent. Weaknesses: needs overlapping support; loses external patients outside the trial covariate range; sensitive to which covariates were included.

### Inverse-probability-of-trial-inclusion weighting

Re-weight the external cohort so its covariate distribution mimics the trial's. Avoids hard matching; uses every external patient. Common in real-world-evidence (RWE) submissions.

### Synthetic-control method (SCM) / generalised SCM

From the econometrics literature (Abadie et al.). For each trial patient, construct the counterfactual by a weighted combination of external patients chosen to minimise pre-treatment covariate distance.

Recent generalisations:

- **Synthetic difference-in-differences** (Arkhangelsky et al. 2021).
- **Augmented synthetic control** (Ben-Michael et al. 2021) — combines SCM with outcome modelling for double robustness.
- **Bayesian synthetic control** (Pinkney 2021) — propagates uncertainty natively.

### Target-trial emulation

A discipline rather than a single estimator: treat the external cohort as if it were enrolled in a hypothetical RCT, with explicit eligibility, treatment-strategy definitions, time-zero, and follow-up. The Hernán target-trial framework ([Hernán & Robins 2016](https://academic.oup.com/aje/article/183/8/758/1739035)) is now the regulator-preferred way to talk about how the external cohort was constructed.

### Doubly-robust ML and DML

Combine an outcome model with a trial-inclusion model under cross-fitting. Robust to mis-specification of one or the other. See [Methods → Causal inference](../methods/causal-inference.md).

## Diagnostics — what makes a synthetic control credible

A regulator will ask for *all* of these:

- **Cohort flow diagram** for the external control. Source, eligibility, exclusions, sample size at each step.
- **Time-zero alignment.** What does "start of follow-up" mean for the external patient, and is it analogous to the trial's enrolment moment? Common pitfall: time-zero in the external cohort is the date of diagnosis; in the trial, it is the date of trial entry, which is days–months later. This **immortal-time bias** can inflate the apparent benefit by a factor of 2.
- **Covariate balance.** Standardised mean differences for every adjustment variable, before and after weighting / matching. Common threshold: SMD < 0.1.
- **Positivity.** Density plots of the propensity score, by trial vs. external. Severe non-overlap is a stop sign.
- **Negative-control outcomes.** Outcomes that should not be affected by the experimental therapy — if a sham association emerges, hidden confounding is at play.
- **Sensitivity analysis.** E-value, Rosenbaum bounds, or a formal tipping-point analysis. How strong an unmeasured confounder would need to be to overturn the conclusion.
- **Pre-specification.** The full SAP — including the external cohort construction — committed before any unblinded data.

## Equity considerations

The external cohort almost always represents a different time period than the trial, often a different geography, often a different care pattern. The CATE estimated against an external control is conditional on the external control's care delivery; whether that translates to the deployment population is an additional inferential step.

## A worked example — single-arm oncology trial with EHR control

A phase-II trial of a novel CDK inhibitor in a rare sarcoma subtype enrols 60 treated patients. No randomised control is feasible — the subtype is too rare. An external cohort of 600 patients with the same subtype is assembled from a multi-institution registry.

Steps:

1. Define the target trial (Hernán-style). Eligibility, treatment strategies, time-zero.
2. Apply the same eligibility to the registry; restrict to time-zero analogue.
3. Estimate propensity-of-trial-inclusion model on the union; check overlap.
4. Construct an IPTW-weighted external cohort; report SMDs.
5. Estimate hazard ratio for overall survival under the propensity-weighted analysis; sensitivity via E-value.
6. Pre-specify a negative-control outcome (e.g. non-cancer mortality) and report it.

The submission package includes a fully reproducible analysis pipeline, the original SAP, and a transparent statement of how unmeasured confounding could overturn the conclusion.

## Common failures

- **Mis-aligned time-zero** (immortal-time bias) — the biggest single source of false positives in external-control studies.
- **Different ascertainment of the outcome** in trial vs. external — the trial has scheduled follow-up; the registry has clinician-driven follow-up.
- **Different standard-of-care era** — the external control got historical chemotherapy; the trial control would have got modern combination therapy.
- **Mis-specified propensity model** — too few covariates, no interactions, no checks of overlap.
- **No pre-specification** — the analysis is reverse-engineered after seeing the results.

## Where this fits regulatory practice

- **FDA** — the [21st Century Cures Act Section 3022](https://www.fda.gov/science-research/science-and-research-special-topics/real-world-evidence) and subsequent guidance documents recognise RWE for "non-primary" endpoints, supplemental indications, and rare-disease primary endpoints. The 2023 guidance "Considerations for the Use of Real-World Data and Real-World Evidence to Support Regulatory Decision-Making" is the operational reference.
- **EMA** — DARWIN EU programme, Reflection Paper on RWD.
- **MHRA / Health Canada / PMDA** — converging frameworks.

A single-arm trial with an external control will not, by itself, satisfy a high-rigour primary endpoint for an approved drug in a common disease. It can support an indication expansion, a rare-disease primary analysis, or an early signal-finding study.

## Exercises

1. **Pick a public observational dataset** (SEER, Flatiron, or a registry) and emulate a target trial against it. Document every choice.
2. **Compute covariate balance** before and after IPTW for the emulated trial.
3. **Sensitivity analysis.** For your estimated treatment effect, compute the E-value and write the sentence you would include in the regulator submission about residual confounding.

## References

1. Hernán MA, Robins JM. Using big data to emulate a target trial when a randomized trial is not available. *Am J Epidemiol.* 2016;183(8):758–764.
2. Hatswell AJ, Freemantle N, Baio G. The effects of model misspecification in single-arm trials with external controls. *Stat Med.* 2020;39(14):1936–1958.
3. Abadie A, Diamond A, Hainmueller J. Synthetic control methods for comparative case studies. *J Am Stat Assoc.* 2010;105(490):493–505.
4. Arkhangelsky D, et al. Synthetic difference-in-differences. *Am Econ Rev.* 2021;111(12):4088–4118.
5. Ben-Michael E, Feller A, Rothstein J. The augmented synthetic control method. *J Am Stat Assoc.* 2021;116(536):1789–1803.
6. FDA. *Considerations for the Use of Real-World Data and Real-World Evidence to Support Regulatory Decision-Making for Drug and Biological Products.* Final guidance, 2023.

## Where to next

[G. Adaptive clinical trials](adaptive-trials.md) — how to design trials that learn from interim data, of which the synthetic-control concept is one corner.
