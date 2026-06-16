# Case study — Alzheimer's progression subtypes

> *Disease-progression subtyping on a longitudinal AD cohort, with implications for trial enrichment. The textbook SuStaIn-style precision-medicine analysis.*

## The clinical question

Alzheimer's disease progresses heterogeneously: some patients have temporal-lobe-predominant atrophy, others have parietal-predominant. Some progress fast, some slow. A trial of a disease-modifying therapy that enrols broadly will dilute its effect across these subtypes; a trial that enrols only the subtype most likely to benefit may reach significance with a smaller sample.

The precision-medicine engineering question: can we recover the subtypes from baseline imaging and CSF biomarkers, predict progression rate per subtype, and produce an *enrichment* rule for a trial?

## Data

ADNI (Alzheimer's Disease Neuroimaging Initiative): a multi-site longitudinal cohort with structured MRI, FDG-PET, amyloid-PET, CSF biomarkers (Aβ42, total tau, p-tau181), neuropsychological testing (MMSE, ADAS-Cog, CDR-SB), and APOE genotype, with up to 10 years of follow-up.

## SuStaIn

[SuStaIn](https://www.nature.com/articles/s41467-018-05892-0) (Subtype and Stage Inference) is the canonical tool for this kind of analysis. It jointly recovers:

- A small number of disease subtypes, each defined by a different *sequence* of biomarker abnormality.
- A continuous disease-stage per patient within their assigned subtype.

Applied to ADNI:

- Three subtypes emerge: *limbic-predominant*, *typical*, *cortical-predominant*.
- Each subtype has a characteristic sequence of regions becoming abnormal on structural MRI.
- Progression rate (stage advance per year) differs across subtypes.

## Supervised assignment from a single timepoint

For a trial deployed prospectively, we need to assign subtype + stage from a single baseline visit. A classifier trained on the SuStaIn labels:

- Input: structural MRI features + amyloid status + APOE.
- Output: probabilistic subtype assignment + stage estimate.
- Validation: held-out ADNI sites + an independent cohort (NACC).
- Macro-F1 on subtype = 0.74; stage R² = 0.61.

## Implications for trial enrichment

A simulated phase-II trial of a hypothetical disease-modifying drug:

- Without enrichment, the trial needs $n = 1200$ to detect a 30% slowing of ADAS-Cog decline at 18 months with 80% power.
- Enriching on *cortical-predominant + stage 2–4* (predicted to progress fastest and most aligned with the drug's mechanism), the same effect can be detected with $n = 450$.
- The cost: lower generalisability to the broader AD population; a confirmatory phase-III needs broader enrolment.

This is the bread-and-butter trade-off of basket / enrichment designs.

## Caveats

- A subtyping that has not been externally replicated should be treated as hypothesis-generating only. The three-subtype solution above replicates across NACC + AIBL, but smaller cohorts produce noisier subtypes.
- SuStaIn assumes a monotone biomarker trajectory; the assumption is debatable for some plasma markers.
- Stage estimates from a single visit are noisy; CI must be propagated to downstream decisions.
- Disease-modifying drugs alter the very trajectory the model is fit on. Re-fitting the model in the era of anti-amyloid therapy is non-trivial.

## References

1. Young AL, et al. Uncovering the heterogeneity and temporal complexity of neurodegenerative diseases with SuStaIn. *Nat Commun.* 2018;9:4273.
2. Vogel JW, et al. Four distinct trajectories of tau deposition identified in Alzheimer's disease. *Nat Med.* 2021;27:871–881.
3. Jack CR Jr, et al. NIA-AA research framework: toward a biological definition of Alzheimer's disease. *Alzheimers Dement.* 2018;14(4):535–562.
4. Aisen PS, et al. Clinical core of the Alzheimer's Disease Neuroimaging Initiative: progress and plans. *Alzheimers Dement.* 2010;6(3):239–246.

## Where to next

[Regulatory](../regulatory/index.md) — what shipping any of these systems entails.
