# B. Treatment-response prediction

> *Which treatment is most likely to help this patient?* This is the single most clinically valuable question in precision medicine — and the one most easily answered incorrectly.

## What we're estimating

The target object is the **conditional average treatment effect** (CATE):

$$
\tau(x) = \mathbb{E}\bigl[Y(1) - Y(0) \mid X = x\bigr]
$$

where $Y(1)$ and $Y(0)$ are the patient's potential outcomes under treatment and control, and $X$ is the patient's pre-treatment features. The CATE is the contrast we *would* observe if we could randomise this patient. The fundamental problem of causal inference is that we never observe both potentials at the same time.

A clinically useful CATE estimator must:

1. **Identify** $\tau(x)$ from the data (assumption check, not just a model).
2. **Estimate** $\tau(x)$ with controlled bias and variance.
3. **Validate** that the estimated $\tau(x)$ corresponds to the *real* benefit when used to make decisions.
4. **Calibrate** decision-relevance — the absolute risks under each arm matter, not just the contrast.

Skipping any one of these is how the field produces models that improve a confusion-matrix metric and harm patients in practice.

## Identification — what assumptions you need

The CATE is identified from observational data under three assumptions:

- **Consistency.** $Y = Y(A)$ — the observed outcome under the observed treatment equals the potential outcome under that treatment. This is broken by hidden versions of the treatment (drug A from two different manufacturers).
- **Positivity.** $0 < \Pr(A = 1 \mid X = x) < 1$ for every $x$ in the population. There is some chance both groups are observed at every $x$. Broken by deterministic clinical-decision rules.
- **No unmeasured confounding (ignorability / exchangeability).** $\{Y(0), Y(1)\} \perp\!\!\!\perp A \mid X$. The treatment is as good as random within strata of $X$. This is the one almost always under threat — see [Foundations → Bias and confounding](../foundations/bias-and-confounding.md).

Under these, $\tau(x) = \mathbb{E}[Y \mid X = x, A = 1] - \mathbb{E}[Y \mid X = x, A = 0]$ — two regressions and a subtraction. The rest of the chapter is about doing that *well* when assumptions are partially violated.

## Method families

### Meta-learners

Reduce CATE estimation to standard regression / classification:

- **T-learner.** Two outcome models, one per arm. Subtract. Simple; variance dominates when arms are small.
- **S-learner.** One outcome model with treatment as a feature. Subtract two predictions per patient. Tends to regularise $\tau$ towards zero.
- **X-learner.** Two outcome models *plus* two imputed-effect models, combined via a propensity-weighted average. Good when arms are imbalanced.
- **R-learner.** Robinson-style decomposition. Solves a residualised regression that delivers oracle convergence rates under regularity.
- **DR-learner.** Doubly-robust. Combines an outcome model with a propensity model; consistent if either is correctly specified.

