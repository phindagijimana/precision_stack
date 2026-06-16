# Causal inference

> *The mathematics of "what would have happened if".* This chapter is the technical core that the [tasks chapters](../tasks/index.md) on treatment-response, synthetic controls, and decision support all draw from.

## The potential-outcomes framework

For each patient $i$ and each candidate treatment $a \in \mathcal{A}$, define a *potential outcome* $Y_i(a)$ — the outcome the patient would have under treatment $a$. The actual observed outcome is $Y_i = Y_i(A_i)$, where $A_i$ is the actually-received treatment.

The **individual treatment effect** (ITE) for binary $\mathcal{A} = \{0, 1\}$ is

$$
\delta_i = Y_i(1) - Y_i(0)
$$

— the contrast we would want, but can never measure. This is the **fundamental problem of causal inference**: we observe one of $\{Y_i(0), Y_i(1)\}$ per patient, never both.

Two coarser quantities are identifiable under assumptions:

- The **average treatment effect** (ATE):
$$ \tau = \mathbb{E}[Y(1) - Y(0)]. $$
- The **conditional ATE** (CATE):
$$ \tau(x) = \mathbb{E}[Y(1) - Y(0) \mid X = x]. $$

The CATE is the precision-medicine target.

## Identification

Identification is the question *can we write our target estimand as a functional of the observable data distribution?* For the CATE, the canonical sufficient conditions are:

- **Consistency.** $Y_i = Y_i(A_i)$.
- **Positivity.** $0 < e(x) := \Pr(A = 1 \mid X = x) < 1$ for all $x$ in the support.
- **No unmeasured confounding (NUC) / ignorability.** $\{Y(0), Y(1)\} \perp\!\!\!\perp A \mid X$.

Under these, the CATE is

$$
\tau(x) = \mu_1(x) - \mu_0(x),\quad \mu_a(x) := \mathbb{E}[Y \mid X = x, A = a].
$$

The estimator is two regressions and a subtraction. The interesting part is when one or more of the assumptions is partially violated — almost always, with observational data.

## DAGs and the back-door criterion

