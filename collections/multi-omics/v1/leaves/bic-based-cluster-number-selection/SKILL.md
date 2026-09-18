---
name: bic-based-cluster-number-selection
description: Use when fitting a LUCID integrated-clustering model (exposures G, omics
  Z, outcome Y) and the number of latent unknown clusters K is unknown and must be
  determined from the data.
when_to_use_negative:
- When K is fixed by study design or prior knowledge (use estimate_lucid() directly
  with a scalar K instead of tuning)
- When using parallel or serial LUCID models where K must mirror the omics-layer topology
  as a vector or nested list, not a single scalar
- When the goal is variable selection rather than cluster-count selection (use the
  lasso/graphical-lasso regularization options instead)
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0091
tools:
- name: LUCIDus
  role: Provides lucid() which automatically performs model selection on the number
    of clusters K; tune_lucid() evaluates grids of K values
  repo: https://github.com/USCbiostats/LUCIDus
- name: mclust
  role: Default method used with regression to initialize LUCID parameters before
    EM fitting for each candidate K
- name: R
  role: Environment in which the LUCIDus package and BIC model selection are run
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbae123
    title: An extension of latent unknown clustering integrating multi-omics data
      (LUCID) incorporating incomplete omics data
  source_content_hash: sha256:6879cb584b140d9e375e653f75e0c85ece61e42bd39150fb638fc6f3dc6ff597
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/USCbiostats/LUCIDus
  tools:
  - LUCIDus
  - R
  - mclust
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# BIC-based cluster number selection

## Summary

Selects the optimal number of latent clusters K in a LUCID model by fitting candidate models and choosing the minimum Bayesian Information Criterion (BIC), as automated by the lucid() function in LUCIDus.

## When to use

Use when fitting a LUCID integrated-clustering model (exposures G, omics Z, outcome Y) and the number of latent unknown clusters K is unknown and must be determined from the data.

## When NOT to use

- When K is fixed by study design or prior knowledge (use estimate_lucid() directly with a scalar K instead of tuning)
- When using parallel or serial LUCID models where K must mirror the omics-layer topology as a vector or nested list, not a single scalar
- When the goal is variable selection rather than cluster-count selection (use the lasso/graphical-lasso regularization options instead)

## Inputs

- Exposure matrix G (e.g., maternal hexachlorobenzene levels) from the Exposome Data Challenge 2021 dataset
- Omics matrix Z (proteomics from 1301 children, complete-case or imputed)
- Outcome vector Y (childhood BMI z-score)
- Candidate grid of cluster counts K (integer or integer vector)

## Outputs

- Fitted LUCID model object at the BIC-optimal K
- BIC values across candidate K values (model-selection table)
- Cluster assignments and cluster-specific parameter estimates via summary_lucid()

## How to apply

Fit LUCID models with lucid() (or estimate_lucid()/tune_lucid() in LUCIDus 3.x) across a grid of candidate cluster counts rather than fixing K a priori. lucid() automatically conducts model selection on the number of clusters K, evaluating each candidate with BIC and returning the best-fitting model; mclust-based regression is the default initialization for parameter estimation via the EM algorithm. For the HCB–BMI exposome application, K is tuned over the proteomics-informed latent structure linking maternal hexachlorobenzene to childhood BMI z-score, and the selected K defines the clusters whose outcome means (e.g., scaled BMI z-score of -1.78 vs -1.52) are then interpreted. Rationale: BIC balances likelihood against model complexity, guarding against over-clustered solutions in the EM-based mixture. After selection, confirm the chosen K with summary_lucid() parameter tables before downstream interpretation or comparison (e.g., against complete-case analyses).

## Related tools

- **LUCIDus** (Provides lucid() which automatically performs model selection on the number of clusters K; tune_lucid() evaluates grids of K values) — https://github.com/USCbiostats/LUCIDus
- **mclust** (Default method used with regression to initialize LUCID parameters before EM fitting for each candidate K)
- **R** (Environment in which the LUCIDus package and BIC model selection are run)

## Examples

```
library(LUCIDus); fit <- lucid(G = hcb_exposure, Z = proteomics, Y = bmi_z, K = 1:5, seed = 1008); summary_lucid(fit)
```

## Evaluation signals

- The returned model object reports the K with minimum BIC across the candidate grid, not a user-imposed K
- summary_lucid() on the selected model yields coherent cluster-specific exposure effects, omics profiles, and outcome distributions
- In the HCB–BMI application, two clusters are recovered with distinguishable outcome means (cluster 1 = -1.78 vs cluster 2 = -1.52 scaled BMI z-score)
- The EM algorithm converges for each candidate K without numerical instability (check_and_stabilize_sigma()/check_convergence() helpers in LUCIDus 3.x)
- Downstream comparisons (e.g., complete-case vs integrated-imputation Model 1) reproduce consistent cluster structure under the selected K

## Limitations

- BIC selection is only as reliable as the initialization; mclust-based regression initialization can affect which local optimum the EM finds
- The published article's discussion section is absent ('No changelog found.'), so documented limitations of the BIC selection approach are not extractable
- In LUCIDus 3.x, K must mirror the model topology — vectors for parallel models and nested lists for serial models — so a single BIC scan over scalar K does not generalize directly

## Evidence

- [intro] lucid() automatically conducts model selection on number of clusters K: "`lucid()` will automatically conduct model selection on number of clusters `K`, select informative variables in `G` or `Z`, or both"
- [intro] mclust default initialization: "The first method (also the default method) uses `mclust` and regression to initialize parameters."
- [intro] EM algorithm for MLE: "we use the Expectation-Maximization (EM) algorithm to obtain the maximum likelihood estimates (MLE) of LUCID."
- [intro] Summary of fitted model: "`summary_lucid()` summarizes the fitted LUCID model by producing summary tables of parameter estimation and interpretation."
- [readme] Tuning K via tune_lucid/lucid: [evidence span withheld — anchor status: not_found]
- [abstract] Simulation support for integrated approach: "Simulation studies demonstrate that our integrated imputation approach produces consistent and less biased estimates, closely reflecting true underlying values."
