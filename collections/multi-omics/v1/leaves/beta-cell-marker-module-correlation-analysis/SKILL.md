---
name: beta-cell-marker-module-correlation-analysis
description: Use when a single-cell dataset annotated by condition (e.g. healthy vs
  mSTZ) yields per-cell confidence scores and you want to test whether confidence
  tracks a graded molecular state, via Spearman correlation against gene-module scores.
when_to_use_negative:
- Input annotations are continuous or probabilistic states rather than nominal categories
  — Annotatability currently treats input annotations as nominal categories only
- Data are convolved or aggregated, e.g. from certain spatial transcriptomics technologies,
  which Annotatability does not directly handle
- The goal is differential expression rather than testing whether annotation trainability
  reflects a graded biological state; use rank_genes_conf_min_counts-based annotation-trainability
  association scoring instead
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Annotatability
  role: Trains the DNN (follow_training_dyn_neural_net) and converts the probability
    list to per-cell confidence and variability (probability_list_to_confidence_and_var)
  repo: https://github.com/nitzanlab/Annotatability
- name: PyTorch
  role: Backend for the DNN training procedure, including device selection ('cuda:0'
    if available) and weighted sampler for class imbalance
- name: Scanpy
  role: AnnData handling and single-cell preprocessing of the mSTZ pancreatic islet
    dataset
- name: scipy
  role: Spearman rank correlation (scipy.stats) between per-cell confidence scores
    and summed marker module scores
- name: pandas
  role: Writes the resulting correlation coefficients to mstz_confidence_correlations.csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:adf820d9298f67d4cd84ca16d66bd6d96429eee0db7394ba49ac4ba2a87699d0
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
  - Scanpy
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# beta-cell-marker-module-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Correlates per-cell Annotatability confidence scores with summed gene-module scores (β-cell activity markers Ins1, Ins2, Slc2a2, Trpm5, G6pc2, Slc30a8, Ucn3, and dedifferentiation markers) to test whether annotation confidence reflects graded biological severity, as demonstrated on the mSTZ diabetic mouse pancreatic islet scRNA-seq dataset.

## When to use

Use when a single-cell dataset annotated by condition (e.g. healthy vs mSTZ) yields per-cell confidence scores and you want to test whether confidence tracks a graded molecular state, via Spearman correlation against gene-module scores.

## When NOT to use

- Input annotations are continuous or probabilistic states rather than nominal categories — Annotatability currently treats input annotations as nominal categories only
- Data are convolved or aggregated, e.g. from certain spatial transcriptomics technologies, which Annotatability does not directly handle
- The goal is differential expression rather than testing whether annotation trainability reflects a graded biological state; use rank_genes_conf_min_counts-based annotation-trainability association scoring instead

## Inputs

- mSTZ pancreatic islet scRNA-seq dataset as an AnnData object (adata) with cells annotated by experimental condition in adata.obs['label'] (healthy vs mSTZ)
- Gene marker lists: β-cell activity markers (Ins1, Ins2, Slc2a2, Trpm5, G6pc2, Slc30a8, Ucn3) and dedifferentiation markers

## Outputs

- Per-cell confidence and variability stored in adata.obs['conf'] and adata.obs['var']
- Per-cell summed marker module scores (activity and dedifferentiation)
- mstz_confidence_correlations.csv containing Spearman rank correlation coefficients between confidence and each marker module score

## How to apply

Load the mSTZ pancreatic islet scRNA-seq AnnData with cells annotated by experimental condition (healthy vs mSTZ) and train the Annotatability DNN using models.follow_training_dyn_neural_net with label_key='label', 50-100 epochs (iterNum), a 3-layer network, batch size 256, and weighted_sampler=True to handle class imbalance, on GPU if available. Convert the recorded per-epoch probability list with models.probability_list_to_confidence_and_var and store per-cell confidence and variability in adata.obs['conf'] and adata.obs['var']. Compute summed module scores of β-cell activity markers (Ins1, Ins2, Slc2a2, Trpm5, G6pc2, Slc30a8, Ucn3) and of dedifferentiation markers per cell. Compute Spearman rank correlations between per-cell confidence and each module score using scipy.stats, writing the coefficients to mstz_confidence_correlations.csv.

## Related tools

- **Annotatability** (Trains the DNN (follow_training_dyn_neural_net) and converts the probability list to per-cell confidence and variability (probability_list_to_confidence_and_var)) — https://github.com/nitzanlab/Annotatability
- **PyTorch** (Backend for the DNN training procedure, including device selection ('cuda:0' if available) and weighted sampler for class imbalance)
- **Scanpy** (AnnData handling and single-cell preprocessing of the mSTZ pancreatic islet dataset)
- **scipy** (Spearman rank correlation (scipy.stats) between per-cell confidence scores and summed marker module scores)
- **pandas** (Writes the resulting correlation coefficients to mstz_confidence_correlations.csv)

## Examples

```
prob_list = models.follow_training_dyn_neural_net(adata, label_key='label', iterNum=50, device=device); all_conf, all_var = models.probability_list_to_confidence_and_var(prob_list, n_obs=adata.n_obs, epoch_num=50); adata.obs['conf'] = all_conf.detach().numpy(); rho = scipy.stats.spearmanr(adata.obs['conf'], module_score)[0]
```

## Evaluation signals

- Reproduced Spearman coefficient for confidence vs summed β-cell activity markers matches the reported Fig. 5a value of ρ = −0.662 within sampling tolerance (negative correlation expected)
- Reproduced Spearman coefficient for confidence vs summed dedifferentiation markers matches the reported value of ρ = 0.573 (positive correlation expected)
- Confidence values are bounded in [0, 1] and stored for all n_obs cells after probability_list_to_confidence_and_var
- Diffusion-pseudotime baseline coefficients are computed alongside for comparison against the confidence-based correlations

## Limitations

- Hyperparameters such as training time (iterNum) are not standardized and may influence the confidence and variability scores
- Thresholding on confidence and variability scores introduces arbitrary cutoffs that could influence data interpretation
- Annotatability currently treats input annotations as nominal categories only; generalization to probabilistic and continuous annotations is deferred to future work

## Evidence

- [other] Annotatability confidence scores for the mSTZ annotation are negatively correlated with summed β cell activity marker expression (Ins1, Ins2, Slc2a2, Trpm5, G6pc2, Slc30a8, Ucn3), with Spearman's ρ = −0.662.: [evidence span withheld — anchor status: not_found]