A causal DAG encodes the data-generating process. The back-door criterion ([Pearl 1995](https://www.jstor.org/stable/2335942)) gives a graph-theoretic test for whether a set $Z$ of covariates suffices for identification: $Z$ blocks every back-door path from $A$ to $Y$ and contains no descendants of $A$.

A practical workflow:

1. Draw the DAG of your data-generating process.
2. Identify all paths from $A$ to $Y$.
3. Determine the minimal sufficient adjustment set $Z$ that closes all back-door paths.
4. Compare $Z$ to your observed covariate set $X$. If $Z \subseteq X$, you can identify; otherwise, the unmeasured difference is a residual-confounding risk.

Tooling: [DAGitty](http://www.dagitty.net), `dagitty` R package, `pgmpy` in Python.

## Propensity-score methods

The propensity score is $e(x) = \Pr(A = 1 \mid X = x)$. The classical result ([Rosenbaum & Rubin 1983](https://academic.oup.com/biomet/article-abstract/70/1/41/240879)) is that, under NUC, conditioning on $e(x)$ is equivalent to conditioning on $X$ — useful because $e(x)$ is one-dimensional regardless of the dimension of $X$.

Four standard uses:

- **Matching.** Pair treated with control by nearest neighbour on $e(x)$.
- **Stratification.** Partition $e(x)$ into strata; compute the within-stratum ATE; pool.
- **Weighting (IPTW).** Re-weight each unit by $1 / e(x)$ if treated and $1 / (1 - e(x))$ if control. The reweighted sample emulates a randomised trial.
- **Regression adjustment.** Include $e(x)$ as a covariate in an outcome model.

The IPTW estimator of the ATE is

$$
\hat{\tau}_{\text{IPTW}} = \frac{1}{n} \sum_{i=1}^n \left[ \frac{A_i Y_i}{\hat{e}(X_i)} - \frac{(1 - A_i) Y_i}{1 - \hat{e}(X_i)} \right]
$$

Variance grows with extreme propensities; trimming or stabilised weights ($w_i = A_i \bar{p} / \hat{e}(X_i) + (1-A_i)(1-\bar{p})/(1-\hat{e}(X_i))$) are the standard mitigations.

## Doubly-robust estimation

Combine an outcome model $\hat{\mu}_a(x)$ with a propensity model $\hat{e}(x)$:

$$
\hat{\tau}_{\text{DR}} = \frac{1}{n} \sum_{i=1}^n \left\{ \hat{\mu}_1(X_i) - \hat{\mu}_0(X_i) + \frac{A_i (Y_i - \hat{\mu}_1(X_i))}{\hat{e}(X_i)} - \frac{(1-A_i)(Y_i - \hat{\mu}_0(X_i))}{1 - \hat{e}(X_i)} \right\}
$$

Consistent for $\tau$ if *either* $\hat{\mu}_a$ or $\hat{e}$ is correctly specified. Augmented IPW (AIPW) and TMLE are concrete instantiations.

## Double / debiased machine learning (DML)

The DML framework ([Chernozhukov et al. 2018](https://academic.oup.com/ectj/article/21/1/C1/5056401)) generalises DR estimation to ML-based nuisance functions. The key technical move is **cross-fitting**: split the sample into folds, train nuisance functions on each fold-leave-out, and use the held-out fold to compute the causal-effect estimator. This breaks the dependence between the ML model's overfitting and the causal estimate, restoring $\sqrt{n}$ convergence.

The DML CATE is implemented in EconML, CausalML, DoWhy. Reference workflow:

```python
from econml.dml import LinearDML
from sklearn.ensemble import GradientBoostingRegressor, GradientBoostingClassifier

dml = LinearDML(
    model_y=GradientBoostingRegressor(),
    model_t=GradientBoostingClassifier(),
    discrete_treatment=True,
    cv=5,
)
dml.fit(Y=y, T=t, X=X, W=W)  # X: effect modifiers, W: confounders only
tau_hat = dml.effect(X)
ci = dml.effect_interval(X)
```

## Instrumental variables

When NUC is implausible and an instrument $Z$ exists — a variable that affects $A$ but is independent of unmeasured confounders and of $Y$ except through $A$ — IV estimation recovers the local average treatment effect for *compliers* (the LATE).

In clinical settings, candidate instruments include:

- **Clinician prescribing preference.** Differences in prescribing rate across physicians treated as exogenous variation.
- **Geographic / time variation.** Policy changes, formulary changes, randomisation in different jurisdictions.
- **Randomised encouragement.** Random nudges that affect uptake.

Two-stage least squares (2SLS), control functions, and the modern machine-learning IV estimators (DeepIV, IVML) are the practical machinery. References: [Imbens & Angrist 1994](https://www.jstor.org/stable/2951620); [Hartford et al. ICML 2017](http://proceedings.mlr.press/v70/hartford17a.html).

## Time-varying treatments — the G-methods

When the treatment evolves with the patient's state and intermediate outcomes (chronic disease, ICU care), standard adjustment is biased.

- **G-computation** — model the conditional outcome distribution, simulate counterfactual trajectories under each policy.
- **G-estimation of structural nested models** — direct parameterisation of the per-period treatment effect.
- **Marginal structural models with IPTW** — re-weight at each time step by inverse probability of the observed treatment history.

Reference: [Hernán & Robins, *Causal Inference: What If*](https://www.hsph.harvard.edu/miguel-hernan/causal-inference-book/), chs. 11–14.

## Sensitivity analysis

Even the best-adjusted CATE rests on the NUC assumption, which is untestable. Sensitivity analysis quantifies *how strong* an unmeasured confounder would have to be to overturn a conclusion.

- **E-value** ([VanderWeele & Ding 2017](https://www.acpjournals.org/doi/10.7326/M16-2607)) — the minimum strength of association on the risk-ratio scale that an unmeasured confounder would need with both treatment and outcome to fully explain away the estimated effect.
- **Rosenbaum bounds** — sensitivity analysis for matched designs.
- **Tipping-point analyses** — pre-specify a confounder strength and report which conclusions reverse.

A pre-deployment write-up that does not include a sensitivity analysis is not a defensible causal claim.

## Negative-control outcomes and falsification tests

A *negative-control outcome* is an outcome the treatment cannot plausibly affect. If the model recovers a non-zero treatment effect on this outcome, residual confounding is at play.

A *negative-control exposure* is an exposure that cannot plausibly affect the outcome. Same logic, inverted.

For both, the standard discipline ([Lipsitch et al. 2010](https://www.acpjournals.org/doi/10.7326/0003-4819-152-3-201002020-00007)) makes residual confounding visible.

## A worked example — propensity-weighted CATE on observational data

The goal: estimate the heterogeneous effect of metformin vs. sulfonylurea on cardiovascular events in a real-world diabetes registry.

1. **DAG.** Treatment depends on age, baseline HbA1c, renal function, prior CV events. Outcome depends on the same plus randomness. Unmeasured: socioeconomic factors, exercise.
2. **Eligibility.** Type-2 diabetes patients initiating monotherapy.
3. **Propensity model.** Gradient-boosted classifier; check overlap, trim extreme propensities.
4. **Outcome model.** Gradient-boosted regressor for time-to-event.
5. **CATE.** DR-learner CATE with cross-fitting.
6. **Validation.** Predicted-vs-observed benefit curve on a held-out site.
7. **Sensitivity.** E-value for the conclusion at each decile of predicted benefit.
8. **Reporting.** TRIPOD+AI + STROBE + CONSORT-AI alignment.

## Exercises

1. **Implement IPTW from scratch.** On a simulated dataset with known $\tau$, recover $\hat{\tau}$ from IPTW and an outcome model. Verify the double-robustness property by mis-specifying each in turn.
2. **DAG drawing.** Take a published observational paper, draw the DAG implied by the analysis, identify the back-door set, check which adjustment variables are used.
3. **Sensitivity.** Compute the E-value for a published treatment-effect estimate of your choice. Is the conclusion robust?

## References

1. Hernán MA, Robins JM. *Causal Inference: What If.* Chapman & Hall/CRC, 2020. ([free PDF](https://www.hsph.harvard.edu/miguel-hernan/causal-inference-book/))
2. Pearl J. *Causality: Models, Reasoning, and Inference.* Cambridge UP, 2009.
3. Rosenbaum PR, Rubin DB. The central role of the propensity score in observational studies for causal effects. *Biometrika.* 1983;70(1):41–55.
4. Chernozhukov V, et al. Double/debiased machine learning. *Econometrics J.* 2018;21(1):C1–C68.
5. Künzel SR, et al. Metalearners for estimating heterogeneous treatment effects using machine learning. *PNAS.* 2019;116(10):4156–4165.
6. Athey S, Wager S. Policy learning with observational data. *Econometrica.* 2021;89(1):133–161.
7. VanderWeele TJ, Ding P. Sensitivity analysis in observational research: introducing the E-value. *Ann Intern Med.* 2017;167(4):268–274.
8. Imbens GW, Angrist JD. Identification and estimation of local average treatment effects. *Econometrica.* 1994;62(2):467–475.
9. Lipsitch M, Tchetgen Tchetgen E, Cohen T. Negative controls. *Epidemiology.* 2010;21(3):383–388.

## Where to next

[Survival analysis](survival-analysis.md) — the time-to-event machinery that risk prediction, synthetic controls, and many CATE problems rest on.
