# Patient heterogeneity

> *Two patients with the same diagnosis are almost never the same patient.* This chapter names the axes along which they differ and points to where each axis is exploited in later chapters.

## The twelve axes

We work from a pragmatic taxonomy used throughout the handbook. None of the boundaries is sharp, but the list is enough to organise the discussion.

1. **Genetics.** Germline variants (SNPs, CNVs, rare variants), somatic mutations (especially in oncology), epigenetics. Captured by sequencing or genotyping panels. Drives pharmacogenomic response (CYP2D6 → tamoxifen metabolism), disease risk (BRCA1/2 → breast/ovarian), and increasingly therapy choice (EGFR mutation → gefitinib).
2. **Age.** Calendar age is a crude summary of biological age. More refined: GrimAge, PhenoAge, epigenetic clocks, frailty indices.
3. **Sex / gender.** Distinguish *sex-as-biological-variable* (chromosomes, hormones, organ structure) from *gender* (social identity, behavioural). Both matter; they are not the same. Many drugs have sex-differential PK/PD.
4. **Disease subtype.** The same diagnosis can hide multiple biological processes. NSCLC adenocarcinoma is not one disease; multiple sclerosis is at least three; epilepsy is dozens. See [Methods → Unsupervised subtyping](../methods/unsupervised-subtyping.md).
5. **Imaging patterns.** Lesion topology, atrophy distribution, perfusion pattern, connectivity disruption. The modality lives in NeuroStack ([Fundamentals → Modalities](https://phindagijimana.github.io/neuro_stack/fundamentals/modalities/)); the precision-medicine use lives here.
6. **Lab values.** Eligibility (GFR for nephrotoxic drug), baseline severity (NIHSS, MELD), treatment-response biomarkers (HbA1c, viral load), prognosis (LDH in melanoma).
7. **Medications.** Concomitant drugs alter response (P-gp inducers vs. inhibitors), adherence drives effective dose, prior failures structure the choice set.
8. **Lifestyle.** Diet, activity, smoking, alcohol, sleep, sexual behaviour. Often poorly captured; wearables and PROs are how we are starting to fix that.
9. **Environment.** Air quality, occupational exposures, latitude (vitamin D, MS), water quality, housing.
10. **Comorbidities.** A diabetic with chronic kidney disease and atrial fibrillation lives in a different therapeutic space than a young patient with the same primary disease.
11. **Immune profile.** HLA type (drug hypersensitivity, transplant), autoantibody status, prior infections (immunological memory), microbiome.
12. **Treatment history.** What worked, what failed, what was discontinued for side effects. In refractory disease, treatment history dominates the choice set.

## What each axis does to the trajectory

Recall the [patient-as-a-trajectory model](../getting-started/patient-trajectory.md). Each axis enters the trajectory in one of three ways:

- **Baseline state $S_0$.** Genetics, sex, environment from birth.
- **Time-varying state $S_t$.** Age, imaging, labs, medications, lifestyle, comorbidities, immune profile.
- **History.** The set of past decisions and outcomes that condition future eligibility — treatment history.

This matters because each entry mode demands a different modelling choice:

- Static covariates → standard regression.
- Time-varying covariates → longitudinal models or recurrent / transformer architectures.
- Treatment history → counterfactual sequence modelling. See [Tasks → Treatment-response prediction](../tasks/treatment-response.md).

## Which axis matters for which task

The honest answer is *all of them, in principle*. The useful answer:

| Task | Axes that typically dominate |
|---|---|
| Stratification | Genetics, disease subtype, imaging, immune profile. |
| Treatment-response prediction | Genetics, disease subtype, comorbidities, treatment history, labs. |
| Risk prediction | Age, comorbidities, labs, treatment history. |
| Clinical decision support | All of the above, weighted by the candidate decision. |
| Trial matching | Disease subtype, labs, treatment history, age. |
| Synthetic control | Whatever the original trial used as inclusion criteria. |
| Adaptive trials | Whatever is updated by interim analyses. |

## Heterogeneity ≠ noise

A persistent confusion: a clinical-AI model that gets noisy when you stratify by subgroup is *not* a model with too much variance. It is a model whose population-level estimate is averaging over heterogeneous mechanisms. The fix is *more granular modelling*, not more regularisation.

Concretely: if you train a single mortality model on a mixed cohort and the calibration plot bends in opposite directions for men and women, the issue is not noise — the population is a mixture of two sub-populations with different relationships between $X$ and $Y$. Either:

1. Fit a per-subgroup model, *or*
2. Include the subgroup interaction terms explicitly, *or*
3. Switch to a method that estimates a CATE rather than a population-level effect.

## Measured heterogeneity is bounded by the axes you observe

You can only stratify on what you measure. A trial enrols 500 patients and randomises drug A vs drug B with respect to disease severity. You discover post-hoc that response is bimodal. Two reasons that is hard to act on:

1. The variable that explains the bimodality may not have been measured.
2. Even if it was measured, the bimodality may be an artefact of the small sample.

This is why [Foundations → Data modalities](data-modalities.md) emphasises *what to measure*, and [Methods → Causal inference](../methods/causal-inference.md) emphasises *what assumptions are needed to act on what you measured*.

## Pre-specification vs. exploratory subgroup analysis

A pre-specified subgroup analysis (in the SAP, before unblinding) is hypothesis testing. A post-hoc subgroup is hypothesis generation. The literature is littered with celebrated post-hoc subgroups that did not replicate.

Modern precision-medicine analyses operationalise this carefully:

- The CATE model is fit on the *training set* with cross-validation.
- Subgroups are *defined by the model* (e.g. by predicted-benefit decile).
- The CATE in each model-defined subgroup is *re-estimated on the held-out test set*.

See [Methods → Evaluation](../methods/evaluation.md) for the formal procedure and [Künzel et al. 2019](https://www.pnas.org/doi/10.1073/pnas.1804597116) for one canonical implementation.

## Exercises

1. **Pick a disease you know well.** Write down the twelve axes for a typical patient. Which axes are easy to get for free from the EHR, which require an expensive assay, and which are essentially never measured?
2. **Find a published RCT in that disease.** Look at the inclusion/exclusion criteria. Which axes are explicitly used as eligibility criteria? Which are *omitted*, implicitly assumed to be balanced by randomisation?
3. **Sketch a CATE.** For two candidate treatments in that disease, sketch which axes you would expect to drive heterogeneity in benefit. This is the prior you'd want a stratification model to recover.

??? success "Solutions"
    1. In oncology, genetics and imaging are now routinely captured; immune profile and microbiome typically not. In epilepsy, EEG and MRI are; genetics is improving; lifestyle and adherence are usually not.
    2. Older trials almost always omit genetics, immune profile, environment, lifestyle. Modern trials increasingly enrich on genetics for a biomarker-defined eligible population.
    3. Useful exercise: a CATE that depends on a variable not in the trial is a CATE you cannot estimate from that trial — even with arbitrary modelling capacity.

## Where to next

[Data modalities](data-modalities.md) covers how each of these axes actually enters the dataset, and the modality-specific failure modes.
