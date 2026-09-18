---
name: anndata-data-manipulation
description: Use when a single-cell pipeline (e.g., Annotatability confidence/variability
  scoring and gene ranking) requires attaching, reading, or transforming results stored
  in AnnData slots (.obs, .var, .obsp) from .h5ad inputs.
when_to_use_negative:
- Input is a flat expression matrix or feature table rather than an annotated Scanpy
  AnnData object with cell labels
- Data is convolved or aggregated (e.g., certain spatial transcriptomics technologies),
  which Annotatability does not directly handle
- You need a standard cell-neighbor graph — sc.pp.neighbors(adata) would overwrite
  the trainability-aware graph in adata.obsp['connectivities']
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Scanpy
  role: Base framework for AnnData object manipulation, UMAP visualization, and .h5ad
    I/O underlying the Annotatability scoring functions
  repo: https://scanpy.readthedocs.io/en/stable/
- name: Annotatability
  role: Provides models.py and metrics.py functions that read and write AnnData slots
    (.obs, .var, .obsp) for confidence, variability, gene ranking, and trainability-aware
    graphs
  repo: https://github.com/nitzanlab/Annotatability
- name: numpy
  role: Converts PyTorch tensors (all_conf, all_var) to NumPy arrays for storage in
    adata.obs columns
- name: pandas
  role: Exports the ranked gene table with conf_score_high/conf_score_low columns
    to CSV (mstz_gene_trainability_scores.csv)
- name: scipy
  role: scipy.sparse used to wrap the trainability-aware connectivities graph as a
    CSR matrix for adata.obsp
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:3a50f00ddfd289550cd3644c950deeb82a28e4ed92dd16c2942df6587a56686f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/nitzanlab/Annotatability
  tools:
  - Annotatability
  - Python
  - Scanpy
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.15384615384615385
n_claims: 13
---

# anndata-data-manipulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Manipulate annotated single-cell AnnData objects (Scanpy .h5ad) as the central data container for the Annotatability workflow, storing per-cell training-dynamics scores in .obs, per-gene trainability association scores in .var, and the trainability-aware graph in .obsp.

## When to use

Use when a single-cell pipeline (e.g., Annotatability confidence/variability scoring and gene ranking) requires attaching, reading, or transforming results stored in AnnData slots (.obs, .var, .obsp) from .h5ad inputs.

## When NOT to use

- Input is a flat expression matrix or feature table rather than an annotated Scanpy AnnData object with cell labels
- Data is convolved or aggregated (e.g., certain spatial transcriptomics technologies), which Annotatability does not directly handle
- You need a standard cell-neighbor graph — sc.pp.neighbors(adata) would overwrite the trainability-aware graph in adata.obsp['connectivities']

## Inputs

- Annotated AnnData object (.h5ad) with cell labels stored as an observation column (label_key='label')
- Per-cell confidence and variability tensors (all_conf, all_var) from models.probability_list_to_confidence_and_var
- Prediction probability list (prob_list) from models.follow_training_dyn_neural_net

## Outputs

- AnnData with per-cell scores in adata.obs['conf'] and adata.obs['var']
- Ranked AnnData with per-gene scores in adata_ranked.var['conf_score_high'] and adata_ranked.var['conf_score_low']
- Gene trainability score table (e.g., mstz_gene_trainability_scores.csv)
- AnnData with trainability-aware graph stored in adata.obsp['connectivities']

## How to apply

Load the annotated dataset as a Scanpy AnnData object (e.g., mSTZ beta-cell data annotated by experimental condition) with the label stored as an observation column (label_key='label'). After training-dynamics analysis, attach the per-cell confidence and variability tensors to adata.obs ('conf', 'var') via .detach().numpy(), since downstream functions like metrics.rank_genes_conf_min_counts(adata) read these columns to compute gene scores. Run the ranking and read back results from adata_ranked.var['conf_score_high'] (positive annotation-trainability association) and adata_ranked.var['conf_score_low'] (negative association), then export the gene table to CSV (e.g., mstz_gene_trainability_scores.csv). For graph-based downstream analysis, replace adata.obsp['connectivities'] with the trainability-aware graph from metrics.make_conf_graph(adata.copy(), alpha=0.9, k=15) wrapped in sp.csr_matrix, taking care not to overwrite it by calling sc.pp.neighbors(adata). Preprocessing such as scaling each gene to variance 1 (L2 normalization) is applied to adata before scoring.

## Related tools

- **Scanpy** (Base framework for AnnData object manipulation, UMAP visualization, and .h5ad I/O underlying the Annotatability scoring functions) — https://scanpy.readthedocs.io/en/stable/
- **Annotatability** (Provides models.py and metrics.py functions that read and write AnnData slots (.obs, .var, .obsp) for confidence, variability, gene ranking, and trainability-aware graphs) — https://github.com/nitzanlab/Annotatability
- **numpy** (Converts PyTorch tensors (all_conf, all_var) to NumPy arrays for storage in adata.obs columns)
- **pandas** (Exports the ranked gene table with conf_score_high/conf_score_low columns to CSV (mstz_gene_trainability_scores.csv))
- **scipy** (scipy.sparse used to wrap the trainability-aware connectivities graph as a CSR matrix for adata.obsp)

## Examples

```
prob_list = models.follow_training_dyn_neural_net(adata, label_key='label',iterNum=epoch_num, device=device); all_conf, all_var = models.probability_list_to_confidence_and_var(prob_list, n_obs=adata.n_obs, epoch_num=epoch_num); adata.obs['conf'] = all_conf.detach().numpy(); adata_ranked = metrics.rank_genes_conf_min_counts(adata)
```

## Evaluation signals

- adata.obs contains float columns 'conf' and 'var' with one value per cell (length == adata.n_obs)
- adata_ranked.var contains 'conf_score_high' and 'conf_score_low' columns populated for every ranked gene
- The exported CSV (e.g., mstz_gene_trainability_scores.csv) has gene identifiers plus both score columns matching adata_ranked.var values
- The trainability-aware graph in adata.obsp['connectivities'] is a CSR matrix with the same n_obs x n_obs shape as the AnnData and was not overwritten by sc.pp.neighbors(adata)
- Genes are ranked by correlation/anticorrelation with the per-cell confidence vector after each gene was scaled to variance 1 (L2 normalization)

## Limitations

- Annotatability does not directly handle convolved or aggregated data, such as that obtained from certain spatial transcriptomics technologies
- Calling sc.pp.neighbors(adata) replaces the trainability-aware graph stored in adata.obsp['connectivities']
- Thresholding on confidence and variability scores introduces arbitrary cutoffs that could influence the interpretation of the data stored in .obs

## Evidence

- [readme] ranked genes stored in var columns: "adata_ranked.var['conf_score_high'] %annotation-trainability positive association score"
- [readme] Scanpy basis of AnnData handling: "Our code is based on [Scanpy](https://scanpy.readthedocs.io/en/stable/) package, but it can be easily adapted for any Python-based single-cell data analysis"
- [methods] trainability-aware graph stored in obsp: "connectivities_graph , distance_graph  = metrics.make_conf_graph(adata.copy(), alpha=0.9 , k=15)"
- [methods] gene ranking function reads AnnData: "adata_ranked = metrics.rank_genes_conf_min_counts(adata)"
- [discussion] L2 normalization preprocessing: "As a preprocessing step, the expression level of each gene is scaled to have variance 1 (L2 normalization)."
