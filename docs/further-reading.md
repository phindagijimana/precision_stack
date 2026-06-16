# Further reading

> *Books, primary references, free courses, and active conferences worth tracking.*

## Books

- Hernán MA, Robins JM. *Causal Inference: What If.* Chapman & Hall/CRC, 2020. ([free PDF](https://www.hsph.harvard.edu/miguel-hernan/causal-inference-book/))
- Pearl J. *Causality: Models, Reasoning, and Inference.* Cambridge UP, 2009.
- Steyerberg EW. *Clinical Prediction Models.* 2nd ed. Springer, 2019.
- Berry SM, Carlin BP, Lee JJ, Müller P. *Bayesian Adaptive Methods for Clinical Trials.* CRC Press, 2010.
- Klein JP, Moeschberger ML. *Survival Analysis.* 2nd ed. Springer, 2003.
- Hastie T, Tibshirani R, Friedman J. *The Elements of Statistical Learning.* Springer, 2009. ([free PDF](https://hastie.su.domains/ElemStatLearn/))
- Murphy KP. *Probabilistic Machine Learning: An Introduction* + *Advanced Topics.* MIT Press, 2022/2023.
- Topol E. *Deep Medicine.* Basic Books, 2019. (Clinical-perspective overview.)
- Kohane I, Drazen JM, Beam AL. *Hidden in Plain Sight: What Healthcare Data Can Tell Us About Bias.* Forthcoming. (Manuscript drafts circulate; check the authors' sites.)

## Foundational papers — keep these close

- Rosenbaum PR, Rubin DB. The central role of the propensity score in observational studies for causal effects. *Biometrika.* 1983;70(1):41–55.
- Pearl J. Causal diagrams for empirical research. *Biometrika.* 1995;82(4):669–688.
- Imbens GW, Angrist JD. Identification and estimation of local average treatment effects. *Econometrica.* 1994;62(2):467–475.
- Hernán MA, Robins JM. Using big data to emulate a target trial when a randomized trial is not available. *Am J Epidemiol.* 2016;183(8):758–764.
- Chernozhukov V, et al. Double/debiased machine learning. *Econometrics J.* 2018;21(1):C1–C68.
- Künzel SR, et al. Metalearners for estimating heterogeneous treatment effects using machine learning. *PNAS.* 2019;116(10):4156–4165.
- Athey S, Tibshirani J, Wager S. Generalized random forests. *Ann Stat.* 2019;47(2):1148–1178.
- Berry DA. The brave new world of clinical cancer research. *JAMA.* 2015;313(16):1571–1572.

## Reporting standards — read in full once

- Collins GS, Moons KGM, et al. TRIPOD+AI statement. *BMJ.* 2024;385:e078378.
- Liu X, et al. CONSORT-AI extension. *Nat Med.* 2020;26(9):1364–1374.
- Cruz Rivera S, et al. SPIRIT-AI extension. *Nat Med.* 2020;26(9):1351–1363.
- Vasey B, et al. DECIDE-AI. *BMJ.* 2022;377:e070904.
- Mongan J, Moy L, Kahn CE. CLAIM. *Radiol Artif Intell.* 2020;2(2):e200029.

## Free courses

- Miguel Hernán, *Causal Diagrams.* HarvardX edX.
- Susan Athey, *Machine Learning and Causal Inference.* Stanford GSB lecture notes.
- Stefan Wager, *Stats 361: Causal Inference.* Stanford lecture notes.
- Andrew Gelman, *Bayesian workflow.* StanCon lecture videos and *Statistical Rethinking* (McElreath).
- Harvard Catalyst, *Reusable Learning Objects.* (Modules on RWE.)

## Conferences and workshops to track

- **Machine Learning for Healthcare (MLHC).** Annual. Strong on precision-medicine ML methods.
- **CHIL — ACM Conference on Health, Inference, and Learning.** Annual.
- **AMIA Annual Symposium.** Major venue for clinical informatics.
- **ML4H workshop at NeurIPS.** Annual.
- **JSM Biostatistics in Practice / Adaptive Designs.** ASA programmes.
- **ENAR / WNAR.** Eastern / Western North American biostatistics meetings.
- **FDA Drug Information Association (DIA) meetings** for regulatory perspectives.

## Active newsletters and blogs

- Eric Topol's *Ground Truths.*
- Atul Butte's lab blog and lecture series.
- Andrew Gelman's *Statistical Modeling, Causal Inference, and Social Science* blog.
- Stefan Wager's notes and code on causal-ml.
- The *Journal of the American Medical Informatics Association* (JAMIA) for ongoing clinical-informatics work.

## Datasets and registries

- **MIMIC-IV** — ICU EHR, freely available with credentialed access.
- **eICU** — multi-centre ICU dataset.
- **UK Biobank** — population-scale biobank with imaging + genomics + EHR.
- **ADNI** — Alzheimer's longitudinal cohort.
- **METABRIC, TCGA** — breast and pan-cancer molecular datasets.
- **All of Us** — diverse US precision-medicine cohort.
- **ATLAS** — stroke imaging dataset.
- **SEER, Flatiron** — real-world oncology cohorts.
- **Sentinel, PCORnet** — distributed-EHR networks.

## Software

| Domain | Tool |
|---|---|
| Causal inference | EconML, CausalML, DoWhy, R `causalForest`, `grf` |
| Survival | lifelines, pycox, scikit-survival, `survival` R |
| Bayesian adaptive trials | `RBesT`, `bcrm`, BayesPharma |
| Clinical NLP | MedSpaCy, cTAKES, ScispaCy, MedCAT |
| EHR engineering | OHDSI / OMOP tooling, Synthea |
| Imaging | NeuroStack's recommendations apply ([Computing → Python stack](https://phindagijimana.github.io/neuro_stack/computing/python-stack/)) |
| Decision-curve analysis | `dcurves` R / Python |
| Conformal prediction | `mapie`, `nonconformist` |

## Where to go next inside the handbook

If you have made it here, you have the full PrecisionStack tour. Treat the [Methods](methods/index.md) chapters as a reference, the [Tasks](tasks/index.md) chapters as a working menu, and the [Case studies](case-studies/index.md) as long-form examples to imitate.

Contributions to the handbook are welcome — open an issue or a PR.
