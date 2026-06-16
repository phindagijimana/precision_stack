# What AI can and cannot answer

> *Most failures of clinical AI are not failures of modelling. They are failures of asking the right question of the data you have.*

This chapter draws the line between three things that look similar and behave very differently:

1. **Prediction** — what will happen.
2. **Explanation** — why it will happen.
3. **Decision** — what we should do about it.

A clinically useful model has to be honest about which of these it is doing. Most published "AI for medicine" papers conflate them, and that is the single largest source of subsequent disappointment.

## Prediction

Prediction is the question "what is $\Pr(Y \mid X)$?" for some outcome $Y$ and observed features $X$. It is an *associational* question — it does not require causal assumptions, only that the joint distribution of $(X, Y)$ in the deployment population matches the training distribution.

What AI is genuinely good at:

- Risk scores — readmission, sepsis onset, in-hospital deterioration. *Provided the deployment distribution matches training.*
- Imaging-based diagnosis when the label is reliable. *Lung nodule classification, diabetic retinopathy grading, fracture detection.*
- Time-to-event prediction — survival, time to seizure recurrence. See [Methods → Survival analysis](../methods/survival-analysis.md).

Where prediction breaks:

- **Distribution shift.** New scanner, new hospital, new patient mix. The model's training distribution no longer holds. AUROC drops by 5–15 points routinely.
- **Label leakage.** The "label" embeds information the model would not have at deployment time. Famously: a sepsis model that learns "if a vasopressor was started, sepsis was diagnosed", which is true on training data and useless at the bedside.
- **Selection bias.** The training cohort is the patients who made it to your dataset. The deployment cohort is broader.

## Explanation

Explanation is the question "why $Y$?" — *which variables in $X$ caused $Y$?* This is a *causal* question. It cannot be answered by training a black-box model and reading SHAP values off it, regardless of how seductive the picture looks.

A SHAP plot ranks features by how much they contribute to the model's *prediction*. That is associational. A feature can rank high because:

1. It causes the outcome.
2. It is caused by the outcome (reverse causation).
3. It is caused by something that causes the outcome (confounding).
4. It is correlated with another feature that causes the outcome.

The model has no way to distinguish these. SHAP is a tool for understanding the *model*, not the *world*.

If you need an explanation — *which variable should I intervene on to change $Y$?* — you need causal inference. See [Methods → Causal inference](../methods/causal-inference.md). The minimum machinery is a graphical model of the data-generating process (a DAG), an identifiability check (back-door / front-door / instrumental variable), and a sensitivity analysis for unmeasured confounding.

## Decision

Decision is the question "what should we do?" — *given a candidate intervention $A$, what is $\mathbb{E}[Y \mid X, do(A)]$ for each candidate?* You will recognise the $do$-operator from Judea Pearl's causal hierarchy; it is the formal notation for "intervene and set $A$ to this value", as opposed to "observe $A$ at this value".

Decision is the precision-medicine ask. It requires *both* a prediction model *and* a causal model. Practically:

| Object | Symbol | What you need to estimate it |
|---|---|---|
| Prediction | $\Pr(Y \mid X)$ | Any well-calibrated supervised model on representative data. |
| Counterfactual prediction | $\Pr(Y \mid X, do(A=a))$ | An *identification* strategy (RCT, IPTW, doubly-robust, G-computation) plus a model. |
| Decision | $\arg\max_a \mathbb{E}[U(Y) \mid X, do(A=a)]$ | The above, plus a utility function $U$ that encodes how much each outcome state matters. |

Three things you have to do at decision time that you don't have to do at prediction time:

1. **Define the action set.** Treatment-as-usual, drug A, drug B, surgery, no-intervention. You cannot recommend an action that is not in the set.
2. **Define the utility.** Seizure-freedom plus minus weight on side-effects. Five-year survival weighted by quality-of-life. Without a utility, "the best action" is undefined.
3. **Account for the policy.** The model is now part of a feedback loop. Its recommendation will change which patients receive which treatment, which will change the data the next version of the model trains on. See [Methods → Evaluation](../methods/evaluation.md) for off-policy evaluation.

## The three-question test

Before writing any code, answer these three questions out loud about your model:

1. *Is the input something the clinician will have at the time the model is supposed to be useful?* If the model needs labs that won't be back for 6 hours, it is not a triage model.
2. *Is the label something we actually wanted to predict, or a proxy that happens to be in the data?* "ICU admission" is a proxy for severity; it is also a proxy for "the ICU had a bed".
3. *Is the model going to inform a decision, or just sit on a dashboard?* If it informs a decision, you have moved from prediction to decision-making, and the rest of the handbook applies.

If question 3 is "decision", read [Methods → Causal inference](../methods/causal-inference.md) before going further.

## What AI cannot do

- **Identify a causal effect that the design does not identify.** If two treatments were never randomised and the indication is unmeasured, no amount of neural-network capacity will recover the CATE.
- **Compensate for missing modalities.** A model with no access to a key biomarker cannot infer that biomarker reliably from the modalities it does see. It can predict it on average, with all the standard imputation caveats.
- **Promise generalisation to populations it has never seen.** External validation is not a nice-to-have. See [Methods → Evaluation](../methods/evaluation.md).
- **Replace clinician judgement at the point of care.** The model is a *decision support* tool, not a *decision* tool. See [Tasks → Clinical decision support](../tasks/clinical-decision-support.md).

## A summary you can put on a sticky note

> *Prediction tells you what is likely. Explanation tells you why. Decision tells you what to do. Don't ship the wrong one.*

## Where to next

You're now ready for [Foundations](../foundations/index.md). Start with [Patient heterogeneity](../foundations/patient-heterogeneity.md), which spells out the axes the trajectory model lives on.
