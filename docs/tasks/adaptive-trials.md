# G. Adaptive clinical trials

> *An adaptive trial is one that uses accumulating data, by pre-specified rules, to change its own conduct. AI fits naturally into adaptive designs because both rest on Bayesian updating, decision-theoretic stopping, and explicit modelling of treatment-effect heterogeneity.*

## What "adaptive" actually means

The FDA's [2019 guidance on adaptive designs](https://www.fda.gov/regulatory-information/search-fda-guidance-documents/adaptive-design-clinical-trials-drugs-and-biologics-guidance-industry) defines an adaptive design as

> *a clinical trial design that allows for prospectively planned modifications to one or more aspects of the design based on accumulating data from subjects in the trial.*

The keyword is **prospectively planned**. Every adaptive trial must pre-specify:

- The interim analyses (timing, decision rules).
- The data-driven modifications they may trigger.
- The statistical machinery that preserves type-I error in the presence of adaptation.

Modifications that are not pre-specified are *post-hoc* and forfeit the trial's regulatory weight.

## What can adapt

- **Sample size re-estimation.** Continue enrolling if early data suggest the effect is smaller than expected.
- **Allocation ratio.** Shift more patients toward the better-performing arm (response-adaptive randomisation).
- **Dropping arms.** Discontinue a treatment that has crossed a futility boundary.
- **Enrichment / population restriction.** Narrow eligibility to a biomarker-defined subgroup whose treatment effect is now estimated to be larger.
- **Adding arms.** Bring in a new candidate therapy as the trial proceeds (platform trials).
- **Switching primary endpoint.** Rare but permitted in carefully constructed designs.

## The statistical framework

