---
name: multi-classifier-benchmarking
description: Use when a new cell-annotation-error score must be quantitatively compared
  to established per-cell mislabel-detection baselines on semi-synthetically perturbed
  annotated single-cell or spatial data, using AUROC vs mislabeled fraction.
when_to_use_negative:
- The dataset has no externally assigned annotations to audit or no way to define
  a correct/incorrect ground-truth flag, since AUROC of mislabel detection cannot
  be computed
- The data is convolved or aggregated spatial transcriptomics data, which Annotatability
  does not directly handle
- Only a single perturbation level (or a single classifier) is evaluated, making an
  AUROC-versus-fraction benchmark comparison meaningless
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3360
tools:
- name: Annotatability
  role: 'Method under test: trains a DNN via follow_training_dyn_neural_net and derives
    per-cell confidence/variability scores used for mislabel-detection AUROC'
  repo: https://github.com/nitzanlab/Annotatability
- name: scReClassify
  role: Provides the SVM, KNN and RF baseline classifiers for per-cell mislabel prediction
    on the same perturbed datasets
- name: PyTorch
  role: Backend used to perform the DNN training procedure of Annotatability
- name: Scanpy
  role: Data structure and processing basis for the annotated AnnData input
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:a6587306eab9465faf16c9c2863ca385e495a51c53f1fa772d55aa8a8b29dda5
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
  - Scanpy
  - numpy
  - pandas
  - scReClassify
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# multi-classifier-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks a novel mislabel-detection signal (Annotatability's confidence/variability training-dynamics scores) against multi-classifier baselines (scReClassify with SVM, KNN, RF) by computing AUROC of mislabel detection across controlled perturbation levels.

## When to use

Use when a new cell-annotation-error score must be quantitatively compared to established per-cell mislabel-detection baselines on semi-synthetically perturbed annotated single-cell or spatial data, using AUROC vs mislabeled fraction.

## When NOT to use

- The dataset has no externally assigned annotations to audit or no way to define a correct/incorrect ground-truth flag, since AUROC of mislabel detection cannot be computed
- The data is convolved or aggregated spatial transcriptomics data, which Annotatability does not directly handle
- Only a single perturbation level (or a single classifier) is evaluated, making an AUROC-versus-fraction benchmark comparison meaningless

## Inputs

- Annotated AnnData object (e.g. MERFISH mouse hypothalamic preoptic region) with cell-type labels in an adata.obs column
- Semi-synthetic perturbed label copies with a per-cell correct/incorrect ground-truth flag
- List of baseline classifiers (scReClassify with SVM, KNN, RF)

## Outputs

- Per-cell confidence and variability scores stored in adata.obs['conf'] and adata.obs['var']
- Per-cell mislabel predictions from each baseline classifier
- AUROC values of mislabel detection per method per mislabeling fraction
- AUROC-versus-mislabeled-fraction figure (auroc_vs_mislabeled_fraction.png)

## How to apply

Load the annotated dataset (e.g. MERFISH mouse hypothalamic preoptic region AnnData with cell-type labels in an obs column) and create semi-synthetic perturbed copies by randomly reassigning the 'label' of controlled fractions of cells (5%–50%) while keeping a per-cell correct/incorrect ground-truth flag. For each perturbation level, run the method under test — here models.follow_training_dyn_neural_net (iterNum=50, lr=0.001, momentum=0.9, batch_size=256, weighted_sampler=True, num_layers=3, GPU if available) followed by models.probability_list_to_confidence_and_var to obtain per-cell confidence and variability stored in adata.obs['conf'] and adata.obs['var']. Score the identical perturbed datasets with scReClassify using SVM, KNN and RF base classifiers to obtain per-cell mislabel predictions. Compute AUROC of mislabel detection against the correct/incorrect flag for every method at every mislabeling percentage, then plot AUROC versus percentage of mislabeled cells so the relative advantage (which grows with more mislabeled cells) is visible.

## Related tools

- **Annotatability** (Method under test: trains a DNN via follow_training_dyn_neural_net and derives per-cell confidence/variability scores used for mislabel-detection AUROC) — https://github.com/nitzanlab/Annotatability
- **scReClassify** (Provides the SVM, KNN and RF baseline classifiers for per-cell mislabel prediction on the same perturbed datasets)
- **PyTorch** (Backend used to perform the DNN training procedure of Annotatability)
- **Scanpy** (Data structure and processing basis for the annotated AnnData input)

## Examples

```
prob_list = models.follow_training_dyn_neural_net(adata, label_key='label', iterNum=50, lr=0.001, momentum=0.9, batch_size=256, weighted_sampler=True, num_layers=3, device=device); all_conf, all_var = models.probability_list_to_confidence_and_var(prob_list, n_obs=adata.n_obs, epoch_num=50)
```

## Evaluation signals

- Validation figure auroc_vs_mislabeled_fraction.png exists and contains one AUROC curve per method (Annotatability, SVM, KNN, RF) across all perturbation levels
- Annotatability AUROC values are computed from confidence/variability scores stored in adata.obs['conf'] and adata.obs['var']
- Annotatability consistently outperforms the scReClassify-based baselines, with the advantage growing as more cells are synthetically misclassified (5%–50%)
- Each training run uses the documented hyperparameters: iterNum=50, lr=0.001, momentum=0.9, batch_size=256, weighted_sampler=True, num_layers=3
- The same perturbed datasets and correct/incorrect flags are used as ground truth for both the method under test and all baselines

## Limitations

- Determination of hyperparameters such as training time needs standardization, as they may influence confidence and variability scores
- Thresholding on confidence and variability scores introduces arbitrary cutoffs that could influence data interpretation
- Annotatability does not directly handle convolved or aggregated data, such as that obtained from certain spatial transcriptomics technologies
- The hard-to-learn percentile q is chosen in a dataset-dependent manner (25%–90%), a manual, non-standardized choice

## Evidence

- [results] Annotatability consistently outperformed the scReClassify-based baselines (SVM, KNN, RF) in distinguishing correctly from erroneously annotated cells: [evidence span withheld — anchor status: not_found]
- [discussion] Confidence definition: [evidence span withheld — anchor status: not_found]
