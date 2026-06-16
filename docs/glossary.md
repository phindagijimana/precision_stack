# Glossary

> *Every PrecisionStack term in one place.*

**Adaptive trial.** A clinical trial whose design includes prospectively planned modifications driven by accumulating data. See [Tasks → Adaptive trials](tasks/adaptive-trials.md).

**ATE (average treatment effect).** $\mathbb{E}[Y(1) - Y(0)]$. The population-level contrast between treated and untreated potential outcomes.

**AUROC.** Area under the receiver operating characteristic curve. A discrimination metric.

**Bayesian adaptive randomisation (BAR).** Allocation probabilities at each enrolment are proportional to the posterior probability that each arm is the best.

**Basket trial.** A trial enrolling multiple diseases that share a biomarker.

**Brier score.** $\frac{1}{n}\sum(p_i - y_i)^2$. Strictly proper scoring rule for probabilistic predictions.

**Calibration.** The property that among patients predicted to have probability $p$, the observed rate is $p$.

**CATE (conditional average treatment effect).** $\tau(x) = \mathbb{E}[Y(1) - Y(0) \mid X = x]$. The central object of precision medicine.

**Causal forest.** A random forest with honest splits chosen to maximise treatment-effect heterogeneity.

**CDS.** Clinical decision support.

**CLAIM.** Checklist for Artificial Intelligence in Medical Imaging.

**Concordance by indication / confounding by indication.** Bias caused by treatment selection depending on patient state.

**Confidence interval.** Frequentist interval that, in repeated sampling, would cover the true parameter with the stated probability.

**Confounder.** A variable that causes both treatment assignment and outcome.

**CONSORT-AI.** Reporting standard for randomised trials of AI interventions.

**Counterfactual.** What would have happened to a patient under a treatment they did not receive.

**Cox proportional hazards model.** Survival regression assuming covariate effects on a multiplicative hazard scale.

**DAG.** Directed acyclic graph; encodes the causal structure of a data-generating process.

**Decision-curve analysis.** Comparison of net benefit of a model-based policy across decision thresholds.

**DECIDE-AI.** Reporting standard for early-stage clinical evaluation of AI-driven decision support.

**Discrimination.** The ability of a model to rank patients correctly by outcome risk.

**Distribution shift.** Change in the joint distribution of $(X, Y)$ between training and deployment.

**DML.** Double / debiased machine learning.

**Doubly robust.** An estimator consistent if either the outcome model or the treatment model is correctly specified.

**E-value.** A sensitivity-analysis quantity for unmeasured confounding.

**EHR.** Electronic health record.

**Enrichment design.** A trial that restricts enrolment to a biomarker-defined subgroup expected to have a larger treatment effect.

**FHIR.** Fast Healthcare Interoperability Resources — HL7's modern interoperability standard.

**Fine–Gray model.** Subdistribution-hazard model for competing risks.

**G-methods.** A family of causal-inference methods for time-varying treatments — G-computation, G-estimation, marginal structural models with IPTW.

**Group-sequential design.** Adaptive trial with pre-computed interim test-statistic boundaries.

**Hazard ratio.** Multiplicative covariate effect in a Cox model.

**IDE.** Investigational Device Exemption.

**IND.** Investigational New Drug.

**Instrumental variable (IV).** A variable that affects treatment but is independent of outcome except through treatment.

**IPTW (inverse-probability-of-treatment weighting).** Re-weighting to break confounding by indication.

**ITE (individual treatment effect).** $Y_i(1) - Y_i(0)$. Unobservable per-patient contrast.

**Kaplan-Meier.** Non-parametric estimator of the survival function.

**LATE (local average treatment effect).** The IV-identified treatment effect for compliers.

**Master protocol.** An overarching protocol governing multiple trials sharing infrastructure and (often) a control arm.

**Mendelian randomisation.** IV analysis using genetic variants as instruments.

**Mixture model.** A probabilistic clustering model with parametric component distributions.

**NLP.** Natural language processing.

**Off-policy evaluation.** Estimating the value of a policy from data collected under a different policy.

**Omics.** Genomics, transcriptomics, proteomics, metabolomics.

**OMOP.** Common Data Model for EHR data.

**PCCP.** Predetermined Change Control Plan.

**Platform trial.** A perpetual trial infrastructure with arms entering and leaving.

**Positivity.** $0 < \Pr(A = 1 \mid X = x) < 1$ for every $x$ — an identifiability assumption.

**Potential outcome.** $Y_i(a)$, the outcome patient $i$ would have under treatment $a$.

**PRO.** Patient-reported outcome.

**Propensity score.** $e(x) = \Pr(A = 1 \mid X = x)$.

**Reliability diagram.** Calibration plot of predicted vs. observed rate by decile.

**RMST.** Restricted mean survival time.

**SaMD.** Software as a Medical Device.

**SHAP.** SHapley Additive exPlanations — feature-attribution for model predictions.

**SMART-on-FHIR.** Authorisation + launch framework for clinical apps integrated with FHIR-compliant EHRs.

**SPIRIT-AI.** Reporting standard for trial *protocols* of AI interventions.

**Stratification.** Dividing patients into groups more homogeneous than the original population.

**Subtype.** A discrete sub-population within a disease, defined by mechanism.

**SuStaIn.** Subtype and Stage Inference — a disease-progression-modelling tool.

**Synthetic control arm.** A counterfactual control constructed from external / historical data.

**Target trial emulation.** A discipline for analysing observational data as if it were a hypothetical RCT.

**TMB.** Tumour mutation burden.

**TMLE.** Targeted Maximum Likelihood Estimation.

**TRIPOD+AI.** Reporting standard for clinical-prediction models that use machine learning.

**Utility function.** A function $U(Y)$ encoding how much each outcome state matters.

**WGS / WES.** Whole-genome / whole-exome sequencing.
