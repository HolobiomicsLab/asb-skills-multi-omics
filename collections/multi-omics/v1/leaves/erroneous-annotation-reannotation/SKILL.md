---
name: erroneous-annotation-reannotation
description: Use when auditing annotated single-cell or spatial omics data (AnnData
  with a label obs key) for suspected mislabeled cells and you want training-dynamics-based
  reannotation plus marker-gene validation.
when_to_use_negative:
- Data are convolved or aggregated, such as from certain spatial transcriptomics technologies
  — Annotatability does not directly handle such inputs
- Input annotations are probabilistic or continuous rather than nominal categories
  — Annotatability currently treats input annotations as nominal categories only
- You only want to remove bad cells without relabeling — the optional filter-out step
  of erroneously annotated cells is the appropriate (simpler) alternative
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0621
tools:
- name: Annotatability
  role: Provides follow_training_dyn_neural_net to record per-epoch DNN predictions,
    probability_list_to_confidence_and_var to compute confidence/variability, and
    the reannotation module to relabel erroneously annotated cells
  repo: https://github.com/nitzanlab/Annotatability
- name: Scanpy
  role: Base framework for AnnData handling and visualization (e.g., sc.tl.umap, sc.pl.umap
    of conf scores) of the single-cell data
  repo: https://scanpy.readthedocs.io/en/stable/
- name: PyTorch
  role: Backend used to perform the neural-network training procedure whose dynamics
    are monitored
- name: Annotatability_notebooks
  role: Reproducibility notebooks supplying the annotated MERFISH hypothalamus dataset
    and worked reannotation examples
  repo: https://github.com/nitzanlab/Annotatability_notebooks
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:37cce74890014377f420b9a563f2349b5beab74e33c0f5b9c82df3ccb410fd04
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
  - Annotatability_notebooks
  - PyTorch
  - Python
  - Scanpy
  - numpy
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# erroneous-annotation-reannotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies erroneously annotated cells from DNN training-dynamics confidence/variability scores using Annotatability, then relabels hard-to-learn cells with the reannotation module trained on easy-to-learn cells, validating the new labels by marker-gene expression alignment (e.g., Gad1 in MERFISH hypothalamus inhibitory neurons).

## When to use

Use when auditing annotated single-cell or spatial omics data (AnnData with a label obs key) for suspected mislabeled cells and you want training-dynamics-based reannotation plus marker-gene validation.

## When NOT to use

- Data are convolved or aggregated, such as from certain spatial transcriptomics technologies — Annotatability does not directly handle such inputs
- Input annotations are probabilistic or continuous rather than nominal categories — Annotatability currently treats input annotations as nominal categories only
- You only want to remove bad cells without relabeling — the optional filter-out step of erroneously annotated cells is the appropriate (simpler) alternative

## Inputs

- Annotated AnnData object (e.g., unperturbed MERFISH hypothalamus dataset) with cell type labels stored in an observation key such as adata.obs['label']
- Epoch number for training (e.g., epoch_num=50) and device specification (cpu/cuda)
- Marker gene panel for validation (e.g., Gad1 for inhibitory neurons)

## Outputs

- adata.obs['conf'] and adata.obs['var'] per-cell confidence and variability scores
- Cell categories: easy-to-learn (correctly annotated), hard-to-learn (erroneously annotated), ambiguous
- Reannotated cell labels for erroneously annotated cells from the reannotation module
- Marker-gene alignment summary table (e.g., gad1_reannotation_results.csv) comparing mean marker expression before and after reannotation

## How to apply

Load the annotated AnnData (e.g., the MERFISH hypothalamus dataset with a 'label' obs key) from github.com/nitzanlab/Annotatability_notebooks and train the neural network with models.follow_training_dyn_neural_net(adata, label_key='label', iterNum=epoch_num, device=device) using defaults (lr=0.001, momentum=0.9, weighted_sampler=True, batch_size=256, num_layers=3, epoch_num=50). Convert the per-epoch probability list via models.probability_list_to_confidence_and_var into adata.obs['conf'] (average probability of the correct label across epochs) and adata.obs['var'] (its standard deviation). Classify cells into easy-to-learn (high confidence, low variability = correctly annotated), hard-to-learn (low confidence, low variability = erroneously annotated), and ambiguous (moderate confidence, high variability) categories from the score distributions. Train the reannotation module on the easy-to-learn cells and use it to relabel the hard-to-learn cells; optionally filter out erroneous cells instead. Validate by comparing marker expression before and after: for inhibitory neurons, mean normalized Gad1 was 6.830 in correctly annotated cells, 1.458 in erroneously annotated cells, and 6.795 after reannotation, confirming alignment with the new labels (Supplementary Table 1, Fig. 3j,k); write results such as gad1_reannotation_results.csv.

## Related tools

- **Annotatability** (Provides follow_training_dyn_neural_net to record per-epoch DNN predictions, probability_list_to_confidence_and_var to compute confidence/variability, and the reannotation module to relabel erroneously annotated cells) — https://github.com/nitzanlab/Annotatability
- **Scanpy** (Base framework for AnnData handling and visualization (e.g., sc.tl.umap, sc.pl.umap of conf scores) of the single-cell data) — https://scanpy.readthedocs.io/en/stable/
- **PyTorch** (Backend used to perform the neural-network training procedure whose dynamics are monitored)
- **Annotatability_notebooks** (Reproducibility notebooks supplying the annotated MERFISH hypothalamus dataset and worked reannotation examples) — https://github.com/nitzanlab/Annotatability_notebooks

## Examples

```
prob_list = models.follow_training_dyn_neural_net(adata, label_key='label', iterNum=50, device=device); all_conf, all_var = models.probability_list_to_confidence_and_var(prob_list, n_obs=adata.n_obs, epoch_num=50); adata.obs['conf'] = all_conf.detach().numpy(); adata.obs['var'] = all_var.detach().numpy()
```

## Evaluation signals

- Post-reannotation marker expression matches the correctly annotated class: mean normalized Gad1 of 6.795 after reannotation vs 6.830 in correctly annotated inhibitory neurons (and 1.458 in erroneously annotated cells before reannotation)
- Confidence/variability distributions separate into easy-to-learn (high conf, low var), hard-to-learn (low conf, low var), and ambiguous (moderate conf, high var) populations
- adata.obs['conf'] and adata.obs['var'] are populated for all adata.n_obs cells with finite values
- Reannotation summary artifacts (e.g., gad1_reannotation_results.csv) are written and agree with Supplementary Table 1 / Fig. 3j,k marker alignment

## Limitations

- Thresholding on confidence and variability scores introduces arbitrary cutoffs that could influence interpretation; the percentile q for hard-to-learn thresholds is chosen dataset-dependently (25%–90%)
- Hyperparameters such as training time need standardization as they may influence confidence and variability scores
- Annotatability does not directly handle convolved or aggregated data, such as that obtained from certain spatial transcriptomics technologies, and currently treats input annotations as nominal categories only

## Evidence

- [results] Classify cells into easy-to-learn (correctly annotated), hard-to-learn (erroneously annotated), and ambiguous categories based on the confidence/variability score distributions.: [evidence span withheld — anchor status: not_found]
- [results] Mean normalized expression of the inhibitory-neuron marker Gad1 was 6.830 in correctly annotated inhibitory neurons, 1.458 in erroneously annotated inhibitory neurons, and 6.795 after reannotation.: [evidence span withheld — anchor status: not_found]