Reference: [Künzel et al. 2019](https://www.pnas.org/doi/10.1073/pnas.1804597116) for T/S/X; [Nie & Wager 2021](https://academic.oup.com/biomet/article/108/2/299/5911092) for R; [Kennedy 2020](https://arxiv.org/abs/2004.14497) for DR.

### Causal forests

Tree ensembles with honest splits chosen to maximise treatment-effect heterogeneity rather than outcome variance. Local CATE estimates at every leaf with valid confidence intervals via the generalised random-forest inference framework.

Reference: [Athey, Tibshirani & Wager 2019](https://www.jstor.org/stable/26581900).

### Doubly-robust machine learning (DML)

A family ([Chernozhukov et al. 2018](https://academic.oup.com/ectj/article/21/1/C1/5056401)) that uses ML for nuisance functions (outcome model, propensity model) and a downstream linear / parametric step for the target estimand. Cross-fitting prevents the bias-from-overfitting issue. The DML CATE is the modern default for medium-sized observational cohorts.

### G-methods for time-varying treatments

When the patient's treatment can change over time and the next treatment depends on intermediate outcomes (almost always, in chronic disease), standard methods fail because of time-varying confounding. The remedy is **G-computation**, **G-estimation**, or **marginal structural models** fit with IPTW.

Reference: [Hernán & Robins, *Causal Inference: What If*](https://www.hsph.harvard.edu/miguel-hernan/causal-inference-book/), chs. 11–14.

### Sequence-aware deep learning

For complex longitudinal regimes:

- **Counterfactual recurrent networks (CRN)** — Bica et al., ICLR 2020.
- **G-Net** — Li et al., NeurIPS 2021.
- **TE-CDE** — controlled-differential-equation parameterisation of counterfactual trajectories.

These methods are the right tool when the trajectory model from [Getting started → Patient trajectory](../getting-started/patient-trajectory.md) is the right framing of the clinical problem.

## Evaluation — the part most papers skip

Treatment-response models cannot be evaluated by held-out AUROC. The CATE has no observable ground truth at the individual level. Three principled ways to evaluate:

### 1. On RCT data

- The held-out set is randomised, so subgroup-level $\tau$ is identified by simple difference-in-means.
- Stratify patients by predicted-benefit decile, compute the observed $\tau$ in each decile, compare to the predicted $\tau$. This is the **predicted-vs-observed benefit curve**, the central CATE-validation plot.
- Decision-curve analysis under each candidate policy.

### 2. On observational data

- Same procedure, but the observed $\tau$ in each decile is now itself a causal estimate (use IPTW / DR within each decile).
- Falsification tests using negative controls (Lipsitch et al. 2010).
- Sensitivity analysis for unmeasured confounding (Rosenbaum bounds, E-value).

### 3. Policy-value estimation

If the model is going to inform decisions, estimate the value of *the policy it implies*:

$$
V(\pi) = \mathbb{E}_{X, Y \sim P}\Bigl[\mathbb{E}\bigl[Y \mid X, do(A = \pi(X))\bigr]\Bigr]
$$

For an RCT, doubly-robust off-policy evaluation gives an unbiased estimate. For observational data, see the policy-learning literature ([Athey & Wager 2021](https://onlinelibrary.wiley.com/doi/10.3982/ECTA15732)).

## Calibration and decision relevance

A model that estimates $\tau(x) = 0.10$ for one patient and $\tau(x) = 0.30$ for another is doing precision medicine *only if* those numbers are absolute risks (or close approximations). A ranking is not enough; calibration is what makes a CATE actionable.

- **Calibration intercept and slope** on the predicted-vs-observed benefit plot.
- **Decision-curve analysis** (Vickers & Elkin 2006) — compare net benefit across plausible decision thresholds.
- **Subgroup calibration** — every subgroup defined by sex, age, race, site should have its own calibration plot.

## A worked example — depression pharmacotherapy

The clinical question: among adults with major depressive disorder presenting for first-line treatment, which SSRI is most likely to achieve remission at 8 weeks?

1. **Data.** STAR\*D + an institutional cohort. EHR, baseline PHQ-9, demographics, comorbidities, prior trials.
2. **Action set.** Sertraline, fluoxetine, escitalopram. Treatment-as-usual = "physician's choice".
3. **Outcome.** PHQ-9 ≤ 5 at 8 weeks (remission); secondary, any adverse event grade ≥ 2.
4. **Method.** DR-learner CATE with cross-fitting. Multi-arm.
5. **Validation.** Held-out site; predicted-vs-observed PHQ-9 reduction by decile.
6. **Decision support.** Recommend the SSRI with the highest predicted remission probability, *if* the predicted gap > 5% absolute. Otherwise defer to clinician preference.

A real implementation would expose this through a clinician-facing UI; see [Tasks → Clinical decision support](clinical-decision-support.md).

## Common failures

- **Comparing predicted ATE to observed ATE.** It looks great because the model is regressing to the population mean. The CATE is what matters.
- **Treating an off-the-shelf classifier's SHAP values as a CATE explanation.** SHAP explains the *prediction*, not the *causal effect*.
- **Validating on the same observational data the model was trained on.** Even a held-out split inherits the confounding.
- **Skipping positivity checks.** If your model is recommending a treatment for a subgroup that never received it in the data, the recommendation is extrapolation, not estimation.

## Exercises

1. **Replicate Künzel et al.'s synthetic experiment.** Implement T-, S-, and X-learners. Reproduce the bias-variance trade-off.
2. **CATE on an RCT.** Take a published trial dataset (e.g. SPRINT). Build a CATE model for systolic-BP-lowering effect on cardiovascular outcomes; produce the predicted-vs-observed benefit plot.
3. **Sensitivity analysis.** For an observational CATE you have estimated, compute the E-value and a Rosenbaum bound. How robust is your conclusion to unmeasured confounding?

## References

1. Künzel SR, Sekhon JS, Bickel PJ, Yu B. Metalearners for estimating heterogeneous treatment effects using machine learning. *PNAS.* 2019;116(10):4156–4165.
2. Athey S, Tibshirani J, Wager S. Generalized random forests. *Ann Stat.* 2019;47(2):1148–1178.
3. Nie X, Wager S. Quasi-oracle estimation of heterogeneous treatment effects. *Biometrika.* 2021;108(2):299–319.
4. Chernozhukov V, et al. Double/debiased machine learning. *Econometrics J.* 2018;21(1):C1–C68.
5. Kennedy EH. Towards optimal doubly robust estimation of heterogeneous causal effects. *arXiv:2004.14497.*
6. Bica I, et al. Estimating counterfactual treatment outcomes over time. *ICLR* 2020.
7. Vickers AJ, Elkin EB. Decision curve analysis. *Med Decis Making.* 2006;26(6):565–574.
8. Hernán MA, Robins JM. *Causal Inference: What If.* Chapman & Hall/CRC, 2020.

## Where to next

[C. Risk prediction](risk-prediction.md) — the prognostic counterpart of this chapter, with its own evaluation traps.
