# Unsupervised subtyping

> *Finding structure in patient features without using the outcome as a label. The most powerful and most over-promised technique in precision medicine.*

## What we are estimating

Given an $n \times p$ matrix of features $X$ (no labels), we want a partition $\{G_1, \ldots, G_K\}$ such that:

- Within each $G_k$, patients are similar with respect to features.
- Across $G_k$'s, patients are dissimilar.
- The partition is reproducible across cohorts.
- The partition is *clinically meaningful* — it explains variation in some independent target (outcome, biomarker, mechanism).

A subtype is a hypothesis about a hidden mechanism. The data alone cannot establish that the hypothesis is true; it can only suggest it.

## Method families

### k-means and hierarchical clustering

The default starting points. k-means assumes spherical clusters of similar variance; agglomerative hierarchical clustering produces a dendrogram with controllable linkage. Both are sensitive to feature scaling and to the choice of $K$.

### Mixture models

A finite mixture model fits each cluster as a parametric distribution:

$$
p(x) = \sum_{k=1}^K \pi_k \, f(x \mid \theta_k)
$$

Gaussian mixtures are the standard. Membership is probabilistic, which is more honest than hard assignment. Dirichlet-process mixtures avoid choosing $K$ — at the cost of having to choose a concentration parameter.

### NMF and topic models

Non-negative matrix factorisation decomposes $X \approx W H$ with non-negative entries. For gene-expression and lesion-burden features, the non-negativity constraint produces parts-based representations that often align with biology better than PCA. NMF subtypes ([Brunet et al. 2004](https://www.pnas.org/doi/10.1073/pnas.0308531101)) are the foundation of many oncology molecular-subtype systems.

### Deep / latent-variable clustering

VAEs and self-supervised models produce a latent embedding $z = f(x)$ on which clustering is applied. Useful for very high-dimensional, multi-modal data (imaging + omics + EHR). Strengths: capture non-linear structure. Weaknesses: training is non-deterministic; clusters can be sensitive to the architecture and the regularisation.

### Subtype and Stage Inference (SuStaIn)

For diseases with both subtypes *and* progression stages, SuStaIn ([Young et al. 2018](https://www.nature.com/articles/s41467-018-05892-0)) jointly recovers both. Each subtype has its own sequence of biomarker abnormality. Particularly successful in Alzheimer's, Parkinson's, MS, and frontotemporal dementia.

### Disease-progression and pseudotime models

Originally developed for single-cell biology (Monocle, PAGA), pseudotime methods are increasingly used in clinical-trajectory modelling. They order patients on a latent disease-progression axis and reveal branching structure (subtypes as branches).

## Stability and validation

A subtype claim that has not been replicated externally should be treated as hypothesis-generating only.

Required diagnostics:

- **Bootstrap stability.** Refit on resampled patients; compare partitions with adjusted Rand index or normalised mutual information.
- **Feature-perturbation stability.** Drop a feature, refit, check partition stability.
- **Independent replication cohort.** Apply the trained subtype assignment rule to a new cohort and check whether the same structure (or at least a coherent generalisation) appears.
- **Association with an independent target.** Subtypes should explain variance in an outcome that was not used to derive them.

How $K$ is chosen matters enormously:

- The elbow / silhouette / gap statistic are necessary but not sufficient.
- Stability across resampling is the strongest signal. The right $K$ is the one whose partition is stable under bootstrap.
- Pre-registration of the analysis pipeline before unblinding the validation cohort is the gold standard.

## CATE-based subgrouping

When the goal is *treatment-effect heterogeneity* rather than mechanism, subtyping should be supervised by the estimated CATE rather than by feature density. See [Methods → Causal inference](causal-inference.md) and [Tasks → Stratification](../tasks/stratification.md).

The typical pipeline:

1. Estimate per-patient CATE $\hat{\tau}(x)$ with a doubly-robust ML estimator.
2. Cluster on $\hat{\tau}(x)$ or fit a tree to find subgroup splits that maximise within-group treatment-effect homogeneity.
3. Validate the subgroups by re-estimating $\tau$ within each group on held-out data.

This avoids the trap of inventing subgroups that correlate with treatment intensity rather than treatment benefit.

## Mixed-membership and probabilistic assignment

A clinically useful subtype assignment is rarely categorical. A patient on the boundary between two subtypes should receive a probability vector, not a hard label. Mixture models, Latent Dirichlet Allocation–style mixed-membership models, and soft k-means all support this.

The decision-support layer downstream should use the probability vector, not the argmax. A 51%/49% split between two subtypes that recommend opposite treatments is the wrong place for a hard cutoff.

## The replication crisis in subtype discovery

A grim observation: most published clinical-subtype studies do not externally replicate. Common culprits:

- $K$ chosen post-hoc on the same data used for clustering.
- Feature subsets that maximise apparent separation.
- Single-cohort discovery without an independent cohort.
- Outcome contamination — outcome variables sneaking into the clustering features.

The expected lifetime of a subtype paper that does not include external replication is short. PrecisionStack's default position: until a subtyping has replicated across at least two independent cohorts with consistent biological correlates, treat it as a working hypothesis.

## A worked example — multi-omic subtyping in breast cancer

1. **Features.** Bulk RNA-seq + DNA methylation + somatic mutations for $n = 1500$ patients.
2. **Method.** Multi-omics NMF (iCluster, SNF, or MOFA).
3. **$K$ selection.** Bootstrap stability + cophenetic correlation.
4. **Result.** Four subtypes with distinct biology and PFS curves.
5. **External replication.** TCGA + METABRIC; subtypes recover with > 0.8 ARI.
6. **Clinical correlate.** PFS curves separate, HER2 status concentrates in one cluster, immune signature concentrates in another.
7. **Actionability.** Cheap supervised classifier from IHC to subtype.

## Exercises

1. **Reproduce a published subtyping** (e.g. PAM50, AD subtypes from ADNI). Bootstrap-stability test the partition.
2. **Compare k-means, mixture models, and SuStaIn** on a synthetic disease-progression dataset.
3. **CATE-based subgrouping.** On an RCT dataset, fit a causal forest, partition by predicted-benefit decile, validate the per-decile CATE on a held-out split.

## References

1. Brunet JP, et al. Metagenes and molecular pattern discovery using matrix factorization. *PNAS.* 2004;101(12):4164–4169.
2. Young AL, et al. Uncovering the heterogeneity and temporal complexity of neurodegenerative diseases with Subtype and Stage Inference. *Nat Commun.* 2018;9:4273.
3. Argelaguet R, et al. Multi-Omics Factor Analysis (MOFA). *Mol Syst Biol.* 2018;14(6):e8124.
4. Wang B, et al. Similarity network fusion for aggregating data types on a genomic scale. *Nat Methods.* 2014;11(3):333–337.
5. Athey S, Imbens GW. Recursive partitioning for heterogeneous causal effects. *PNAS.* 2016;113(27):7353–7360.

## Where to next

[Evaluation and calibration](evaluation.md) — the rubric for assessing the models from every previous chapter.
