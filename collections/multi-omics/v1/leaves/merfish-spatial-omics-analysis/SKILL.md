---
name: merfish-spatial-omics-analysis
description: Use when analyzing annotated MERFISH spatial transcriptomics AnnData
  (e.g. mouse hypothalamic preoptic region with a 'label' obs column) to detect or
  fix erroneous cell-type annotations via DNN training dynamics.
when_to_use_negative:
- Data are convolved or aggregated spatial transcriptomics measurements rather than
  single-cell resolution — the article states Annotatability does not directly handle
  such data
- The goal is a standard Scanpy neighborhood/cluster analysis rather than annotation
  auditing — running sc.pp.neighbors overwrites adata.obsp['connectivities'] and replaces
  the trainability-aware graph
- Input annotations are probabilistic or continuous — Annotatability currently treats
  input annotations as nominal categories only
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3934
- http://edamontology.org/topic_3170
tools:
- name: Annotatability
  role: 'Core framework: trains a DNN on annotated MERFISH cells, computes confidence/variability
    training-dynamics scores, and reannotates erroneous cells via its reannotation
    module'
  repo: https://github.com/nitzanlab/Annotatability
- name: PyTorch
  role: Backend for training the neural network (models.follow_training_dyn_neural_net)
    on GPU/CPU with lr=0.001, momentum=0.9, batch_size=256
- name: Scanpy
  role: Base package for handling the AnnData MERFISH object and visualizing the trainability-aware
    graph (sc.tl.umap, sc.pl.umap color='conf')
  repo: https://scanpy.readthedocs.io/en/stable/
- name: scReClassify
  role: Baseline framework providing SVM, KNN and RF base classifiers for per-cell
    mislabel predictions used in the AUROC benchmark
merged_aliases:
- merfish-spatial-gene-expression-quantification
merged_alias_records:
- alias: merfish-spatial-gene-expression-quantification
  slug: merfish-spatial-gene-expression-quantification
  cosine_score: 0.8353
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:e96a3be5dc1038536eed4a496dfde6d6d9d4360ad09dfe779518944357a8ac57
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
  - scReClassify
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3934
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# merfish-spatial-omics-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies Annotatability's training-dynamics framework to spatial transcriptomics MERFISH data (mouse hypothalamic preoptic region) to benchmark mislabel detection against scReClassify baselines and to reannotate erroneously annotated cells so marker expression (e.g. Gad1) aligns with new labels.

## When to use

Use when analyzing annotated MERFISH spatial transcriptomics AnnData (e.g. mouse hypothalamic preoptic region with a 'label' obs column) to detect or fix erroneous cell-type annotations via DNN training dynamics.

## When NOT to use

- Data are convolved or aggregated spatial transcriptomics measurements rather than single-cell resolution — the article states Annotatability does not directly handle such data
- The goal is a standard Scanpy neighborhood/cluster analysis rather than annotation auditing — running sc.pp.neighbors overwrites adata.obsp['connectivities'] and replaces the trainability-aware graph
- Input annotations are probabilistic or continuous — Annotatability currently treats input annotations as nominal categories only

## Inputs

- Annotated MERFISH mouse hypothalamic preoptic region AnnData object with cell-type labels in an 'label' obs column (from github.com/nitzanlab/Annotatability_notebooks)
- Semi-synthetic perturbed copies of the AnnData with controlled mislabeling fractions (5%-50%) and a per-cell correct/incorrect ground-truth flag
- Marker-gene reference for validation (e.g. Gad1 for inhibitory neurons, Supplementary Table 1)

## Outputs

- Per-cell confidence and variability scores stored in adata.obs['conf'] and adata.obs['var']
- AUROC curves of mislabel detection versus percentage of mislabeled cells, saved as auroc_vs_mislabeled_fraction.png (one curve per method: Annotatability, scReClassify SVM/KNN/RF)
- Reannotated cell labels for erroneously annotated (hard-to-learn) cells
- Marker-alignment summary table (gad1_reannotation_results.csv) comparing mean Gad1 expression before/after reannotation

