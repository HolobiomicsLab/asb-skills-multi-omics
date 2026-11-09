---
name: training-dynamics-confidence-score-interpretation
description: Use when an annotated AnnData object (e.g. EMT, mSTZ, PBMC, mouse small-intestine)
  has been passed through a DNN with per-epoch predictions recorded, and per-cell
  confidence/variability scores must be computed, interpreted, and classified to identify
  mislabeled or ambiguous cells.
when_to_use_negative:
- Input is convolved or aggregated data, such as that obtained from certain spatial
  transcriptomics technologies — Annotatability does not directly handle it
- Annotations are probabilistic or continuous — Annotatability currently treats input
  annotations as nominal categories only
- The goal is comparative analysis of global dataset properties based on collective
  training dynamics — not yet implemented, deferred to future work
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Annotatability
  role: Provides models.follow_training_dyn_neural_net and models.probability_list_to_confidence_and_var
    to train the DNN, record per-epoch predictions, and compute confidence/variability
    scores
  repo: https://github.com/nitzanlab/Annotatability
- name: PyTorch
  role: Backend used to perform the DNN training procedure on CPU or CUDA device
- name: Scanpy
  role: AnnData container holding adata.obs['conf'] and adata.obs['var'] scores for
    annotated single-cell data
  repo: https://scanpy.readthedocs.io/en/stable/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:56ddfc2d27b8ee1297e43dd2a0fd6b4db4c2627cdd835a7112bd98af2691d7db
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
  - scipy
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# training-dynamics-confidence-score-interpretation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Interpret DNN training dynamics on annotated single-cell data by computing per-cell confidence (average probability assigned to the correct label across epochs) and variability (standard deviation of that probability), then categorizing cells as easy to learn, hard to learn, or ambiguous to flag annotation mismatches and biological structure.

## When to use

Use when an annotated AnnData object (e.g. EMT, mSTZ, PBMC, mouse small-intestine) has been passed through a DNN with per-epoch predictions recorded, and per-cell confidence/variability scores must be computed, interpreted, and classified to identify mislabeled or ambiguous cells.

## When NOT to use

- Input is convolved or aggregated data, such as that obtained from certain spatial transcriptomics technologies — Annotatability does not directly handle it
- Annotations are probabilistic or continuous — Annotatability currently treats input annotations as nominal categories only
- The goal is comparative analysis of global dataset properties based on collective training dynamics — not yet implemented, deferred to future work

## Inputs

- Annotated AnnData object (adata) with a label key (e.g. adata.obs['label']) and preprocessed expression matrix (EMT data filtered: <5% mitochondrial counts, <4000 expressed genes, ≥200 expressed genes; 3,000 highly variable genes retained)
- Hyperparameter settings: epoch_num (e.g. 50), device ('cpu' or 'cuda:0'), optional lr, momentum, batch_size, num_layers, weighted_sampler

## Outputs

- prob_list: per-epoch prediction probabilities from the DNN training run
- adata.obs['conf']: per-cell confidence score (average probability assigned to the correct label across epochs)
- adata.obs['var']: per-cell variability score (standard deviation of that probability across epochs)
- Three-way cell classification: easy to learn (correctly annotated), hard to learn (erroneously annotated), ambiguous

## How to apply

Train a neural network on the annotated AnnData via Annotatability's models.follow_training_dyn_neural_net, recording the prediction probability after each epoch (epoch_num is user-set, e.g. 50, with hyperparameters lr=0.001, momentum=0.9, batch_size=256, num_layers=3, weighted_sampler=True). Convert the probability list with models.probability_list_to_confidence_and_var into per-cell confidence and variability, storing them as adata.obs['conf'] and adata.obs['var']. Interpret the joint scores: high confidence and low variability indicate correctly annotated (easy-to-learn) cells; low confidence and low variability indicate probably mislabeled (hard-to-learn) cells; moderate confidence with high variability indicates ambiguous intermediate states. Classify each cell into one of these three categories, then optionally filter or reannotate the hard-to-learn cells; note that score distributions differ by annotation class (e.g. mSTZ annotated by treatment shows larger spread and fewer easy-to-learn cells than annotation by cell type).

## Related tools

- **Annotatability** (Provides models.follow_training_dyn_neural_net and models.probability_list_to_confidence_and_var to train the DNN, record per-epoch predictions, and compute confidence/variability scores) — https://github.com/nitzanlab/Annotatability
- **PyTorch** (Backend used to perform the DNN training procedure on CPU or CUDA device)
- **Scanpy** (AnnData container holding adata.obs['conf'] and adata.obs['var'] scores for annotated single-cell data) — https://scanpy.readthedocs.io/en/stable/

## Examples

```
prob_list = models.follow_training_dyn_neural_net(adata, label_key='label',iterNum=50, device=device); all_conf, all_var = models.probability_list_to_confidence_and_var(prob_list, n_obs=adata.n_obs, epoch_num=50); adata.obs['conf']=all_conf.detach().numpy(); adata.obs['var']=all_var.detach().numpy()
```

## Evaluation signals

- adata.obs['conf'] and adata.obs['var'] are populated with numeric scores whose shapes equal adata.n_obs (all_conf/all_var detached from tensors via .detach().numpy())
- Score distributions differ by annotation class in the expected direction (e.g. condition-based annotation of mSTZ shows larger spread and lower proportion of easy-to-learn cells than cell-type annotation)
- In small-intestine data, stem-cell labels should show lower confidence / more difficulty than labels of more differentiated cells
- Cells separate into the three expected categories: high conf + low var (correct), low conf + low var (mislabeled), moderate conf + high var (ambiguous)

## Limitations

- Thresholding on confidence and variability scores for categorizing introduces arbitrary cutoffs that could influence data interpretation
- Hyperparameters such as training time (epoch_num) are not standardized and may influence the confidence and variability scores
- Percentile q for hard-to-learn thresholds is chosen dataset-dependently (25%–90%), a manual, non-standardized choice
- Additional training-dynamics measures beyond confidence and variability are not yet incorporated; deferred to future work

## Evidence

- [discussion] Confidence is defined as the average probability assigned by the model to the correct label across all epochs; variability is the standard deviation of that probability across epochs.: [evidence span withheld — anchor status: not_found]
- [discussion] the use of thresholding ... on these scores for categorizing introduces arbitrary cutoffs that could influence the interpretation of the data: [evidence span withheld — anchor status: not_found]
