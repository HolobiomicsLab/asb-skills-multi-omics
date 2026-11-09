---
name: neural-network-probability-trajectory-monitoring
description: Use when an annotated single-cell AnnData object (e.g. mSTZ islet cells
  labeled by condition) must be audited for annotation mismatches, or when per-cell
  trainability scores are needed to relate labels to continuous biology like β cell
  activity.
when_to_use_negative:
- Data are convolved or aggregated, such as from certain spatial transcriptomics technologies
  — Annotatability does not directly handle such inputs
- The input annotations are continuous or probabilistic quantities — Annotatability
  currently treats input annotations as nominal categories only
- The goal is downstream differential expression on already-validated labels rather
  than auditing annotation–biology agreement
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: Annotatability
  role: Provides models.follow_training_dyn_neural_net to record per-epoch correct-label
    probabilities and models.probability_list_to_confidence_and_var to compute confidence
    and variability
  repo: https://github.com/nitzanlab/Annotatability
- name: PyTorch
  role: Backend for the DNN training procedure (device, optimizer, weighted sampler)
- name: Scanpy
  role: AnnData handling and single-cell preprocessing on which the code is based
  repo: https://scanpy.readthedocs.io/en/stable/
- name: scipy
  role: Spearman rank correlation between per-cell confidence and marker module scores
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:be9e348f92f2a73973d2d9ff30e0d51b10129ad89ed7c54d549b99d9cd9c2f04
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# neural-network-probability-trajectory-monitoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Record the probability a DNN assigns to each cell's correct label after every training epoch, then summarize these trajectories into per-cell confidence (mean probability) and variability (standard deviation) scores that quantify each cell's 'goodness-of-match' to its annotation.

## When to use

Use when an annotated single-cell AnnData object (e.g. mSTZ islet cells labeled by condition) must be audited for annotation mismatches, or when per-cell trainability scores are needed to relate labels to continuous biology like β cell activity.

## When NOT to use

- Data are convolved or aggregated, such as from certain spatial transcriptomics technologies — Annotatability does not directly handle such inputs
- The input annotations are continuous or probabilistic quantities — Annotatability currently treats input annotations as nominal categories only
- The goal is downstream differential expression on already-validated labels rather than auditing annotation–biology agreement

## Inputs

- Annotated AnnData object (scRNA-seq, e.g. mSTZ pancreatic islet dataset) with cell labels in adata.obs['label']
- Annotation key of interest (e.g. 'label' for condition: healthy vs mSTZ)
- Hyperparameters: iterNum (50–100 epochs), batch_size=256, num_layers=3, weighted_sampler=True, device ('cuda:0' or 'cpu')

## Outputs

- prob_list: per-epoch list of correct-label probabilities for all cells
- adata.obs['conf']: per-cell confidence score (mean probability across epochs)
- adata.obs['var']: per-cell variability score (SD of probability across epochs)
- Spearman correlation coefficients between conf and marker module scores (e.g. mstz_confidence_correlations.csv)

## How to apply

Train a 3-layer PyTorch DNN on the AnnData expression matrix with models.follow_training_dyn_neural_net using label_key='label', iterNum=50–100 epochs, batch_size=256, lr=0.001, momentum=0.9, weighted_sampler=True for class imbalance, and device set to 'cuda:0' when a GPU is available; the function returns prob_list, the per-epoch probability of the correct label for every cell. Convert prob_list with models.probability_list_to_confidence_and_var (passing n_obs=adata.n_obs and epoch_num) to obtain all_conf and all_var, storing them as adata.obs['conf'] (average probability across epochs) and adata.obs['var'] (standard deviation across epochs). Interpret high confidence + low variability as correctly annotated ('easy to learn'), low confidence + low variability as probably mislabeled ('hard to learn'), and moderate confidence + high variability as ambiguous states. In the mSTZ case, correlate the resulting conf scores against summed β cell activity marker expression (Ins1, Ins2, Slc2a2, Trpm5, G6pc2, Slc30a8, Ucn3) with scipy.stats Spearman rank correlation, expecting ρ ≈ −0.662.

## Related tools

- **Annotatability** (Provides models.follow_training_dyn_neural_net to record per-epoch correct-label probabilities and models.probability_list_to_confidence_and_var to compute confidence and variability) — https://github.com/nitzanlab/Annotatability
- **PyTorch** (Backend for the DNN training procedure (device, optimizer, weighted sampler))
- **Scanpy** (AnnData handling and single-cell preprocessing on which the code is based) — https://scanpy.readthedocs.io/en/stable/
- **scipy** (Spearman rank correlation between per-cell confidence and marker module scores)

## Examples

```
prob_list = models.follow_training_dyn_neural_net(adata, label_key='label', iterNum=50, device=device); all_conf, all_var = models.probability_list_to_confidence_and_var(prob_list, n_obs=adata.n_obs, epoch_num=50); adata.obs['conf'] = all_conf.detach().numpy(); adata.obs['var'] = all_var.detach().numpy()
```

## Evaluation signals

- Reproduced Spearman ρ between mSTZ confidence and summed β cell activity markers (Ins1, Ins2, Slc2a2, Trpm5, G6pc2, Slc30a8, Ucn3) matches the reported Fig. 5a value of −0.662 within sampling tolerance
- Reproduced ρ for dedifferentiation markers matches the reported 0.573
- adata.obs['conf'] and adata.obs['var'] are populated for all n_obs cells with values in [0,1] and non-negative variability respectively
- Probability distributions differ by annotation class: mSTZ condition annotation shows larger spread and fewer easy-to-learn cells than cell-type annotation
- Easy-to-learn cells have high conf and low var; hard-to-learn cells have low conf and low var; ambiguous cells have moderate conf and high var

## Limitations

- Hyperparameters such as training time (iterNum) are not standardized and may influence the determination of confidence and variability scores
- Thresholding on confidence and variability for categorizing cells introduces arbitrary cutoffs that could influence data interpretation; the percentile q for hard-to-learn thresholds is chosen dataset-dependently between 25% and 90%
- No PyTorch version is reported, and no changelog exists for the referenced repository, limiting reproducibility
- Only confidence and variability are incorporated; additional training-dynamics measures are deferred to future work

## Evidence

- [discussion] we define confidence, $\widehat{{u}_{i}}$, as the average probability assigned by the model to the correct label: [evidence span withheld — anchor status: not_found]
