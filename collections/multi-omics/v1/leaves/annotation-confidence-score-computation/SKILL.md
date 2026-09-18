---
name: annotation-confidence-score-computation
description: Use when annotated scRNA-seq or spatial omics AnnData (with labels in
  adata.obs, e.g. 'label') needs per-cell trainability scores to audit annotations,
  find mislabeled or intermediate cells, or order cells along a transition.
when_to_use_negative:
- Input annotations are probabilistic or continuous rather than nominal categories
  — Annotatability currently treats input annotations as nominal categories only
- Data are convolved or aggregated, such as from certain spatial transcriptomics technologies,
  which the method does not directly handle
- The goal is a standard neighbors graph for UMAP/leiden — note that sc.pp.neighbors
  overwrites adata.obsp['connectivities'], clobbering the trainability-aware graph
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Annotatability
  role: Provides models.follow_training_dyn_neural_net and models.probability_list_to_confidence_and_var
    to train the DNN and derive confidence/variability scores
  repo: https://github.com/nitzanlab/Annotatability
- name: PyTorch
  role: Backend used to perform the neural network training procedure and provide
    the device (cpu/cuda) for training
- name: Scanpy
  role: Base package the code is built on for AnnData handling, preprocessing, and
    visualization (sc.tl.umap, sc.pl.umap of 'conf')
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:a94c4b04d459585b358e83de8ea96c2c0c4f087e7106fe19623068cee01e8d4e
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# annotation-confidence-score-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-cell annotation confidence and variability scores from the training dynamics of a deep neural network trained on annotated single-cell data, using the Annotatability package. Confidence is the average probability assigned to the correct label across epochs, and these scores reflect each cell's 'goodness-of-match' to its annotation.

## When to use

Use when annotated scRNA-seq or spatial omics AnnData (with labels in adata.obs, e.g. 'label') needs per-cell trainability scores to audit annotations, find mislabeled or intermediate cells, or order cells along a transition.

## When NOT to use

- Input annotations are probabilistic or continuous rather than nominal categories — Annotatability currently treats input annotations as nominal categories only
- Data are convolved or aggregated, such as from certain spatial transcriptomics technologies, which the method does not directly handle
- The goal is a standard neighbors graph for UMAP/leiden — note that sc.pp.neighbors overwrites adata.obsp['connectivities'], clobbering the trainability-aware graph

## Inputs

- Annotated AnnData (adata) with cell labels stored in an observation column, e.g. adata.obs['label']
- Number of training epochs (e.g. epoch_num=50)
- Device specification ('cpu' or 'cuda:0')

## Outputs

- adata.obs['conf']: per-cell confidence score (average probability of the correct label across epochs)
- adata.obs['var']: per-cell variability score (standard deviation of that probability across epochs)
- prob_list: list of per-epoch DNN prediction probabilities

## How to apply

Load the annotated AnnData (e.g. the EMT dataset of MCF10A and HuMEC cells, mock vs TGF-β, low-quality cells already filtered for ≥5% mitochondrial counts, >4,000 expressed genes, or <200 expressed genes), keeping genes expressed in >3 cells and 3,000 highly variable genes with each gene's expression scaled to variance 1 (L2 normalization). Train the DNN with models.follow_training_dyn_neural_net(adata, label_key='label', iterNum=50, device=device), using the documented defaults lr=0.001, momentum=0.9, weighted_sampler=True, batch_size=256, num_layers=3, recording the prediction probability after each epoch. Convert the probability list via models.probability_list_to_confidence_and_var(prob_list, n_obs=adata.n_obs, epoch_num=epoch_num) and store the outputs as adata.obs['conf'] and adata.obs['var']. High confidence with low variability marks easy-to-learn (probably correctly labeled) cells; low confidence with low variability marks probably mislabeled cells; moderate confidence with high variability marks ambiguous states. Validate by checking the scores reproduce known structure, e.g. Spearman correlations of epithelial/mesenchymal marker means with confidence of 0.410/−0.421 in the EMT dataset.

## Related tools

- **Annotatability** (Provides models.follow_training_dyn_neural_net and models.probability_list_to_confidence_and_var to train the DNN and derive confidence/variability scores) — https://github.com/nitzanlab/Annotatability
- **PyTorch** (Backend used to perform the neural network training procedure and provide the device (cpu/cuda) for training)
- **Scanpy** (Base package the code is built on for AnnData handling, preprocessing, and visualization (sc.tl.umap, sc.pl.umap of 'conf'))

## Examples

```
from Annotatability import models; prob_list = models.follow_training_dyn_neural_net(adata, label_key='label', iterNum=50, device=device); all_conf, all_var = models.probability_list_to_confidence_and_var(prob_list, n_obs=adata.n_obs, epoch_num=50); adata.obs['conf'] = all_conf.detach().numpy(); adata.obs['var'] = all_var.detach().numpy()
```

## Evaluation signals

- adata.obs contains numeric 'conf' and 'var' columns of length n_obs after probability_list_to_confidence_and_var
- Cells with high confidence and low variability should correspond to probably correctly labeled cells, and low-confidence/low-variability cells to probably mislabeled cells, as expected from the score definitions
- Reproduction check: for EMT epithelial-annotated cells, mean epithelial marker expression correlates with confidence at Spearman ρ=0.410 and mesenchymal markers at ρ=−0.421 (Fig. 4e,f); mesenchymal cells give 0.606/−0.210
- epoch_num passed to both follow_training_dyn_neural_net and probability_list_to_confidence_and_var must match (e.g. 50), and all documented hyperparameters stay at defaults unless deliberately changed

## Limitations

- Hyperparameters such as training time (iterNum) influence confidence and variability scores and their determination is not standardized
- Thresholding on confidence and variability scores to classify cells introduces arbitrary cutoffs that could influence data interpretation
- The hard-to-learn percentile q is chosen dataset-dependently between 25% and 90%, a manual, non-standardized choice
- No PyTorch version is reported, limiting reproducibility of the training procedure

## Evidence

- [discussion] we define confidence, $$\widehat{{u}_{i}}$$, as the average probability assigned by the model to the correct label: [evidence span withheld — anchor status: not_found]
- [discussion] cells with high confidence and low variability (easy to learn) probably have correctly labeled annotations: "cells with high confidence and low variability (easy to learn) probably have correctly labeled annotations"
- [methods] prob_list = models.follow_training_dyn_neural_net(adata, label_key='label',iterNum=epoch_num, device=device): "prob_list = models.follow_training_dyn_neural_net(adata, label_key='label',iterNum=epoch_num, device=device)"
- [methods] all_conf , all_var = models.probability_list_to_confidence_and_var(prob_list, n_obs= adata.n_obs, epoch_num=epoch_num): "all_conf , all_var = models.probability_list_to_confidence_and_var(prob_list, n_obs= adata.n_obs, epoch_num=epoch_num)"
- [discussion] As a preprocessing step, the expression level of each gene is scaled to have variance 1 (L2 normalization).: "As a preprocessing step, the expression level of each gene is scaled to have variance 1 (L2 normalization)."
