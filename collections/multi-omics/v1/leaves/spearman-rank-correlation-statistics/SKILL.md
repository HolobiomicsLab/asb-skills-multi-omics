---
name: spearman-rank-correlation-statistics
description: Use when you have per-cell continuous scores (Annotatability confidence/variability
  stored in adata.obs) plus marker module scores and need a rank-based, outlier-robust
  association coefficient to test whether training-dynamics confidence tracks graded
  cell state.
when_to_use_negative:
- When the annotation variable is nominal only — Annotatability treats input annotations
  as nominal categories, and the article defers generalization to continuous annotations
  to future work, so correlation against confidence for continuous labels is out of
  scope
- When a linear (Pearson) relationship with Gaussian-distributed scores is expected
  and no rank-based robustness is needed — Spearman discards magnitude information
- When the data are convolved or aggregated (e.g. certain spatial transcriptomics
  technologies), since Annotatability does not directly handle such data and confidence
  scores cannot be validly computed per cell
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: scipy
  role: scipy.stats provides the Spearman rank correlation test between per-cell confidence
    and marker module scores
- name: Annotatability
  role: Produces the per-cell confidence/variability scores (via follow_training_dyn_neural_net
    and probability_list_to_confidence_and_var) that are one side of the correlation
  repo: https://github.com/nitzanlab/Annotatability
- name: pandas
  role: Assembles per-cell confidence and module scores into a DataFrame and writes
    the coefficients to mstz_confidence_correlations.csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:61c7fa6ac3867828512f3fcd099236d454b4c2b305f695c63a27a888da4c251b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/nitzanlab/Annotatability
  repo_url: https://github.com/nitzanlab/Annotatability
  tools:
  - Annotatability
  - PyTorch
  - Python
  - numpy
  - pandas
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# spearman-rank-correlation-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Use scipy Spearman rank correlation to quantify monotonic association between per-cell Annotatability confidence scores and continuous biological scores (e.g. summed β-cell activity or dedifferentiation marker module scores) in scRNA-seq data, as done for the mSTZ diabetic islet analysis.

## When to use

Use when you have per-cell continuous scores (Annotatability confidence/variability stored in adata.obs) plus marker module scores and need a rank-based, outlier-robust association coefficient to test whether training-dynamics confidence tracks graded cell state.

## When NOT to use

- When the annotation variable is nominal only — Annotatability treats input annotations as nominal categories, and the article defers generalization to continuous annotations to future work, so correlation against confidence for continuous labels is out of scope
- When a linear (Pearson) relationship with Gaussian-distributed scores is expected and no rank-based robustness is needed — Spearman discards magnitude information
- When the data are convolved or aggregated (e.g. certain spatial transcriptomics technologies), since Annotatability does not directly handle such data and confidence scores cannot be validly computed per cell

## Inputs

- AnnData object with per-cell Annotatability confidence and variability in adata.obs['conf'] and adata.obs['var']
- Gene expression matrix (Scanpy-compatible) containing the marker genes Ins1, Ins2, Slc2a2, Trpm5, G6pc2, Slc30a8, Ucn3
- Marker gene lists for activity and dedifferentiation modules

## Outputs

- Spearman ρ coefficients per marker module (target: ρ = −0.662 for activity markers, ρ = 0.573 for dedifferentiation markers in mSTZ)
- Correlation table written to mstz_confidence_correlations.csv

## How to apply

First obtain per-cell confidence and variability by training the Annotatability DNN with models.follow_training_dyn_neural_net (label_key='label', iterNum=50–100 epochs, batch_size=256, num_layers=3, weighted_sampler=True for class imbalance) and converting the probability list via models.probability_list_to_confidence_and_var into adata.obs['conf'] and adata.obs['var']. Next compute per-cell module scores by summing marker expression: β-cell activity markers (Ins1, Ins2, Slc2a2, Trpm5, G6pc2, Slc30a8, Ucn3) and dedifferentiation markers. Then call scipy.stats.spearmanr between the confidence column and each marker module score to get ρ. Rank-based correlation is chosen because confidence is bounded in [0,1] and module scores are skewed, so monotonic (not linear) association is the right model. Validate by comparing reproduced ρ against reported Fig. 5a values within sampling tolerance.

## Related tools

- **scipy** (scipy.stats provides the Spearman rank correlation test between per-cell confidence and marker module scores)
- **Annotatability** (Produces the per-cell confidence/variability scores (via follow_training_dyn_neural_net and probability_list_to_confidence_and_var) that are one side of the correlation) — https://github.com/nitzanlab/Annotatability
- **pandas** (Assembles per-cell confidence and module scores into a DataFrame and writes the coefficients to mstz_confidence_correlations.csv)

## Examples

```
from scipy.stats import spearmanr; rho, p = spearmanr(adata.obs['conf'], module_score)
```

## Evaluation signals

- Reproduced Spearman ρ for summed β-cell activity markers matches the reported Fig. 5a value of −0.662 within sampling tolerance
- Reproduced Spearman ρ for dedifferentiation markers matches the reported value of 0.573 within sampling tolerance
- Correlation coefficients for the diffusion-pseudotime baseline are also computed and compared as a sanity check
- adata.obs['conf'] and adata.obs['var'] are populated for all n_obs cells and confidence values lie in [0,1]
- The output CSV contains one row per marker module with coefficient columns matching the reported schema

## Limitations

- Spearman coefficients are sampling-dependent; the article reports validation only 'within sampling tolerance', so exact reproduction is not guaranteed
- Confidence and variability scores depend on hyperparameters such as training time (iterNum) which are not standardized and may influence the downstream correlations
- Annotatability treats annotations as nominal categories; correlation of confidence with continuous biological variables (like pseudotime or module scores) is an interpretive extension rather than a supported annotation type

## Evidence

- [other] Annotatability confidence scores for the mSTZ annotation are negatively correlated with summed β cell activity marker expression (Ins1, Ins2, Slc2a2, Trpm5, G6pc2, Slc30a8, Ucn3), with Spearman's ρ = −0.662.: [evidence span withheld — anchor status: not_found]
- [other] Compute Spearman rank correlations between the per-cell confidence scores and each marker module score using scipy.stats.: [evidence span withheld — anchor status: not_found]
- [discussion] we define confidence, $$\widehat{{u}_{i}}$$, as the average probability assigned by the model to the correct label: [evidence span withheld — anchor status: not_found]
