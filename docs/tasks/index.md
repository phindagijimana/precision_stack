# Tasks

> *The seven workhorse problem types in precision medicine. Each chapter starts in plain language and ends with the statistical formulation, canonical methods, evaluation rubric, and a worked example.*

| Task | Statistical object | Canonical method family |
|---|---|---|
| **[A. Patient stratification](stratification.md)** | Partition $\mathcal{X}$ into subgroups $\{G_k\}$ | Unsupervised clustering, supervised subtype prediction, mixture models |
| **[B. Treatment-response prediction](treatment-response.md)** | CATE $\tau(x)$ | Meta-learners (T-, S-, X-, R-), causal forests, doubly-robust ML |
| **[C. Risk prediction](risk-prediction.md)** | Prognostic $\Pr(Y \mid X)$ or hazard $\lambda(t \mid X)$ | Logistic regression, gradient boosting, Cox / DeepSurv, LSTM / Transformer |
| **[D. Clinical decision support](clinical-decision-support.md)** | Recommended action $\arg\max_a \mathbb{E}[U(Y) \mid X, do(A=a)]$ | Policy learning, off-policy evaluation, contextual bandits |
| **[E. Trial matching](trial-matching.md)** | $\mathbb{1}\{x \in \mathcal{E}_t\}$ for trial $t$ | Rule-based + NLP; LLM-augmented eligibility parsing |
| **[F. Synthetic control arms](synthetic-controls.md)** | Counterfactual outcome distribution under control | Propensity score, target-trial emulation, synthetic-control method |
| **[G. Adaptive clinical trials](adaptive-trials.md)** | Posterior over treatment effects with sequential updating | Bayesian adaptive randomisation, response-adaptive designs, platform-trial machinery |

Each chapter is self-contained but assumes you have read [Foundations → Bias and confounding](../foundations/bias-and-confounding.md) and [Getting started → What AI can and cannot answer](../getting-started/what-ai-can-answer.md). The deeper methodological content is broken out in [Methods](../methods/index.md); we link forward and back where appropriate.

A reader can pick any chapter and start. A reader who wants the full picture should read them in order — each task builds on patterns from the previous.
