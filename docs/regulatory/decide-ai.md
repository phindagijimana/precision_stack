# DECIDE-AI

> *The reporting standard for the early-stage clinical evaluation of decision-support systems driven by AI. The bridge between a validation paper and an RCT.*

## Why DECIDE-AI exists

Most clinical AI systems undergo a long awkward phase between *"validated on retrospective data"* and *"randomised against standard of care"*. In that phase:

- The system is deployed in a real clinical workflow.
- A handful of clinicians use it.
- Outcomes are observed in a small, non-randomised cohort.
- Lessons feed back into the system, the workflow, and the planning of a future RCT.

This phase is a goldmine of safety and usability information; until [DECIDE-AI](https://www.bmj.com/content/377/bmj-2022-070904) (Vasey et al., 2022) there was no agreed format for reporting it.

## What DECIDE-AI covers

The standard has 17 AI-specific items plus 10 generic items, organised around:

- **System description** — version, intended use, training data, limitations.
- **Implementation context** — site, workflow integration, training of users, IT plumbing.
- **Patients and clinicians** — eligibility, demographics, sample size justification.
- **Performance evaluation** — predictive performance with calibration, AI-human agreement, override patterns.
- **Safety evaluation** — adverse events, near-misses, deviation reporting.
- **Usability and human factors** — usability scores, workflow disruption, alert fatigue.
- **Lessons for the future RCT** — what the early evaluation tells you about the design of the definitive trial.

## How DECIDE-AI sits relative to the others

| Phase | Standard |
|---|---|
| Algorithm development + retrospective validation | TRIPOD+AI |
| Imaging-specific algorithm reporting | CLAIM |
| Trial protocol | SPIRIT-AI |
| Trial reporting | CONSORT-AI |
| Early-stage clinical evaluation | **DECIDE-AI** |
| Diagnostic accuracy study | STARD-AI |

DECIDE-AI is the *only* standard that explicitly covers the messy real-world piloting phase between validation and trial. If you are running a pilot, you should be reporting against DECIDE-AI.

## A minimal DECIDE-AI deployment

Suppose you've built a sepsis early-warning model and you want to pilot it in a single hospital before the multi-site RCT. The DECIDE-AI commitments:

- **Pre-specify** the version of the model, the workflow integration (how it appears in the EHR, who sees it, at what time), and the user training plan.
- **Pre-specify** which clinicians use it (e.g. one ICU vs. another as concurrent control).
- **Pre-specify** the predictive-performance metrics, the AI-human agreement metric, and the safety metrics.
- **Pre-specify** the override capture mechanism — the model must allow override and record the reason.
- **Pre-specify** the qualitative usability evaluation (System Usability Scale, semi-structured interviews).
- **Plan adverse-event capture** — both clinically significant model errors and near-misses where the model would have erred had it not been overridden.
- **Plan the analysis** — both quantitative (predictive performance, override rate) and qualitative (themes from clinician interviews).

The report is then a 6,000–10,000-word manuscript that covers all 27 items.

## What DECIDE-AI catches that other standards do not

- **Workflow drift.** The model that worked at the prototype was used by motivated researchers; the deployed version is used by tired clinicians at 3 a.m.
- **Alert fatigue.** Quantified by per-clinician override rate over time.
- **Trust dynamics.** Captured by usability scales and qualitative interviews.
- **Integration brittleness.** When the EHR update breaks the FHIR feed, the model breaks. Documenting that.
- **Unanticipated stakeholder effects.** The model may shift workload from the bedside nurse to the resident; that is data.

## References

1. Vasey B, Nagendran M, Campbell B, et al. Reporting guideline for the early-stage clinical evaluation of decision-support systems driven by AI: DECIDE-AI. *BMJ.* 2022;377:e070904.
2. McCradden MD, et al. Patient safety and quality improvements with AI-supported care. *NPJ Digit Med.* 2023.
3. Sendak MP, et al. Surfacing AI safety and effectiveness for clinicians. *Lancet Digit Health.* 2024.

## Where to next

[FDA pathways](fda-pathways.md) — the regulatory frame for any of this becoming a commercial product.
