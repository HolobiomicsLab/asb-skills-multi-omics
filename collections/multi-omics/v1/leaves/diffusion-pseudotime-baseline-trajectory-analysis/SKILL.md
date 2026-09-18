---
name: diffusion-pseudotime-baseline-trajectory-analysis
description: Use when single-cell data spans a gradual biological transition (e.g.
  EMT) and you need a continuous pseudotime baseline to compare against DNN training-dynamics
  confidence scores or marker expression trends.
when_to_use_negative:
- Data are convolved or aggregated, such as from certain spatial transcriptomics technologies
  — Annotatability does not directly handle such data
- The input annotation is continuous or probabilistic rather than nominal categories,
  which Annotatability and this confidence-vs-pseudotime comparison do not currently
  support
- The dataset contains no gradual transition trajectory, so pseudotime ordering and
  monotone marker trends are not meaningful baselines
tools:
- name: Annotatability
  role: Trains the neural classifier and derives per-cell confidence and variability
    scores whose correlation with marker expression is benchmarked against the pseudotime
    baseline
  repo: https://github.com/nitzanlab/Annotatability
- name: Scanpy
  role: Provides diffusion-pseudotime computation used as the classical trajectory
    baseline for correlation comparison
  repo: https://scanpy.readthedocs.io/en/stable/
- name: PyTorch
  role: Backend used to perform the DNN training procedure that produces the per-epoch
    prediction probabilities
- name: scipy
  role: Computes Spearman rank correlations between marker-set means and confidence/pseudotime
    scores
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:90009b0240626bf2fa5f3b705ef381c839b3d3a2f516f7b455d14176ebeb560a
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
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# Diffusion-pseudotime baseline trajectory analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes diffusion-pseudotime (DPT) trajectories with Scanpy and correlates them with marker-gene expression or Annotatability confidence scores, providing a classical trajectory baseline for judging whether confidence scores track gradual biological transitions such as the EMT.

## When to use

Use when single-cell data spans a gradual biological transition (e.g. EMT) and you need a continuous pseudotime baseline to compare against DNN training-dynamics confidence scores or marker expression trends.

## When NOT to use

- Data are convolved or aggregated, such as from certain spatial transcriptomics technologies — Annotatability does not directly handle such data
- The input annotation is continuous or probabilistic rather than nominal categories, which Annotatability and this confidence-vs-pseudotime comparison do not currently support
- The dataset contains no gradual transition trajectory, so pseudotime ordering and monotone marker trends are not meaningful baselines

## Inputs

- EMT scRNA-seq AnnData object (MCF10A and HuMEC, mock vs TGF-β conditions) with low-quality cells filtered and epithelial/mesenchymal annotation in adata.obs['label']
- Per-epoch prediction probability list from the Annotatability DNN training (prob_list)
- Lists of epithelial and mesenchymal marker gene sets

## Outputs

- adata.obs['conf'] and adata.obs['var'] per-cell training-dynamics scores
- Per-cell mean epithelial and mesenchymal marker expression on L2-normalized values
- Spearman rank correlation table comparing confidence-score and diffusion-pseudotime baselines (e.g. emt_confidence_spearman.csv)

## How to apply

Load the EMT scRNA-seq AnnData (MCF10A and HuMEC, mock vs TGF-β, low-quality cells already filtered: ≥5% mitochondrial counts, >4,000 expressed genes, or <200 genes removed) with the Louvain-derived epithelial/mesenchymal annotation in the 'label' observation. Optionally derive confidence scores first by running models.follow_training_dyn_neural_net(adata, label_key='label', iterNum=50, device=device) with documented defaults (lr=0.001, momentum=0.9, weighted_sampler=True, batch_size=256, num_layers=3), then models.probability_list_to_confidence_and_var to fill adata.obs['conf'] and adata.obs['var']. Compute per-cell mean expression of epithelial and mesenchymal marker sets on L2-normalized (variance-scaled to 1) expression. Run Scanpy diffusion pseudotime as the baseline and compute Spearman rank correlations of each marker-set mean with both pseudotime and the confidence score; the article reports epithelial-marker correlations of 0.410 (confidence) vs −0.421 and mesenchymal values 0.606/−0.210 for comparison. Write per-cell scores and the correlation table (e.g. emt_confidence_spearman.csv) and judge success by monotonic trends and proximity to the reported rank correlations shown in Fig. 4e,f.

## Related tools

- **Annotatability** (Trains the neural classifier and derives per-cell confidence and variability scores whose correlation with marker expression is benchmarked against the pseudotime baseline) — https://github.com/nitzanlab/Annotatability
- **Scanpy** (Provides diffusion-pseudotime computation used as the classical trajectory baseline for correlation comparison) — https://scanpy.readthedocs.io/en/stable/
- **PyTorch** (Backend used to perform the DNN training procedure that produces the per-epoch prediction probabilities)
- **scipy** (Computes Spearman rank correlations between marker-set means and confidence/pseudotime scores)

## Examples

```
from Annotatability import models; import scanpy as sc; prob_list = models.follow_training_dyn_neural_net(adata, label_key='label', iterNum=50, device=device); conf, var = models.probability_list_to_confidence_and_var(prob_list, n_obs=adata.n_obs, epoch_num=50); adata.obs['conf'] = conf.detach().numpy(); sc.tl.dpt(adata)
```

## Evaluation signals

- Mean epithelial marker expression increases monotonically and mean mesenchymal marker expression decreases monotonically with the confidence score for epithelial-labeled cells
- Reproduced Spearman correlations approach the reported values: epithelial markers 0.410 (epithelial) / −0.421, mesenchymal 0.606 / −0.210, as in Fig. 4e,f
- adata.obs contains finite 'conf' and 'var' columns with lengths equal to adata.n_obs after probability_list_to_confidence_and_var
- Correlation output CSV (emt_confidence_spearman.csv) contains per-cell scores plus a correlation table covering both confidence and DPT baselines

## Limitations

- Hyperparameters such as training time (epoch count) are not standardized and may influence the confidence and variability scores used in the correlations
- Thresholding on confidence and variability scores introduces arbitrary cutoffs (hard-to-learn percentile q chosen dataset-dependently between 25% and 90%) that could influence interpretation
- No PyTorch version is reported, limiting reproducibility of the training procedure underlying the confidence scores

## Evidence

- [results] For cells originally labeled as epithelial, mean epithelial marker gene expression increases monotonically and mean mesenchymal marker expression decreases monotonically with Annotatability's confidence score (Spearman's rank correlations: 0.410 and −0.421, respectively): [evidence span withheld — anchor status: not_found]
