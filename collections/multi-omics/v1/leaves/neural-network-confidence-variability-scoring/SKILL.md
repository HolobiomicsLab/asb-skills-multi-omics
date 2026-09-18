---
name: neural-network-confidence-variability-scoring
description: Use when an annotated AnnData (e.g. PBMC scRNA-seq with an 'label' observation)
  must be audited for annotation mismatches, since training-dynamics confidence/variability
  scores expose likely erroneous annotations.
when_to_use_negative:
- Data is convolved or aggregated, e.g. from certain spatial transcriptomics technologies
  — Annotatability does not directly handle such data
- The input annotations are probabilistic or continuous rather than nominal categories
  — the method currently treats input annotations as nominal categories only
- Cells must be freshly clustered without existing labels — the scoring requires externally
  assigned annotations to monitor training against
tools:
- name: Annotatability
  role: Provides models.follow_training_dyn_neural_net for epoch-by-epoch training-dynamics
    recording and models.probability_list_to_confidence_and_var for computing per-cell
    confidence and variability scores
  repo: https://github.com/nitzanlab/Annotatability
- name: Scanpy
  role: AnnData container and single-cell data handling on which Annotatability's
    code is based; obs['conf']/obs['var'] are stored in the Scanpy AnnData object
  repo: https://scanpy.readthedocs.io/en/stable/
- name: PyTorch
  role: Backend used to perform the neural network training procedure and device selection
    (cuda:0/cpu)
- name: numpy
  role: Converts the returned confidence/variability tensors to arrays for storage
    in adata.obs via .detach().numpy()
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:67e33c287b7da999eab828457032e7e192f704c3cdd9288a5436171b0e2c1945
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# neural-network-confidence-variability-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute per-cell annotation-trainability scores — confidence (average probability of the correct label across training epochs) and variability (its standard deviation) — by monitoring a deep neural network's training dynamics on annotated single-cell data, then classify cells as easy-to-learn (correctly annotated), hard-to-learn (erroneously annotated), or ambiguous.

## When to use

Use when an annotated AnnData (e.g. PBMC scRNA-seq with an 'label' observation) must be audited for annotation mismatches, since training-dynamics confidence/variability scores expose likely erroneous annotations.

## When NOT to use

- Data is convolved or aggregated, e.g. from certain spatial transcriptomics technologies — Annotatability does not directly handle such data
- The input annotations are probabilistic or continuous rather than nominal categories — the method currently treats input annotations as nominal categories only
- Cells must be freshly clustered without existing labels — the scoring requires externally assigned annotations to monitor training against

## Inputs

- Annotated AnnData (h5ad) with cell-type annotations stored as an observation, e.g. adata.obs['label'] (e.g. PBMC scRNA-seq, MERFISH hypothalamus)
- Choice of label_key (e.g. 'label') and device (e.g. 'cuda:0' if available)

## Outputs

- adata.obs['conf']: per-cell confidence score (average probability of the correct label across epochs)
- adata.obs['var']: per-cell variability score (standard deviation of that probability across epochs)
- Per-cell classification into easy-to-learn, hard-to-learn, or ambiguous categories

## How to apply

Load the annotated AnnData and train a multilayer perceptron with models.follow_training_dyn_neural_net(adata, label_key='label', iterNum=epoch_num, device=device) using package defaults (iterNum=100, lr=0.001, momentum=0.9, batch_size=256, weighted_sampler=True, num_layers=3, cuda:0 if available), recording the predicted probability of the correct label after each epoch. Convert the per-epoch probability list into per-cell scores with models.probability_list_to_confidence_and_var and store them as adata.obs['conf'] and adata.obs['var']. Classify each cell into three categories: high confidence with low variability (easy-to-learn, likely correctly annotated), low confidence with low variability (hard-to-learn, likely erroneously annotated), or moderate confidence with high variability (ambiguous). The category thresholds are dataset-dependent percentiles (q between 25% and 90%). Confidence is defined as the average probability assigned by the model to the correct label across all epochs, and variability as the standard deviation of that probability.

## Related tools

- **Annotatability** (Provides models.follow_training_dyn_neural_net for epoch-by-epoch training-dynamics recording and models.probability_list_to_confidence_and_var for computing per-cell confidence and variability scores) — https://github.com/nitzanlab/Annotatability
- **Scanpy** (AnnData container and single-cell data handling on which Annotatability's code is based; obs['conf']/obs['var'] are stored in the Scanpy AnnData object) — https://scanpy.readthedocs.io/en/stable/
- **PyTorch** (Backend used to perform the neural network training procedure and device selection (cuda:0/cpu))
- **numpy** (Converts the returned confidence/variability tensors to arrays for storage in adata.obs via .detach().numpy())

## Examples

```
prob_list = models.follow_training_dyn_neural_net(adata, label_key='label', iterNum=100, device=device); all_conf, all_var = models.probability_list_to_confidence_and_var(prob_list, n_obs=adata.n_obs, epoch_num=100); adata.obs['conf'] = all_conf.detach().numpy(); adata.obs['var'] = all_var.detach().numpy()
```

## Evaluation signals

- prob_list returned by follow_training_dyn_neural_net has shape consistent with (epoch_num, n_obs) before conversion
- all_conf and all_var are returned by probability_list_to_confidence_and_var with n_obs=adata.n_obs entries each, and detach().numpy() succeeds for storage in adata.obs['conf'] and adata.obs['var']
- Class distribution sanity check: datasets with mostly correct annotations (e.g. PBMC scRNA-seq) show a high proportion of easy-to-learn cells, while condition-annotated mSTZ shows a larger score spread and fewer easy-to-learn cells
- Marker-gene validation: hard-to-learn CD14+ monocytes underexpress CD14 while ~80% express IL7R and ~60% express NKG7, confirming annotation mismatch (PBMC Fig. 2 result)
- Confidence values fall in [0,1] as averages of per-epoch correct-label probabilities; variability is the standard deviation of the same probability across epochs

## Limitations

- Thresholding on confidence and variability scores for categorizing introduces arbitrary, dataset-dependent cutoffs (percentile q chosen between 25% and 90%) that could influence data interpretation
- Determination of hyperparameters such as training time (iterNum) needs standardization, as they may influence the confidence and variability scores
- Annotatability currently treats input annotations as nominal categories only; generalization to probabilistic and continuous annotations is deferred to future work
- No software version is reported for PyTorch and no changelog exists for the repository, limiting reproducibility

## Evidence

- [discussion] confidence and variability definitions: [evidence span withheld — anchor status: not_found]
