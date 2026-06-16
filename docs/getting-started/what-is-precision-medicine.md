# What precision medicine actually is

> *Precision medicine means choosing the right treatment for the right patient at the right time.* That is the slogan. The rest of this chapter unpacks what the slogan actually commits you to.

## The slogan, plain-language version

For most of the 20th century, clinical research answered one question:

> *Does drug X work, on average, in patients with disease Y?*

That question is answered with a randomised controlled trial (RCT). You enrol a representative sample, randomise to drug or control, measure the outcome at a fixed time point, and report a single number — the **average treatment effect** (ATE). Clinical practice then offers drug X to every patient with disease Y.

This is enormously useful. It is also enormously lossy. Two patients with the same diagnosis are almost never the same patient. They can differ in:

- Genetics — single nucleotide polymorphisms, copy-number variants, rare variants.
- Demographics — age, sex, ancestry, gender, sex-as-biological-variable.
- Disease subtype — the same ICD-10 code can hide three biologically different diseases.
- Imaging patterns — lesion location, lesion burden, atrophy pattern.
- Lab values — kidney function, liver function, inflammatory markers.
- Medications — what they are taking now and what they have taken before.
- Lifestyle — diet, activity, smoking, alcohol, sleep.
- Environment — air quality, occupational exposures, latitude.
- Comorbidities — what other diseases they carry.
- Immune profile — HLA type, autoantibody status, prior infections.
- Treatment history — what worked, what failed, what was discontinued for side effects.

The ATE averages over all of these. **Precision medicine is the discipline of asking the within-patient question instead** — what is most likely to work for *this* patient, given everything we know about them, *now*?

## Why "average" is not enough

A drug that helps 60% of patients and harms 10% has a positive average effect. If you can tell in advance which group is which, you can offer the drug to the 60% and avoid harming the 10%. The ATE was the same — the *decision* is now individualised.

This re-frames the goal of clinical AI. Instead of estimating

$$
\tau = \mathbb{E}[Y(1) - Y(0)]
$$

(the average treatment effect), precision medicine wants to estimate

$$
\tau(x) = \mathbb{E}[Y(1) - Y(0) \mid X = x]
$$

(the **conditional average treatment effect**, or CATE — the expected benefit *for the kind of patient described by features $x$*). The CATE is the central object that every chapter of *Tasks* and *Methods* in this handbook is trying to estimate, validate, or act on.

Two important nuances:

- **Subgroup ≠ individual.** Even a CATE estimated on a richly described $x$ is still an average over patients sharing $x$. The fundamental problem of causal inference — that we never observe the same person under both treatments — never disappears.
- **Heterogeneity exists at every scale.** Sometimes you find a discrete subgroup (e.g. EGFR-mutated NSCLC patients respond to gefitinib while wild-type do not). Sometimes you find a continuous gradient (the CATE varies smoothly with kidney function). Sometimes both. The right model class depends on which kind of heterogeneity is present — see [Methods → Causal inference](../methods/causal-inference.md).

## Where AI fits

The promise of AI in precision medicine is the ability to **integrate** the modalities listed above into a single decision-support output. Three concrete contributions:

1. **High-dimensional representation.** Imaging, genomics, and EHR free-text don't fit into a logistic regression with 12 covariates. Neural networks and gradient-boosted trees do the dimensionality reduction that lets a downstream CATE estimator work.
2. **Pattern recognition across modalities.** A radiologist sees MRI; a geneticist sees the variant calls. An AI model can score them jointly and surface interactions that a single specialist cannot see.
3. **Scale.** Trial-matching, cohort discovery, and pharmacovigilance all involve scanning millions of records. NLP and structured-data models do this orders of magnitude faster than manual chart review.

What AI *cannot* do — and the parts of this handbook that exist to remind you of that:

- It cannot manufacture causal information that is not in the data. See [Methods → Causal inference](../methods/causal-inference.md).
- It cannot avoid the fundamental problem of causal inference. The CATE is identified only under assumptions (consistency, no-unmeasured-confounding, positivity). See [Foundations → Bias and confounding](../foundations/bias-and-confounding.md).
- It cannot replace clinician judgement. See [Tasks → Clinical decision support](../tasks/clinical-decision-support.md) for *why*.

## A first vocabulary checkpoint

Before reading further, you should be able to name in one sentence each:

| Term | One-sentence definition |
|---|---|
| ATE | Average treatment effect across a population. |
| CATE | ATE conditional on patient covariates $x$. |
| ITE | Individual treatment effect — the (unobservable) contrast for a single patient. |
| Subgroup analysis | An ATE estimated on a pre-specified or post-hoc patient slice. |
| Stratification | Dividing patients into groups that are more homogeneous in mechanism or response. |
| Precision medicine | Using all available patient information to choose between candidate treatments at decision time. |
| Personalised medicine | A largely synonymous term, with slightly more emphasis on individual-level (rather than subgroup-level) tailoring. |
| Stratified medicine | The narrower idea that the population is divided into a small number of biologically distinct groups. |

If any of those is fuzzy, the [Glossary](../glossary.md) has fuller definitions and citations.

## Why this matters in practice

The shift from "what usually works" to "what is most likely to work for this specific patient" is not a small refinement. It changes:

- **What trials you run.** Adaptive designs, basket trials, enrichment designs, platform trials — all are responses to the same underlying recognition that the disease label is a poor handle on the biology.
- **What outcomes you measure.** Time-to-event, patient-reported outcomes, and trajectory-derived endpoints become first-class. See [Methods → Survival analysis](../methods/survival-analysis.md).
- **What the regulatory ask is.** A model that personalises a treatment recommendation is, by FDA / IMDRF definitions, almost certainly Software-as-a-Medical-Device. See [Regulatory](../regulatory/index.md).
- **What it means for a model to "work".** Discrimination (AUROC) is no longer enough. Calibration, decision-curve analysis, and within-subgroup performance are mandatory. See [Methods → Evaluation](../methods/evaluation.md).

## Where to next

Read [The patient-as-a-trajectory model](patient-trajectory.md) next. It introduces the single mental model that every downstream chapter relies on.
