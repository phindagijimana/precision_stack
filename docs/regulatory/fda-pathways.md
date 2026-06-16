# FDA pathways for AI/ML CDS

> *The regulatory landscape for AI/ML-driven clinical decision support in the US. What the carve-outs do and do not cover, how the FDA risk-classifies CDS, and what the predetermined-change-control plan (PCCP) lets you commit to up front.*

## Software as a Medical Device (SaMD)

The international frame ([IMDRF](https://www.imdrf.org/working-groups/software-medical-device-samd)) defines SaMD as software that performs a clinical function on its own.

The IMDRF risk framework crosses two axes:

- **State of the healthcare situation** — critical / serious / non-serious.
- **Significance of information** — treat-or-diagnose / drive clinical management / inform clinical management.

The cross product yields four risk classes that map to FDA Class I–III and EU MDR equivalents. A model that drives a treatment decision in a critical condition is the highest class; a model that informs (but does not drive) clinical management in a non-serious situation is the lowest.

## The 21st Century Cures Act CDS carve-out

Section 3060 of the Cures Act removed certain clinical-decision-support software from the FDA's definition of a device. The carve-out has four conjunctive conditions ([FDA final guidance, 2022](https://www.fda.gov/regulatory-information/search-fda-guidance-documents/clinical-decision-support-software)):

1. **Not analysing medical imagery or signals.** Imaging and waveform analysis remain devices.
2. **Displaying / analysing patient information already in the EHR.** Aggregating EHR data is allowed.
3. **Supporting a clinician's decision, not driving it.** The clinician must be able to *independently review the basis* for the recommendation.
4. **Not intended for emergencies.**

The third condition is the operationally critical one. A black-box deep-learning model whose output the clinician cannot independently scrutinise *does not qualify* for the carve-out and is regulated as SaMD.

Practical consequence: most precision-medicine ML models that output an opaque risk score with no human-interpretable basis remain devices, regardless of the carve-out.

## Pre-market pathways

- **510(k)** — substantial equivalence to a predicate. Most current AI/ML CDS clearances are 510(k).
- **De Novo** — for novel devices without a predicate.
- **PMA** — pre-market approval. Highest evidence bar; required for high-risk devices.
- **Breakthrough Device Designation** — accelerated review for devices addressing unmet needs in serious conditions.

The FDA's [AI/ML-enabled device list](https://www.fda.gov/medical-devices/software-medical-device-samd/artificial-intelligence-and-machine-learning-aiml-enabled-medical-devices) tracks all cleared devices.

## Predetermined Change Control Plan (PCCP)

A traditional 510(k) locks the model at clearance time. Changing a single weight requires a new 510(k). For AI/ML systems that need to evolve, this is fatal.

The **PCCP framework** ([FDA final guidance, 2024](https://www.fda.gov/regulatory-information/search-fda-guidance-documents/marketing-submission-recommendations-predetermined-change-control-plan-artificial)) lets a sponsor pre-declare:

- **Types of changes** anticipated — periodic retraining on more data, deployment to a new site, addition of new input modalities.
- **Methods** for making those changes — data quality bar, retraining protocol, validation procedure.
- **Performance maintenance** — every iteration must meet pre-declared metrics or halt.

The PCCP is reviewed and cleared once, then changes within its scope can be implemented without a new 510(k). Out-of-scope changes still require new submissions.

This is the modern pathway for any AI/ML CDS that is going to evolve. PrecisionStack treats writing a defensible PCCP as a deployment prerequisite, not an optional add-on.

## IDE and IND for AI in trials

When an AI/ML system is being investigated in a clinical trial:

- **IDE (Investigational Device Exemption)** — required for *significant-risk* device investigations.
- **IND (Investigational New Drug)** — required when the AI system is paired with an investigational drug whose use is informed by the AI's output (e.g. AI-guided dose selection).

A trial of an AI-driven CDS that influences treatment decisions almost certainly needs an IDE; an early signal-finding study with no patient-level intervention may not. The IDE / IND decision should be made with regulatory counsel and the FDA Q-Sub program.

## Equity, monitoring, and the post-market commitments

Cleared AI/ML CDS systems carry post-market commitments:

- **Real-world performance monitoring** — usually a registry-based study.
- **Periodic re-validation** — both on internally collected data and on external benchmarks.
- **Subgroup-equity audit** — performance stratified by demographics, with halt conditions.
- **Adverse-event reporting** — every clinically significant error logged and, where appropriate, reported.

These are not optional. They are the price of a PCCP and the operational counterpart to TRIPOD+AI / CONSORT-AI / DECIDE-AI reporting.

## A pre-submission checklist

- [ ] Intended-use statement — population, setting, user, what it does *not* do.
- [ ] Risk classification — SaMD class, FDA pathway candidate.
- [ ] Cures-Act CDS carve-out analysis — does it apply? Why or why not?
- [ ] Training-data inventory with consent / de-identification status.
- [ ] Subject-level + site-level held-out evaluation; per-subgroup metrics.
- [ ] Decision-curve analysis at deployment-relevant thresholds.
- [ ] Reproducibility artefacts — commit SHA, container digest, lockfile, weights hash.
- [ ] TRIPOD+AI, CLAIM (if imaging), DECIDE-AI checklists completed.
- [ ] PCCP draft if the model will evolve.
- [ ] Post-market monitoring plan with halt conditions.
- [ ] Adverse-event reporting plan.

A complete answer set is a multi-month effort. The way to make it tractable is to build the artefacts during development, not after.

## References

1. IMDRF. *Software as a Medical Device (SaMD): Key Definitions.* [link](https://www.imdrf.org/working-groups/software-medical-device-samd)
2. FDA. *Clinical Decision Support Software — Final Guidance.* 2022.
3. FDA. *Marketing Submission Recommendations for a Predetermined Change Control Plan for AI/ML-Enabled Device Software Functions.* Final guidance, 2024.
4. FDA. *Real-World Evidence to Support Regulatory Decision-Making for Drugs and Biological Products.* Final guidance, 2023.
5. Wu E, et al. How medical AI devices are evaluated: limitations and recommendations from an analysis of FDA approvals. *Nat Med.* 2021;27:582–584.
6. Sendak MP, et al. Real-world integration of an AI deep learning model into routine clinical care. *JMIR Med Inform.* 2020.

## Where to next

[Glossary](../glossary.md) — every term in the handbook in one place.
