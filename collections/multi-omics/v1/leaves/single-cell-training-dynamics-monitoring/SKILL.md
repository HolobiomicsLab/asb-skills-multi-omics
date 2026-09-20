---
name: single-cell-training-dynamics-monitoring
description: Use when annotated scRNA-seq/spatial omics data (AnnData with a 'label'
  obs column) needs auditing for erroneous annotations, ambiguous intermediate states,
  or gene-trainability ranking via DNN training dynamics.
when_to_use_negative:
- Input is convolved or aggregated spatial transcriptomics data — Annotatability does
  not directly handle such data
- Annotations are probabilistic or continuous — the method currently treats input
  annotations as nominal categories only
- The goal is standard cell clustering or neighbors analysis where sc.pp.neighbors
  would overwrite/replace the trainability-aware graph in adata.obsp['connectivities']
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: Annotatability
  role: 'Core package: models.follow_training_dyn_neural_net trains the DNN and records
    per-epoch probabilities; models.probability_list_to_confidence_and_var computes
    confidence/variability; metrics.rank_genes_conf_min_counts ranks genes by trainability
    association'
  repo: https://github.com/nitzanlab/Annotatability
- name: PyTorch
  role: Backend for the neural-network training procedure; device selection via torch.device('cuda:0'
    if torch.cuda.is_available() else 'cpu')
- name: Scanpy
  role: AnnData handling and visualization of the trainability-aware graph (sc.tl.umap,
    sc.pl.umap color='conf')
  repo: https://scanpy.readthedocs.io/en/stable/
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:1a4791f229cf01467dbf3439b34cc9520f886ba597be2aa37577322049d7a515
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
yes_rate: 0.8125
n_claims: 16
---

# single-cell-training-dynamics-monitoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotatability-style annotation-trainability analysis: train a deep neural network on annotated single-cell (AnnData) or spatial omics data, recording the model's predicted probability of each cell's correct label after every epoch, then derive per-cell confidence (mean probability across epochs) and variability (standard deviation across epochs) to classify cells as easy-to-learn, hard-to-learn, or ambiguous.

## When to use

Use when annotated scRNA-seq/spatial omics data (AnnData with a 'label' obs column) needs auditing for erroneous annotations, ambiguous intermediate states, or gene-trainability ranking via DNN training dynamics.

## When NOT to use

- Input is convolved or aggregated spatial transcriptomics data — Annotatability does not directly handle such data
- Annotations are probabilistic or continuous — the method currently treats input annotations as nominal categories only
- The goal is standard cell clustering or neighbors analysis where sc.pp.neighbors would overwrite/replace the trainability-aware graph in adata.obsp['connectivities']

## Inputs

- Annotated AnnData (h5ad) with cell-type or condition labels in a named .obs column (e.g., 'label'), e.g., PBMC scRNA-seq or EMT (MCF10A/HuMEC) datasets
- Optional gene marker sets (e.g., CD14, IL7R, NKG7 for PBMC; epithelial/mesenchymal marker sets for EMT) for verification
- Device specification (cuda:0 or cpu) and epoch number (e.g., iterNum=50 or 100)

## Outputs

- adata.obs['conf'] — per-cell confidence (average probability of the correct label across epochs)
- adata.obs['var'] — per-cell variability (standard deviation of that probability across epochs)
- Per-cell classification into easy-to-learn, hard-to-learn, or ambiguous categories
- adata_ranked.var['conf_score_high'] and adata_ranked.var['conf_score_low'] — gene annotation-trainability association scores
- Saved results file (e.g., pbmc_annotatability_results.h5ad or emt_confidence_spearman.csv)

## How to apply

Load the annotated AnnData and optionally exclude rare annotations (e.g., in the PBMC dataset, exclude megakaryocytes, 88/11,990 cells). Train the MLP classifier with models.follow_training_dyn_neural_net(adata, label_key='label', iterNum=epoch_num, device=device) using package defaults (iterNum=100 epochs, lr=0.001, momentum=0.9, batch_size=256, weighted_sampler=True, num_layers=3) on cuda:0 if available; scale each gene's expression to variance 1 (L2 normalization) before scoring trainability associations. Convert the returned probability list with models.probability_list_to_confidence_and_var(prob_list, n_obs=adata.n_obs, epoch_num=epoch_num) and store adata.obs['conf'] and adata.obs['var']. Interpret high confidence + low variability as correctly annotated (easy-to-learn), low confidence + low variability as likely erroneous (hard-to-learn), and moderate confidence + high variability as ambiguous. Optionally rank genes by annotation-trainability with metrics.rank_genes_conf_min_counts, yielding adata_ranked.var['conf_score_high'] (positive association) and adata_ranked.var['conf_score_low'] (negative association), then verify marker-gene behavior (e.g., erroneously annotated CD14+ monocytes underexpress CD14 but express IL7R and NKG7).

## Related tools

- **Annotatability** (Core package: models.follow_training_dyn_neural_net trains the DNN and records per-epoch probabilities; models.probability_list_to_confidence_and_var computes confidence/variability; metrics.rank_genes_conf_min_counts ranks genes by trainability association) — https://github.com/nitzanlab/Annotatability
- **PyTorch** (Backend for the neural-network training procedure; device selection via torch.device('cuda:0' if torch.cuda.is_available() else 'cpu'))
- **Scanpy** (AnnData handling and visualization of the trainability-aware graph (sc.tl.umap, sc.pl.umap color='conf')) — https://scanpy.readthedocs.io/en/stable/

## Examples

```
prob_list = models.follow_training_dyn_neural_net(adata, label_key='label', iterNum=epoch_num, device=device); all_conf, all_var = models.probability_list_to_confidence_and_var(prob_list, n_obs=adata.n_obs, epoch_num=epoch_num); adata.obs['conf'] = all_conf.detach().numpy(); adata.obs['var'] = all_var.detach().numpy()
```

## Evaluation signals

- adata.obs['conf'] and adata.obs['var'] are populated for all n_obs cells with values in [0,1] (probabilities of the correct label)
- PBMC verification: cells classified as erroneously annotated CD14+ monocytes underexpress CD14, while ~80% express IL7R and ~60% express NKG7 (annotation mismatches confirmed)
- EMT verification: for epithelial-labeled cells, Spearman correlation of mean epithelial marker expression with confidence ≈ 0.410 and mesenchymal ≈ −0.421 (Fig. 4e,f)
- Category proportions: easy-to-learn cells have high conf/low var; hard-to-learn cells have low conf/low var; ambiguous cells have moderate conf/high var
- rank_genes_conf_min_counts output contains both conf_score_high and conf_score_low columns with marker genes of mismatched cell types negatively associated (conf_score_low)

## Limitations

- Hyperparameters such as training time (epoch number) influence confidence and variability scores and are not yet standardized
- Thresholding on confidence and variability to categorize cells introduces arbitrary cutoffs; the percentile q for hard-to-learn thresholds is chosen dataset-dependently (25%–90%)
- Input annotations are treated as nominal categories only; probabilistic/continuous annotations are deferred to future work
- Does not directly handle convolved or aggregated data from certain spatial transcriptomics technologies
- sc.pp.neighbors(adata) will overwrite the trainability-aware graph stored in adata.obsp['connectivities']

## Evidence

- [discussion] Confidence is defined as the average probability assigned by the model to the correct label across all epochs; variability is the standard deviation of that probability across epochs.: [evidence span withheld — anchor status: not_found]
