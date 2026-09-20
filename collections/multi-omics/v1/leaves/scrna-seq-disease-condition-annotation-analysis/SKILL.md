---
name: scrna-seq-disease-condition-annotation-analysis
description: Use when a scRNA-seq AnnData object is annotated by coarse experimental
  condition (disease vs healthy) and you want to test whether DNN training-dynamics
  confidence tracks graded disease severity via marker module expression.
when_to_use_negative:
- The annotation of interest is a continuous or probabilistic variable, since Annotatability
  treats input annotations as nominal categories only
- Data are convolved or aggregated (e.g. certain spatial transcriptomics technologies),
  which Annotatability does not directly handle
- The goal is simply to reannotate or filter mislabeled cells by cell type rather
  than to test whether confidence reflects a graded disease phenotype
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Annotatability
  role: Trains the DNN via follow_training_dyn_neural_net and converts the probability
    list to per-cell confidence and variability scores
  repo: https://github.com/nitzanlab/Annotatability
- name: PyTorch
  role: Backend for the DNN training procedure and device selection (cuda:0 if available)
- name: Scanpy
  role: Base framework for handling the AnnData object and marker module score computation
  repo: https://scanpy.readthedocs.io/en/stable/
- name: scipy
  role: Computes Spearman rank correlations between per-cell confidence and marker
    module scores
- name: pandas
  role: Writes correlation coefficients to mstz_confidence_correlations.csv
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:ad92033f621ca7d8205e15a2662b549dcdf071244f46221a7c930168fdfc8d72
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# scrna-seq-disease-condition-annotation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains the Annotatability DNN on scRNA-seq data annotated by experimental condition (e.g. healthy vs mSTZ) and correlates per-cell confidence scores with disease-relevant marker module scores to test whether annotation confidence reflects graded cellular damage, as in the mSTZ diabetic islet analysis (Spearman ρ = −0.662 with β cell activity markers).

## When to use

Use when a scRNA-seq AnnData object is annotated by coarse experimental condition (disease vs healthy) and you want to test whether DNN training-dynamics confidence tracks graded disease severity via marker module expression.

## When NOT to use

- The annotation of interest is a continuous or probabilistic variable, since Annotatability treats input annotations as nominal categories only
- Data are convolved or aggregated (e.g. certain spatial transcriptomics technologies), which Annotatability does not directly handle
- The goal is simply to reannotate or filter mislabeled cells by cell type rather than to test whether confidence reflects a graded disease phenotype

## Inputs

- AnnData object of scRNA-seq cells annotated by experimental condition in obs['label'] (e.g. healthy vs mSTZ pancreatic islet cells)
- Predefined marker gene lists for disease-relevant modules (β-cell activity: Ins1, Ins2, Slc2a2, Trpm5, G6pc2, Slc30a8, Ucn3; dedifferentiation markers)

## Outputs

- Per-cell confidence scores in adata.obs['conf'] and variability in adata.obs['var']
- Spearman rank correlation coefficients between confidence and marker module scores (e.g. mstz_confidence_correlations.csv)

## How to apply

Load the condition-annotated dataset (e.g. mSTZ pancreatic islets, healthy vs mSTZ) as AnnData with the annotation stored in obs under 'label'. Train the Annotatability DNN with models.follow_training_dyn_neural_net using label_key='label', iterNum of 50–100 epochs, a 3-layer network (num_layers=3), batch_size=256, and weighted_sampler=True to handle class imbalance, on GPU if available. Convert the per-epoch probability list via models.probability_list_to_confidence_and_var and store adata.obs['conf'] and adata.obs['var']; confidence is the average probability assigned to the correct label across epochs, variability its standard deviation. Compute summed module scores of β-cell activity markers (Ins1, Ins2, Slc2a2, Trpm5, G6pc2, Slc30a8, Ucn3) and dedifferentiation markers per cell, then compute Spearman rank correlations between per-cell confidence and each module score with scipy.stats. Write coefficients to a CSV (e.g. mstz_confidence_correlations.csv) and compare against reported values (ρ = −0.662 activity, ρ = 0.573 dedifferentiation) and a diffusion-pseudotime baseline.

## Related tools

- **Annotatability** (Trains the DNN via follow_training_dyn_neural_net and converts the probability list to per-cell confidence and variability scores) — https://github.com/nitzanlab/Annotatability
- **PyTorch** (Backend for the DNN training procedure and device selection (cuda:0 if available))
- **Scanpy** (Base framework for handling the AnnData object and marker module score computation) — https://scanpy.readthedocs.io/en/stable/
- **scipy** (Computes Spearman rank correlations between per-cell confidence and marker module scores)
- **pandas** (Writes correlation coefficients to mstz_confidence_correlations.csv)

## Examples

```
prob_list = models.follow_training_dyn_neural_net(adata, label_key='label', iterNum=50, device=device); all_conf, all_var = models.probability_list_to_confidence_and_var(prob_list, n_obs=adata.n_obs, epoch_num=50)
```

## Evaluation signals

- Reproduced Spearman ρ for summed β-cell activity markers matches −0.662 (Fig. 5a) within sampling tolerance
- Reproduced Spearman ρ for dedifferentiation markers matches 0.573 (Fig. 5a) within sampling tolerance
- Correlations are compared against diffusion-pseudotime baseline coefficients as a sanity check
- all_conf and all_var returned by probability_list_to_confidence_and_var have length equal to adata.n_obs and are stored in adata.obs['conf'] and adata.obs['var']
- Training uses weighted_sampler=True and the negative correlation direction (confidence decreases with β-cell activity) is reproduced

## Limitations

- Hyperparameters such as training time (iterNum) influence confidence and variability scores and are not standardized
- Thresholding confidence and variability into easy/hard/ambiguous categories introduces arbitrary dataset-dependent cutoffs (q between 25% and 90%)
- Annotatability treats input annotations as nominal categories only; continuous annotations are deferred to future work
- No PyTorch version is reported, limiting reproducibility of the training procedure

## Evidence

- [results] Spearman ρ = −0.662 for summed β-cell activity markers; ρ = 0.573 for dedifferentiation markers: [evidence span withheld — anchor status: not_found]
- [discussion] we define confidence, $$\widehat{{u}_{i}}$$, as the average probability assigned by the model to the correct label: [evidence span withheld — anchor status: not_found]