The classical framework is **group-sequential**: compute an interim test statistic, compare it to pre-computed boundaries (O'Brien-Fleming, Pocock, etc.), make a binary decision.

The modern framework is **Bayesian adaptive**: maintain a posterior over treatment effects, update it at each interim, decide on the basis of posterior probabilities of efficacy / futility / dose. The Bayesian posterior is a natural place to plug in AI/ML — particularly for CATE-aware enrichment.

### Bayesian adaptive randomisation (BAR)

At each enrolment, assign probabilities of allocation proportional to the posterior probability that each arm is the best. Concrete: $\pi_k \propto \Pr(\theta_k = \max_j \theta_j \mid \text{data})$. This shifts patients toward the better-performing arm without ever crossing a frequentist boundary; the type-I error is controlled by simulation, not closed-form.

Strengths: more patients on the winning arm; faster effective stopping.

Limitations: requires reliable interim data (low noise relative to true effect), can over-allocate if drift exists across time, can mask time-trends without careful adjustment.

References: [Berry SM et al., 2010, *Bayesian Adaptive Methods for Clinical Trials*](https://www.routledge.com/Bayesian-Adaptive-Methods-for-Clinical-Trials/Berry-Carlin-Lee-Muller/p/book/9781439825488).

### Response-adaptive designs

A broader umbrella that includes BAR, drop-the-loser, play-the-winner, and CARA (covariate-adjusted response-adaptive).

### Platform trials and master protocols

A single protocol, multiple therapies, shared infrastructure, shared control. Therapies enter and leave as data accumulates. **STAMPEDE** (prostate cancer), **REMAP-CAP** (severe pneumonia / COVID), **I-SPY 2** (breast cancer), **GBM AGILE** (glioblastoma) are the canonical examples.

The key statistical objects:

- A shared control arm whose allocation can be larger than any single experimental arm.
- An umbrella for biomarker-defined sub-populations (basket trials).
- A protocol for adding arms that does not require restarting the trial.

Reference: [Woodcock & LaVange, NEJM 2017](https://www.nejm.org/doi/full/10.1056/NEJMra1510062).

### Basket and umbrella trials

- **Basket.** One biomarker, many diseases. *Does drug X work across cancers sharing mutation Y?*
- **Umbrella.** One disease, many biomarkers. *Across NSCLC subtypes, which drug works for which subtype?*

Both are special cases of master-protocol designs.

## Where AI fits

AI shows up in adaptive trials at four points:

1. **CATE-driven enrichment.** A CATE model trained on the run-in data narrows future enrolment to predicted-responders.
2. **Trial-matching scale-up.** Adaptive trials with multiple arms create combinatorial eligibility logic; an NLP-driven matcher is essential.
3. **Bayesian posteriors over treatment effects.** A natural place for ML-based outcome models to enter, in particular for surrogate endpoints with strong-prior models.
4. **Synthetic-control augmentation.** External control data can be borrowed dynamically, with Bayesian dynamic borrowing controlling how much weight the external control receives based on its concordance with concurrent control.

## A worked design — small-sample adaptive epilepsy device trial

A novel responsive-neurostimulation device for drug-resistant focal epilepsy. The disease is rare enough that a 1000-patient RCT is impractical.

- **Design.** Bayesian adaptive, two arms, shared concurrent control + dynamically-borrowed historical control.
- **Initial allocation.** 1:1 between device and standard medical management.
- **Interim analyses.** Every 30 enrolments.
- **Adaptations.**
    - Sample-size re-estimation up to 300 patients if posterior of efficacy is between 0.6 and 0.9.
    - Response-adaptive randomisation kicks in after $n = 90$ such that allocation tracks the posterior probability of being the better arm.
    - Futility stop if $\Pr(\tau > 0 \mid \text{data}) < 0.05$.
    - Early efficacy stop if $\Pr(\tau > 0 \mid \text{data}) > 0.99$ and the posterior median $\tau$ exceeds the minimal clinically important difference.
- **Historical borrowing.** A modified power-prior or commensurate-prior on historical RNS data, with the borrowing weight $\alpha$ determined by the commensurability test.
- **Type-I error.** Simulated at 5%; full simulation report attached to the protocol.
- **Stopping for harm.** Independent DSMB charter; pre-specified harm boundaries.

This is the kind of trial that 10 years ago would not have been considered analytically tractable. Today, with the Bayesian and computational machinery in place, it is the new norm in rare-disease device development.

## What you have to pre-specify

A reviewable adaptive-trial SAP includes:

- The posterior model (likelihood + priors).
- The decision boundaries at every interim.
- The simulation report establishing type-I error and operating characteristics across the realistic effect-size grid.
- The allocation algorithm.
- The borrowing prior (if any) and its data-dependent attenuation rule.
- The DSMB charter and information-sharing rules.
- The plan for analysis of partial data if the trial is stopped early.

## Operating characteristics

A pre-specified simulation is the only way to evaluate a Bayesian adaptive design.

- Bias and root-mean-square error of the treatment-effect estimate.
- Type-I error and power across the effect-size grid.
- Expected sample size.
- Expected number of patients allocated to the inferior arm.
- Probability of correctly selecting the best arm.

The simulation grid should include the *worst-case* scenarios — time trends in patient mix, drift in standard of care, mis-specified prior — not just the design's intended sweet spot.

## Common failures

- **Type-I error inflation** from unaccounted-for interim looks.
- **Time-trend confounding** when response-adaptive randomisation interacts with secular changes in patient mix.
- **Over-borrowing** from a historical cohort whose actual effect differs from the concurrent control.
- **Operational fragility** — the EDC, RTSM, and lab pipelines must support frequent interim analyses.

## Where this fits the precision-medicine vision

An adaptive trial is the closest thing to *running the patient-as-a-trajectory model live*. The trial is the system; the data accumulates; the design updates; the next patient's allocation depends on what has been learned from previous patients. The CATE is estimated on-the-fly and used to enrich enrolment for the population most likely to benefit. This is the destination state of the discipline — slow trials with fixed inclusion criteria are increasingly the legacy choice.

## Exercises

1. **Implement a single-arm Bayesian design** with predictive probability of success and a pre-specified futility boundary. Simulate operating characteristics.
2. **Compare a fixed 1:1 trial** to a response-adaptive design on the same simulated CATE. How much do you gain in expected sample size?
3. **Read the protocol of a published platform trial** (I-SPY 2, REMAP-CAP) and identify which precision-medicine objects (CATE, enrichment population, biomarker subgroups) are pre-specified.

## References

1. FDA. *Adaptive Designs for Clinical Trials of Drugs and Biologics — Guidance for Industry.* 2019.
2. Berry SM, Carlin BP, Lee JJ, Müller P. *Bayesian Adaptive Methods for Clinical Trials.* CRC Press, 2010.
3. Woodcock J, LaVange LM. Master protocols to study multiple therapies, multiple diseases, or both. *NEJM.* 2017;377:62–70.
4. Berry DA. The brave new world of clinical cancer research. *JAMA.* 2015;313(16):1571–1572.
5. Park JJH, et al. Systematic review of basket trials, umbrella trials, and platform trials. *Trials.* 2019;20:572.
6. Angus DC, et al. The REMAP-CAP randomized adaptive platform trial. *NEJM.* 2021.
7. Berry SM. Bayesian dynamic borrowing in adaptive trials. *JAMA.* 2018;320(2):145–146.

## Where to next

[Methods](../methods/index.md) — the technical machinery that the previous chapters depended on, in fuller detail: causal inference, survival analysis, unsupervised subtyping, evaluation.
