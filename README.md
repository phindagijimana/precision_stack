# PrecisionStack

> *The open precision-medicine and clinical-trials handbook.*

### 🌐 Read it online → **[https://phindagijimana.github.io/precision_stack/](https://phindagijimana.github.io/precision_stack/)**

[![Site](https://img.shields.io/badge/site-PrecisionStack-009688?logo=readthedocs&logoColor=white)](https://phindagijimana.github.io/precision_stack/)
[![GitHub](https://img.shields.io/badge/github-phindagijimana%2Fprecision__stack-181717?logo=github)](https://github.com/phindagijimana/precision_stack)
[![CI](https://github.com/phindagijimana/precision_stack/actions/workflows/ci.yml/badge.svg)](https://github.com/phindagijimana/precision_stack/actions/workflows/ci.yml)
[![Deploy docs](https://github.com/phindagijimana/precision_stack/actions/workflows/docs.yml/badge.svg)](https://github.com/phindagijimana/precision_stack/actions/workflows/docs.yml)
[![License: MIT](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Python](https://img.shields.io/badge/python-3.10%2B-blue.svg)](pyproject.toml)

---

**PrecisionStack** is an open reference for building, evaluating, and deploying AI for **precision medicine and clinical trials** — written for a reader who walks in as a curious beginner and walks out able to ship a TRIPOD+AI-compliant model, run a Bayesian adaptive trial simulation, or stand up a synthetic-control arm with appropriate causal-inference machinery.

It is the clinical-decision counterpart to **[NeuroStack](https://github.com/phindagijimana/neuro_stack)** (the open neuroimaging handbook). Where NeuroStack covers *how to turn signal into evidence*, PrecisionStack covers *how to turn evidence into individualised decisions and trials*.

## Quick links

- 🌐 **Site:** [phindagijimana.github.io/precision_stack](https://phindagijimana.github.io/precision_stack/)
- 📚 **Getting started:** [What precision medicine actually is](https://phindagijimana.github.io/precision_stack/getting-started/what-is-precision-medicine/) · [The patient-as-a-trajectory model](https://phindagijimana.github.io/precision_stack/getting-started/patient-trajectory/) · [What AI can and cannot answer](https://phindagijimana.github.io/precision_stack/getting-started/what-ai-can-answer/)
- 🧱 **Foundations:** [Patient heterogeneity](https://phindagijimana.github.io/precision_stack/foundations/patient-heterogeneity/) · [Data modalities](https://phindagijimana.github.io/precision_stack/foundations/data-modalities/) · [Bias and confounding](https://phindagijimana.github.io/precision_stack/foundations/bias-and-confounding/)
- 🧪 **Tasks (A–G):** [Stratification](https://phindagijimana.github.io/precision_stack/tasks/stratification/) · [Treatment-response](https://phindagijimana.github.io/precision_stack/tasks/treatment-response/) · [Risk prediction](https://phindagijimana.github.io/precision_stack/tasks/risk-prediction/) · [Decision support](https://phindagijimana.github.io/precision_stack/tasks/clinical-decision-support/) · [Trial matching](https://phindagijimana.github.io/precision_stack/tasks/trial-matching/) · [Synthetic controls](https://phindagijimana.github.io/precision_stack/tasks/synthetic-controls/) · [Adaptive trials](https://phindagijimana.github.io/precision_stack/tasks/adaptive-trials/)
- 🔬 **Methods:** [Causal inference](https://phindagijimana.github.io/precision_stack/methods/causal-inference/) · [Survival analysis](https://phindagijimana.github.io/precision_stack/methods/survival-analysis/) · [Unsupervised subtyping](https://phindagijimana.github.io/precision_stack/methods/unsupervised-subtyping/) · [Evaluation](https://phindagijimana.github.io/precision_stack/methods/evaluation/)
- 🏥 **Case studies:** [Epilepsy surgery](https://phindagijimana.github.io/precision_stack/case-studies/epilepsy/) · [Oncology — immunotherapy response](https://phindagijimana.github.io/precision_stack/case-studies/oncology/) · [Alzheimer's progression subtypes](https://phindagijimana.github.io/precision_stack/case-studies/alzheimers/)
- 📜 **Regulatory:** [CONSORT-AI / SPIRIT-AI](https://phindagijimana.github.io/precision_stack/regulatory/consort-spirit/) · [DECIDE-AI](https://phindagijimana.github.io/precision_stack/regulatory/decide-ai/) · [FDA pathways](https://phindagijimana.github.io/precision_stack/regulatory/fda-pathways/)
- 📖 [Glossary](https://phindagijimana.github.io/precision_stack/glossary/) · [Further reading](https://phindagijimana.github.io/precision_stack/further-reading/)

## What's inside

- `docs/` — the handbook content, rendered with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/).
  - **Getting started** — what precision medicine actually is, the patient-as-a-trajectory mental model, the questions AI can and cannot answer.
  - **Foundations** — patient heterogeneity, the modalities that capture it (EHR, omics, imaging, wearables, PRO), bias and confounding.
  - **Tasks** — the seven workhorse problem types: stratification, treatment-response prediction, risk prediction, clinical decision support, trial matching, synthetic control arms, and adaptive trials.
  - **Methods** — causal inference, survival analysis, unsupervised subtyping, evaluation and calibration for individualised predictions.
  - **Case studies** — worked examples in epilepsy, oncology, and Alzheimer's disease.
  - **Regulatory** — CONSORT-AI, SPIRIT-AI, DECIDE-AI, FDA IDE/IND pathways, and the documentation an auditor will ask for.
- `src/precision_handbook/` — a small companion Python package (placeholder; utilities will land here as the handbook grows).
- `examples/` — short scripts exercising the package on a small synthetic cohort.
- `tests/` — pytest suite for the package.

## Quick start

```bash
git clone https://github.com/phindagijimana/precision_stack.git
cd precision_stack
pip install -e ".[docs,dev]"

# preview the site locally
mkdocs serve
```

## Status

Early. Content is being ported and expanded chapter-by-chapter. The tasks and methods sections are the most complete; case studies and regulatory sections are growing. Contributions and corrections are welcome — open an [issue](https://github.com/phindagijimana/precision_stack/issues) or a [pull request](https://github.com/phindagijimana/precision_stack/pulls).

## License

[MIT](LICENSE).
