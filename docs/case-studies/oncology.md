# Case study — oncology immunotherapy response

> *Stratifying melanoma patients for checkpoint-inhibitor benefit using molecular and clinical features. A canonical application of CATE-based subgrouping under modest sample size.*

## The clinical question

Anti-PD-1 therapy has transformed outcomes in metastatic melanoma — but the response rate is approximately 40%, and severe immune-related adverse events occur in 10–20%. Identifying *who* will benefit is the first-order precision-medicine question.

Working hypothesis: there are three biologically distinct groups of patients:

1. **Inflamed responders** — pre-existing tumour-infiltrating lymphocytes, high PD-L1, intact interferon signalling.
2. **High-TMB responders** — high tumour-mutation burden creates antigen targets.
3. **Cold-tumour non-responders** — neither of the above; resistant to checkpoint blockade alone.

The precision-medicine engineering question: can we recover these groups from observable features at decision time, and what is the predicted benefit of anti-PD-1 vs. an alternative?

## Data

| Modality | Source |
|---|---|
| Tumour mutation burden (TMB) | WES on archival tumour |
| PD-L1 IHC | Routine pathology |
| Tumour-infiltrating lymphocytes | H&E plus IHC for CD8 |
| Peripheral blood T-cell receptor repertoire | Adaptive ImmunoSeq |
| LDH, neutrophil-to-lymphocyte ratio | Routine labs |
| Prior treatment history | EHR |
| Sites of metastases | Imaging |

Sample size: 600 patients across two academic centres. Treated with anti-PD-1, ipilimumab, or combination.

## Stratification analysis

### Unsupervised step

Mixture model on the four molecular features (TMB, PD-L1, TIL score, TCR diversity), $K$ chosen by bootstrap stability:

- Three stable clusters emerge.
- Cluster A: high TIL, high PD-L1, intermediate TMB.
- Cluster B: high TMB, low TIL, low PD-L1.
- Cluster C: low across the board.

### Supervised step

Predict cluster membership from cheap, routinely-available features: LDH, NLR, prior-treatment history, age, sex, sites of metastases.

- Gradient-boosted classifier.
- Macro-F1 = 0.78 on the held-out site.
- The cheap classifier is what would deploy at decision time when full molecular profile is unavailable.

## CATE estimation

For each patient, estimate the contrast in 12-month PFS under anti-PD-1 alone vs. combination ipilimumab + anti-PD-1, using:

- A doubly-robust ML CATE estimator with cross-fitting.
- Adjustment for confounding by indication (combo is given to sicker patients with worse prognostic features).
- Outcome model: a survival forest.
- Propensity model: gradient-boosted classifier.

Results:

- Cluster A: $\hat{\tau}$ small — combination adds little over monotherapy and adds toxicity.
- Cluster B: $\hat{\tau}$ moderate, with wide CI — combination probably preferred but uncertain.
- Cluster C: $\hat{\tau}$ favours combination, with the largest absolute benefit.

A decision-support recommendation must include the wide CI and the toxicity profile, not just the point estimate.

## Validation

- Predicted-vs-observed benefit curve by decile.
- Calibration at 6, 12, 24 months.
- Subgroup calibration: sex, age band, prior immunotherapy, BRAF status.
- Sensitivity: E-value computation; the conclusion in Cluster C reverses only with a confounder of risk-ratio ≥ 4 with both treatment and outcome — i.e. a strong residual confounder.
- External cohort: 200 patients at a third site; partial replication of clusters with consistent direction of CATE.

## Decision-support deployment

- Within the tumour board, the model surfaces the patient's predicted cluster and the per-strategy benefit with CI.
- The tumour board overrides freely; overrides logged with rationale.
- Quarterly audit of override patterns and outcome differences.

## What is *not* claimed

- This is not a causal claim about *why* a patient responds; SHAP values from the cluster classifier are an aid to thinking, not an explanation.
- The CATE is conditional on the historical treatment policy; deployment in a setting with a different combo-use baseline requires re-evaluation.
- The model is conditional on having the molecular features. Deployment in low-resource settings without TMB and TCR data requires a stripped-down version with weaker stratification.

## References

1. Twomey JD, Zhang B. Cancer immunotherapy update: FDA-approved checkpoint inhibitors. *AAPS J.* 2021;23(2):39.
2. Litchfield K, et al. Meta-analysis of tumour- and T-cell intrinsic mechanisms of resistance to checkpoint blockade. *Cell.* 2021;184(3):596–614.
3. Snyder A, et al. Genetic basis for clinical response to CTLA-4 blockade in melanoma. *NEJM.* 2014;371:2189–2199.
4. Rizvi NA, et al. Mutational landscape determines sensitivity to PD-1 blockade in NSCLC. *Science.* 2015;348:124–128.

## Where to next

[Case study — Alzheimer's](alzheimers.md) for a longitudinal disease-progression-subtyping example.