## How to apply

Load the annotated MERFISH AnnData (cell-type label stored as an observation column 'label') from the nitzanlab/Annotatability_notebooks reproducibility repository. Train the DNN while recording per-epoch predictions with models.follow_training_dyn_neural_net(adata, label_key='label', iterNum=50, lr=0.001, momentum=0.9, batch_size=256, weighted_sampler=True, num_layers=3, device=cuda if available), then convert the probability list to per-cell scores via models.probability_list_to_confidence_and_var, storing them in adata.obs['conf'] and adata.obs['var']. For the semi-synthetic AUROC benchmark, create perturbed copies by randomly reassigning 'label' for 5%-50% of cells, score Annotatability and scReClassify (SVM, KNN, RF) with the correct/incorrect flag as ground truth, and plot AUROC versus mislabeled fraction (Annotatability consistently wins, with the gap growing as more cells are mislabeled). For reannotation, classify cells as easy-to-learn (correct), hard-to-learn (erroneous) or ambiguous from the confidence/variability distributions, train the reannotation module on easy-to-learn cells to relabel erroneous ones, and validate by comparing mean normalized marker expression — e.g. Gad1 was 6.830 in correctly annotated inhibitory neurons, 1.458 in erroneous ones, and 6.795 after reannotation, confirming post-reannotation alignment with Supplementary Table 1 markers (Fig. 3j,k).

## Related tools

- **Annotatability** (Core framework: trains a DNN on annotated MERFISH cells, computes confidence/variability training-dynamics scores, and reannotates erroneous cells via its reannotation module) — https://github.com/nitzanlab/Annotatability
- **PyTorch** (Backend for training the neural network (models.follow_training_dyn_neural_net) on GPU/CPU with lr=0.001, momentum=0.9, batch_size=256)
- **Scanpy** (Base package for handling the AnnData MERFISH object and visualizing the trainability-aware graph (sc.tl.umap, sc.pl.umap color='conf')) — https://scanpy.readthedocs.io/en/stable/
- **scReClassify** (Baseline framework providing SVM, KNN and RF base classifiers for per-cell mislabel predictions used in the AUROC benchmark)

## Examples

```
from Annotatability import metrics, models
prob_list = models.follow_training_dyn_neural_net(adata, label_key='label', iterNum=50, device=device)
all_conf, all_var = models.probability_list_to_confidence_and_var(prob_list, n_obs=adata.n_obs, epoch_num=50)
```

## Evaluation signals

- auroc_vs_mislabeled_fraction.png exists and contains one AUROC curve per method (Annotatability plus scReClassify SVM/KNN/RF) across all perturbation levels (5%-50%)
- Annotatability's AUROC values are computed from adata.obs['conf'] and adata.obs['var'], and its curves sit above the scReClassify baselines, with the advantage growing as the mislabeled fraction increases
- After reannotation, mean normalized Gad1 expression of reannotated inhibitory neurons (~6.795) matches correctly annotated inhibitory neurons (6.830), not the erroneous pre-reannotation value (1.458)
- Reannotation agreement with Supplementary Table 1 marker alignment (Fig. 3j,k) for the new cell-type labels
- Confidence/variability score distributions separate easy-to-learn, hard-to-learn and ambiguous cells in the expected direction (high conf/low var = correct; low conf/low var = mislabeled)

## Limitations

- Annotatability does not directly handle convolved or aggregated data, such as that obtained from certain spatial transcriptomics technologies
- Hyperparameters such as training time (iterNum) influence confidence and variability scores and their determination needs standardization
- Thresholding on confidence/variability scores (hard-to-learn percentile q chosen between 25% and 90%, dataset-dependently) introduces arbitrary cutoffs that could influence interpretation
- No PyTorch software version is reported and no changelog exists for the repository, limiting reproducibility

## Evidence
