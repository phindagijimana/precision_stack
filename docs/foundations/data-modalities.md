# Data modalities

> *Every modality in clinical AI comes with its own representation, its own biases, and its own failure modes. The patient is the join key.*

This chapter walks through the modalities you'll encounter, what they look like as data, how they get linked, and the modality-specific failure modes that the model-fitting chapters cannot rescue you from.

For neuroimaging modalities specifically, NeuroStack has a much more detailed treatment ([Fundamentals → Modalities](https://phindagijimana.github.io/neuro_stack/fundamentals/modalities/)). Here we focus on the *precision-medicine integration* layer.

## Electronic health records (EHR)

The workhorse modality. Structured tables (encounters, problems, diagnoses, medications, labs, vitals) plus unstructured notes.

What it gives you:

- Longitudinal observation of the trajectory at irregular time points.
- Routinely-collected, low-cost coverage of the population.
- The ground truth for trial-matching and cohort-discovery tasks.

What it lies about:

- **Billing-driven coding.** Diagnoses are added when they affect reimbursement; presence in the record does not mean presence in the patient. Absence is even more ambiguous.
- **Care-pattern as label.** "Sepsis" is often defined by what was ordered (lactate, blood cultures, vasopressors). A model trained on this learns the care pattern, not the biology.
- **Missingness is informative.** If a lab is not ordered, the clinician did not suspect the condition. Missing-at-random is almost never the right assumption.
- **Site-specific encoding.** The same condition can be coded differently across systems. Same vendor, different sites, same site, different units (lb vs kg, F vs C).

Engineering checklist:

- Map to a common data model (OMOP, PCORnet, Sentinel, i2b2). OMOP is the de facto standard for cross-institution precision-medicine work.
- For free text, extract structured concepts with NLP (cTAKES, MetaMap, MedSpaCy, recent LLM-based extractors) and store them alongside the structured fields with provenance.
- Capture and preserve *timestamps*. The decision-time of every observation is the most clinically important metadatum and the easiest to lose.

See also: [Tasks → Trial matching](../tasks/trial-matching.md), [Tasks → Risk prediction](../tasks/risk-prediction.md).

## Omics — genomics, transcriptomics, proteomics, metabolomics

Higher-dimensional, lower-frequency, expensive.

| Layer | Typical assay | Dimensionality | Latency to result |
|---|---|---|---|
| Genomics (germline) | WGS, WES, SNP array | $10^4$–$10^7$ | Days–weeks |
| Genomics (somatic) | tumour panel, ctDNA | $10^2$–$10^4$ | Days |
| Transcriptomics | bulk RNA-seq, scRNA-seq | $10^4$ | Days–weeks |
| Proteomics | mass spec, SOMAscan, Olink | $10^3$–$10^4$ | Days |
| Metabolomics | LC-MS, NMR | $10^2$–$10^3$ | Days |

What omics gives you:

- Mechanism. The closest modality to causal handles on biology.
- High-information stratification axes that aren't in the EHR.

What omics lies about:

- **Batch effects.** Plate, day, technician, lot of reagent. ComBat, SVA, RUV — pick one.
- **Selection.** Whose tumour was sequenced? Patients who could afford it, patients in trials, patients who survived long enough.
- **Variant interpretation.** ClinVar pathogenicity drifts over time; the same variant can move from "uncertain" to "pathogenic" between data freezes.

Engineering checklist:

- Pin the reference build (GRCh37 vs GRCh38) and the variant-annotation version.
- Store raw + processed + summary. The summary is what models train on; the raw is what you re-process when annotation changes.
- Treat tumour and germline as separate keys with documented derivation (matched-normal subtraction).

## Imaging

Already an established modality. PrecisionStack-specific issues:

- **Site / scanner effects** are the imaging analogue of EHR site effects. ComBat for radiomics, harmonisation networks (e.g. CycleGAN-style) for raw images. See NeuroStack [Analysis → Group statistics](https://phindagijimana.github.io/neuro_stack/analysis/group-stats/).
- **The acquisition time** is critical. An MRI from three years ago is a different feature than one from last week.
- **Quantitative vs categorical imaging features.** Radiomics derives $10^2$–$10^3$ features per ROI; deep features come out of a CNN backbone at $10^2$–$10^3$ per study. Combine carefully.

Imaging for precision medicine is most often used as input to stratification, risk prediction, and treatment-response models — see [Case studies](../case-studies/index.md).

## Wearables and digital biomarkers

Continuous, passively-collected, patient-facing.

Examples: actigraphy from a smartwatch, continuous glucose monitoring, ECG patch, voice biomarkers, smartphone keystroke dynamics, gait from inertial sensors.

What they give you:

- Frequency. Sampling rates that would be impractical clinically.
- Ecologically valid measurement — at home, not at the clinic.

What they lie about:

- **Adherence and wear time.** Missingness is non-ignorable; a patient who removes the device for a week is signalling.
- **Device generations.** A model trained on Apple Watch Series 6 may not generalise to Series 10 or to a Fitbit.
- **Calibration drift.** Sensor accuracy changes over device lifetime.

Engineering checklist:

- Define an "analysis-grade" wear-time threshold and report it.
- Pin firmware versions in the metadata.
- Treat the time series as the unit of analysis, not a daily summary.

## Patient-reported outcomes (PRO)

Surveys, scales, ePROs delivered through an app.

Examples: PROMIS, EQ-5D, PHQ-9, GAD-7, MS Quality of Life, Seizure Severity Questionnaire.

What they give you:

- The patient's perspective. The single dimension regulators are increasingly asking for.
- A measure that *is the outcome*, not a proxy for it.

What they lie about:

- **Selection.** Patients who respond to surveys differ systematically.
- **Recall bias.** "How were your symptoms in the last month?"
- **Cultural and linguistic translation.** Validated scales are validated in their original population.

PROs are central to oncology trials and to the modern conception of clinical-benefit assessment. They are under-represented in EHR-derived AI work and should be added when available.

## Joining modalities

Every modality has a primary key (MRN, deidentified subject ID, biospecimen ID). Joins happen at two levels:

1. **Subject level.** A single anchoring identifier links every modality. Provenance: who created the link, what authority authorised it, what de-identification was applied.
2. **Event level.** A specific encounter / acquisition / visit / sample. Timestamps matter; the same patient on different days can yield contradictory states.

A reliable engineering pattern is the OMOP **Common Data Model** for structured EHR plus a **manifest** ([NeuroStack → Data engineering](https://phindagijimana.github.io/neuro_stack/data-engineering/dag/)) per modality that names every source file, its hash, its acquisition time, and its provenance. Anything joined off the manifest is reproducible; anything joined off filenames is not.

## The information-versus-cost-versus-availability triangle

In practice every project chooses two of:

- **High information** — the modality answers the question precisely (omics, multi-modal imaging).
- **Low cost** — the modality is collected anyway (EHR, claims).
- **Wide availability** — every patient in the cohort has it.

You almost never get all three. Recognising the trade-off explicitly is how you avoid pretending your model is widely deployable when it really requires a $5000 assay.

## Modality-specific failure modes — a reference table

| Modality | Most common failure mode | Mitigation |
|---|---|---|
| EHR (structured) | Care-pattern as label | Use clinician-adjudicated labels for a held-out subset |
| EHR (free text) | Entity recognition drift across sites | Test extractor on each site separately |
| Genomics | Batch effects | ComBat / RUV, include batch as a covariate |
| Imaging | Scanner shift | Harmonisation, multi-site held-out evaluation |
| Wearables | Adherence-correlated missingness | Treat wear-time as a covariate, sensitivity analysis |
| PRO | Selection on response | Inverse-probability-of-response weighting |

## Exercises

1. **Pick the most ambitious project in your group** and list which modalities it depends on. For each, name the failure mode you are most exposed to and the concrete mitigation step in your pipeline.
2. **Find a published multi-modal model.** Read the *Data* section and identify which of the modality-specific failure modes the authors did and did not address. Most papers address one or two; almost none address all of them.
3. **Manifest exercise.** For a project you control, write a one-line description of every file your model consumes, what it depends on, and which manifest entry covers it. If you cannot do this in under an hour, the model is harder to defend than it should be.

## Where to next

[Bias and confounding](bias-and-confounding.md) — the systematic distortions that survive even a clean modality join, and the named mitigations for each.
