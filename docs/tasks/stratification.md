# A. Patient stratification

> *Patient stratification is the act of dividing a population into subgroups that are more homogeneous in mechanism, prognosis, or treatment response than the original population.* It is the foundational task of precision medicine — every other task is implicitly stratified.

## Clinical reframing

When two patients with the same diagnosis behave differently, one of three things is true:

1. They share the same disease and the same mechanism, and the difference is noise. **Implication:** stop trying to stratify.
2. They share the same disease but the disease has *subtypes*, and they belong to different subtypes. **Implication:** find the subtypes.
3. They share neither the disease nor the mechanism; the diagnosis label is too coarse. **Implication:** *re-define* the disease.

Each of these is a different stratification problem. In oncology, modern molecular subtyping rejected option 1 and chose between options 2 and 3 depending on the cancer (HER2 status redefined breast cancer; PAM50 subtypes refined it). In epilepsy, the modern ILAE classification has moved towards option 3 — what we used to call "temporal lobe epilepsy" is being decomposed into mesial vs. neocortical vs. dual-pathology subtypes with different surgical outcomes.

## The statistical object

Stratification is a *partition* of the feature space $\mathcal{X}$ into a set of groups $\{G_1, G_2, \ldots, G_K\}$. The partition is useful when:

- Within-group variance of some target (mechanism marker, prognosis, treatment effect) is much smaller than the between-group variance.
- The groups are clinically actionable — they can be assigned at decision time from observable features.
- The groups are reproducible — re-applying the procedure to a new cohort recovers the same structure.

Two formal flavours:

- **Unsupervised.** Find structure in $\mathcal{X}$ alone. Useful for *discovery* of subtypes. See [Methods → Unsupervised subtyping](../methods/unsupervised-subtyping.md).
- **Supervised.** Find a partition that aligns with a known label $Y$. This is just classification; we call it stratification when the goal is *understanding* rather than just prediction.

In precision medicine we increasingly want a third flavour: stratification *with respect to treatment effect*, where the partition is chosen so that within-group treatment effect is most extreme. This is the goal of CATE-based subgroup methods — see [Methods → Causal inference](../methods/causal-inference.md).

## Method families

### Unsupervised

- **k-means / hierarchical clustering** on selected features. Easy to abuse; results depend strongly on feature scaling and the number of clusters $K$.
- **Mixture models** (Gaussian, Dirichlet-process). Probabilistic membership, interpretable density.
- **NMF / topic models.** Useful for non-negative feature data (gene-expression signatures, lesion-burden maps).
- **Deep clustering / variational autoencoders.** Latent-space clustering for high-dimensional modalities.
- **Sub-type and Stage Inference (SuStaIn)** — disease-progression model from cross-sectional data. Recovers subtypes *and* the disease-progression stages within each subtype. Widely used in Alzheimer's, Parkinson's, MS.

### Supervised subtype prediction

Once subtypes have been defined (e.g. by molecular reference standard), a supervised classifier maps observable features to subtype. The interesting precision-medicine question is *which observable features carry the subtype signal*, since the reference standard may be expensive (sequencing, biopsy) and you want a cheap proxy.

### CATE-based subgrouping

- **Causal forests** (Athey, Tibshirani, Wager). Honest split for treatment-effect heterogeneity.
- **Virtual twins.** Estimate $\hat{Y}(1) - \hat{Y}(0)$ per patient, cluster on the estimated effect.
- **Bayesian additive regression trees (BART)** for non-parametric CATE estimation.

## Validation

Stratification is the easiest task to over-fit and the hardest task to validate.

A reproducible stratification analysis must report:

- **Internal stability.** Bootstrapped re-runs should recover the same partition. Adjusted Rand index, normalised mutual information.
- **External replication.** The partition discovered on cohort A should be recoverable on cohort B with consistent biology.
- **Clinical association.** Subgroup membership should associate with an outcome of interest *that was not used to derive the subgroups*.
- **Actionability.** The features needed to assign a new patient must be obtainable at decision time.

Pitfalls:

- **The "k = 4" fallacy.** Most clustering algorithms produce *something* for any $K$. Selecting $K$ by silhouette / gap statistic without a stability test is how most spurious subtypes are born.
- **Tautological labels.** Defining subgroups by predicted outcome and then "validating" that they have different outcomes.

## A worked example — oncology immunotherapy responders

The goal: stratify melanoma patients into subgroups defined by their likely response to checkpoint inhibitor therapy.

1. **Features.** Tumour mutation burden (TMB), PD-L1 IHC score, T-cell receptor diversity from peripheral blood, baseline LDH, prior treatment history.
2. **Outcome.** Progression-free survival at 12 months, plus immune-related adverse-event grade.
3. **Method.** Mixture model on the four molecular features; supervised classifier from clinical features to predicted subgroup.
4. **Result.** Three subgroups emerge — "inflamed-responder", "high-TMB-responder", "non-responder" — with strikingly different PFS curves.
5. **External validation.** Apply the supervised classifier to an independent cohort. The same three groups recover, with overlapping confidence intervals on PFS.

See [Case study → Oncology](../case-studies/oncology.md) for the long-form treatment of this analysis.

## Implementation sketch — causal forest

```python
from econml.dml import CausalForestDML
from sklearn.ensemble import RandomForestRegressor, RandomForestClassifier

# X: covariates; T: treatment indicator; Y: outcome
forest = CausalForestDML(
    model_t=RandomForestClassifier(),
    model_y=RandomForestRegressor(),
    n_estimators=2000,
    min_samples_leaf=20,
    honest=True,
)
forest.fit(Y=y, T=t, X=X)

# CATE per patient
tau_hat = forest.effect(X)

# Stratify on predicted benefit
deciles = pd.qcut(tau_hat, 10, labels=False)
```

The honest split is essential: it prevents the same data being used to choose splits and to estimate effects within each leaf. Without it, treatment-effect estimates in the discovered subgroups are biased.

## Limitations

- A stratification is only as good as the features that define it. Stratifying on EHR data alone surfaces care-pattern subgroups, not biological ones.
- Reproducibility across cohorts is the bar. Most published subtypes do not survive external replication; treat single-cohort results as hypothesis-generating only.
- Even a "real" subtype is a population-level statement. A given patient may fall on the boundary; report a probability of membership, not a hard assignment.

## Exercises

1. **Take a public cohort** (UK Biobank diabetes cohort, METABRIC for breast cancer, ADNI for AD) and reproduce a published subtype paper. Check the stability across bootstraps.
2. **Compare CATE-based subgrouping** with simple outcome-based clustering on the same RCT. Which one gives a more clinically actionable partition?
3. **Write the stratification check-list** for your project: features, label, method, $K$ selection, stability test, external cohort, actionability.

## References

1. Wang Q, Black SE, et al. SuStaIn: Subtype and Stage Inference. *Nat Commun.* 2018;9:4273.
2. Athey S, Tibshirani J, Wager S. Generalized random forests. *Ann Stat.* 2019;47(2):1148–1178.
3. Künzel SR, Sekhon JS, Bickel PJ, Yu B. Metalearners for estimating heterogeneous treatment effects using machine learning. *PNAS.* 2019;116(10):4156–4165.
4. Hastie T, Tibshirani R, Friedman J. *The Elements of Statistical Learning,* Ch. 14 (clustering). Springer, 2009.
5. Hingorani AD, van der Schaar M, et al. Improving the odds of drug development success through human genomics: modelling study. *Sci Rep.* 2019;9:18911.

## Where to next

[B. Treatment-response prediction](treatment-response.md) — the most clinically valuable, and methodologically hazardous, of the seven tasks.
