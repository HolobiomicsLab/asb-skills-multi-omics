---
name: marker-gene-alignment-validation
description: Use when cells have been reannotated by Annotatability's reannotation
  module and you must confirm the new labels biologically by checking whether canonical
  marker expression (e.g., Gad1 for inhibitory neurons) now matches the reassigned
  cell type.
when_to_use_negative:
- Input is aggregated or convolved spatial transcriptomics data, which Annotatability
  does not directly handle
- No canonical marker gene is known for the cell type being reannotated, so alignment
  cannot be judged against expected expression
- The task is only to filter out erroneous annotations without relabeling; use the
  optional filtering step instead of the reannotation module
tools:
- name: Annotatability
  role: Provides follow_training_dyn_neural_net, probability_list_to_confidence_and_var,
    and the reannotation module used to relabel erroneously annotated cells before
    marker validation
  repo: https://github.com/nitzanlab/Annotatability
- name: Scanpy
  role: Base framework for handling the AnnData expression matrix and computing per-cell
    marker expression statistics
  repo: https://scanpy.readthedocs.io/en/stable/
- name: PyTorch
  role: Backend for DNN training and the reannotation module, executed on a configurable
    device (cpu/cuda)
- name: numpy
  role: Numerical handling of confidence/variability tensors (e.g., all_conf.detach().numpy())
    and mean marker-expression aggregation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:f015761e75c72ce1fedf530ce18d6169ca9fe12e7b0778bef3c4ea65d5cbc6b3
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# marker-gene-alignment-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates that reannotated cells from Annotatability's reannotation module have recovered marker-gene expression consistent with their new cell type labels, by comparing mean normalized marker expression (e.g., the inhibitory-neuron marker Gad1 in the MERFISH hypothalamus dataset) before and after reannotation.

## When to use

Use when cells have been reannotated by Annotatability's reannotation module and you must confirm the new labels biologically by checking whether canonical marker expression (e.g., Gad1 for inhibitory neurons) now matches the reassigned cell type.

## When NOT to use

- Input is aggregated or convolved spatial transcriptomics data, which Annotatability does not directly handle
- No canonical marker gene is known for the cell type being reannotated, so alignment cannot be judged against expected expression
- The task is only to filter out erroneous annotations without relabeling; use the optional filtering step instead of the reannotation module

## Inputs

- Annotated AnnData object (e.g., MERFISH hypothalamus dataset) with gene expression matrix and a 'label' observation column
- Per-cell confidence and variability scores stored in adata.obs['conf'] and adata.obs['var']
- Cell-type/category assignment of easy-to-learn (correct) and hard-to-learn (erroneous) cells
- Canonical marker gene(s) for the target cell type (e.g., Gad1 for inhibitory neurons)

## Outputs

- Reannotated AnnData with hard-to-learn cells relabeled by the reannotation module
- Mean normalized marker expression per annotation class (correct, erroneous-pre-reannotation, reannotated), e.g., Gad1 values 6.830 / 1.458 / 6.795
- Marker-alignment summary CSV (e.g., gad1_reannotation_results.csv)

## How to apply

Load the annotated AnnData (e.g., the MERFISH hypothalamus dataset with a 'label' obs column from github.com/nitzanlab/Annotatability_notebooks), train the DNN with models.follow_training_dyn_neural_net (default lr=0.001, momentum=0.9, weighted_sampler=True, batch_size=256, num_layers=3), and convert the per-epoch probability list to per-cell confidence and variability via models.probability_list_to_confidence_and_var. Classify cells into easy-to-learn (correctly annotated), hard-to-learn (erroneously annotated), and ambiguous categories from the conf/var score distributions, then train the reannotation module on easy-to-learn cells and relabel hard-to-learn cells. Compute mean normalized expression of a canonical marker (e.g., Gad1) for correctly annotated versus reannotated cells of the target type and compare against expected values — for Gad1, ~6.830 (correct) vs ~6.795 (after reannotation) vs ~1.458 (before reannotation) — to judge whether marker expression aligns with the new annotations, as in Supplementary Table 1 / Fig. 3j,k. Persist the marker comparison and reannotation summary (e.g., gad1_reannotation_results.csv) for reporting.

## Related tools

- **Annotatability** (Provides follow_training_dyn_neural_net, probability_list_to_confidence_and_var, and the reannotation module used to relabel erroneously annotated cells before marker validation) — https://github.com/nitzanlab/Annotatability
- **Scanpy** (Base framework for handling the AnnData expression matrix and computing per-cell marker expression statistics) — https://scanpy.readthedocs.io/en/stable/
- **PyTorch** (Backend for DNN training and the reannotation module, executed on a configurable device (cpu/cuda))
- **numpy** (Numerical handling of confidence/variability tensors (e.g., all_conf.detach().numpy()) and mean marker-expression aggregation)

## Examples

```
prob_list = models.follow_training_dyn_neural_net(adata, label_key='label', iterNum=epoch_num, device=device); all_conf, all_var = models.probability_list_to_confidence_and_var(prob_list, n_obs=adata.n_obs, epoch_num=epoch_num)
```

## Evaluation signals

- Mean normalized Gad1 expression in reannotated inhibitory neurons (~6.795) matches correctly annotated inhibitory neurons (~6.830), not the pre-reannotation erroneous value (~1.458)
- Marker-expression alignment agrees with Supplementary Table 1 and the patterns shown in Fig. 3j,k
- Confidence/variability scores are present in adata.obs['conf'] and adata.obs['var'] after probability_list_to_confidence_and_var
- Marker alignment holds for additional cell-type marker pairs beyond Gad1, consistent with the reported Supplementary Table 1 alignment
- Reannotation summary CSV (gad1_reannotation_results.csv) contains per-class mean expression values within expected ranges

## Limitations

- Thresholding on confidence and variability scores introduces arbitrary cutoffs that could influence which cells are reannotated and thus which are validated
- Hyperparameters such as training time (epoch_num, default 100 in the package, 50 in the README example) are not standardized and may influence confidence and variability scores
- Percentile q for hard-to-learn thresholds is chosen dataset-dependently (25%–90%), a manual, non-standardized choice

## Evidence

- [results] Gad1 marker alignment finding: [evidence span withheld — anchor status: not_found]
