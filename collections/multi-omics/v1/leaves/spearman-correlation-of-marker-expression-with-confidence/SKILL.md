---
name: spearman-correlation-of-marker-expression-with-confidence
description: Use when cells carry a nominal annotation over an underlying gradient
  (e.g. EMT, development) and you need to check that DNN confidence scores track marker
  gene expression in a monotonic way.
when_to_use_negative:
- Data are convolved or aggregated, such as that from certain spatial transcriptomics
  technologies, which Annotatability does not directly handle
- The annotation is continuous or probabilistic rather than nominal categories, since
  Annotatability treats input annotations as nominal only
- The goal is differential expression between discrete annotated groups rather than
  validating confidence scores against a continuous gradient
edam_operation: http://edamontology.org/operation_0474
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Annotatability
  role: Trains the DNN (follow_training_dyn_neural_net) and converts per-epoch probabilities
    into confidence/variability scores (probability_list_to_confidence_and_var)
  repo: https://github.com/nitzanlab/Annotatability
- name: Scanpy
  role: AnnData handling, L2 normalization/variance scaling preprocessing, and diffusion-pseudotime
    baseline correlations
- name: PyTorch
  role: Backend for training the neural network classifier whose training dynamics
    produce the confidence scores
- name: scipy
  role: Computes Spearman rank correlations between marker-set means and confidence
    scores
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:2c89d5919ff4f8e71ba7fabb1d6871e5325fdc42fd18eadb9c552c91aeb9ae18
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
  edam_operation: http://edamontology.org/operation_0474
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# spearman-correlation-of-marker-expression-with-confidence

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates that Annotatability confidence scores reflect continuous biological states by computing Spearman rank correlations between per-cell marker-set mean expression (L2-normalized) and DNN training-dynamics confidence, e.g. epithelial 0.410/−0.421 and mesenchymal 0.606/−0.210 along the EMT.

## When to use

Use when cells carry a nominal annotation over an underlying gradient (e.g. EMT, development) and you need to check that DNN confidence scores track marker gene expression in a monotonic way.

## When NOT to use

- Data are convolved or aggregated, such as that from certain spatial transcriptomics technologies, which Annotatability does not directly handle
- The annotation is continuous or probabilistic rather than nominal categories, since Annotatability treats input annotations as nominal only
- The goal is differential expression between discrete annotated groups rather than validating confidence scores against a continuous gradient

## Inputs

- Annotated scRNA-seq AnnData (.h5ad) with cell-type/condition labels in adata.obs['label'] (EMT dataset: MCF10A and HuMEC, mock vs TGF-β, low-quality cells filtered, genes restricted to >3 cells and top 3,000 highly variable)
- Marker gene sets (e.g. epithelial and mesenchymal marker lists) mapped to the AnnData var_names
- Per-epoch prediction probability list from follow_training_dyn_neural_net (prob_list)

## Outputs

- Per-cell confidence (adata.obs['conf']) and variability (adata.obs['var']) scores
- Spearman rank correlation table of marker-set means vs confidence (epithelial 0.410/−0.421; mesenchymal 0.606/−0.210) and diffusion-pseudotime baseline correlations
- CSV of per-cell scores and correlations (e.g. emt_confidence_spearman.csv)

## How to apply

Load the scRNA-seq AnnData (here the EMT dataset: MCF10A and HuMEC, mock vs TGF-β, low-quality cells already filtered) with the Louvain-derived epithelial/mesenchymal annotation in adata.obs['label']. Train the neural classifier with models.follow_training_dyn_neural_net(adata, label_key='label', iterNum=50, device=device) using documented defaults (lr=0.001, momentum=0.9, weighted_sampler=True, batch_size=256, num_layers=3), then convert the probability list via models.probability_list_to_confidence_and_var into adata.obs['conf'] and adata.obs['var']. Compute per-cell mean expression of the epithelial and mesenchymal marker gene sets on variance-1-scaled (L2-normalized) expression, the article's preprocessing before trainability association. Compute Spearman rank correlations of each marker-set mean with the confidence score (Scanpy diffusion pseudotime as baseline) and compare against the reported values: for epithelial-annotated cells, epithelial markers 0.410 and mesenchymal markers −0.421 (Fig. 4e,f), with the monotonicity of the means across confidence bins as an additional check.

## Related tools

- **Annotatability** (Trains the DNN (follow_training_dyn_neural_net) and converts per-epoch probabilities into confidence/variability scores (probability_list_to_confidence_and_var)) — https://github.com/nitzanlab/Annotatability
- **Scanpy** (AnnData handling, L2 normalization/variance scaling preprocessing, and diffusion-pseudotime baseline correlations)
- **PyTorch** (Backend for training the neural network classifier whose training dynamics produce the confidence scores)
- **scipy** (Computes Spearman rank correlations between marker-set means and confidence scores)

## Examples

```
prob_list = models.follow_training_dyn_neural_net(adata, label_key='label', iterNum=50, device=device); all_conf, all_var = models.probability_list_to_confidence_and_var(prob_list, n_obs=adata.n_obs, epoch_num=50); adata.obs['conf'] = all_conf.detach().numpy()
```

## Evaluation signals

- Reproduced Spearman correlation for epithelial marker mean vs confidence in epithelial-annotated cells is close to the reported 0.410, and mesenchymal marker mean vs confidence is close to −0.421 (Fig. 4e,f)
- Mesenchymal-annotated cells give correlations near the reported 0.606 (mesenchymal markers) and −0.210 (epithelial markers)
- Binned mean epithelial marker expression increases monotonically and mean mesenchymal marker expression decreases monotonically with confidence score
- Confidence values in adata.obs['conf'] lie in [0, 1] and equal the average probability of the correct label across all epochs; variability is the standard deviation across epochs
- Diffusion-pseudotime baseline correlations are computed alongside for comparison, and outputs are written to emt_confidence_spearman.csv

## Limitations

- Hyperparameters such as training time (epoch_num) are not standardized and may influence the confidence and variability scores, hence the correlations
- Percentile thresholds (q, 25%–90%) used for categorizing cells are dataset-dependent manual choices
- Thresholding on confidence and variability introduces arbitrary cutoffs that could influence interpretation of the gradient
- Annotatability treats input annotations as nominal categories only, so the continuous state is probed only post hoc via marker correlations

## Evidence

- [results] For cells originally labeled as epithelial, mean epithelial marker gene expression increases monotonically and mean mesenchymal marker expression decreases monotonically with Annotatability's confidence score: [evidence span withheld — anchor status: not_found]
- [discussion] Confidence definition: [evidence span withheld — anchor status: not_found]
