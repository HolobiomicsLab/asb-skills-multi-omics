---
name: single-cell-rna-seq-quality-control-filtering
description: Use when raw scRNA-seq AnnData (e.g., the EMT MCF10A/HuMEC mock vs TGF-β
  dataset) contains low-quality cells that must be removed before training a DNN or
  computing annotation-trainability scores.
when_to_use_negative:
- Input is already filtered and normalized, or is a convolved/aggregated matrix from
  spatial transcriptomics technologies that Annotatability does not directly handle
- Data is not cell-by-gene scRNA-seq counts (e.g., proteomics or bulk RNA-seq), where
  mitochondrial-fraction and expressed-gene-count cutoffs are not meaningful
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Scanpy
  role: Scanpy-based preprocessing for filtering cells/genes, HVG selection, and L2
    normalization before trainability analysis
  repo: https://github.com/nitzanlab/Annotatability
- name: Annotatability
  role: Consumes the QC-filtered AnnData to train a DNN and compute confidence/variability
    training-dynamics scores
  repo: https://github.com/nitzanlab/Annotatability
- name: Python
  role: Execution environment for the filtering and preprocessing pipeline (python
    >3.0)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:f07382a7e244f63eed9aae29994d59f9a08c3746a1883d63fa1266785e53e6b7
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
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-rna-seq-quality-control-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Filter low-quality cells from single-cell RNA-seq data using mitochondrial fraction, total count, and expressed-gene-count thresholds, then retain only genes expressed in >3 cells and select 3,000 highly variable genes, so downstream methods (e.g., Annotatability training dynamics) operate on clean inputs.

## When to use

Use when raw scRNA-seq AnnData (e.g., the EMT MCF10A/HuMEC mock vs TGF-β dataset) contains low-quality cells that must be removed before training a DNN or computing annotation-trainability scores.

## When NOT to use

- Input is already filtered and normalized, or is a convolved/aggregated matrix from spatial transcriptomics technologies that Annotatability does not directly handle
- Data is not cell-by-gene scRNA-seq counts (e.g., proteomics or bulk RNA-seq), where mitochondrial-fraction and expressed-gene-count cutoffs are not meaningful

## Inputs

- Raw scRNA-seq AnnData object (.h5ad) with cell-by-gene count matrix (e.g., EMT dataset: MCF10A and HuMEC, mock vs TGF-β)
- Observation-level annotations ('label' in adata.obs) for downstream trainability analysis

## Outputs

- Filtered AnnData with low-quality cells removed (mito ≥5%, genes >4,000, genes <200 excluded)
- Gene-filtered matrix restricted to genes expressed in >3 cells and 3,000 highly variable genes
- L2-normalized (variance-1 per gene) expression matrix ready for DNN training

## How to apply

Load the raw scRNA-seq data (AnnData/h5ad) and remove cells with high mitochondrial gene content (≥5% of total counts), cells with a high total count (more than 4,000 expressed genes), and cells expressing fewer than 200 genes; this mirrors the exact filtering applied to the EMT dataset in the article. After cell filtering, retain only genes expressed in more than 3 cells and keep the top 3,000 highly variable genes. As a preprocessing step for annotation-trainability analyses, scale the expression level of each gene to variance 1 (L2 normalization) before computing confidence-score associations. The filtered, HVG-selected, L2-normalized AnnData is then ready for DNN training with Annotatability's follow_training_dyn_neural_net on the 'label' observation.

## Related tools

- **Scanpy** (Scanpy-based preprocessing for filtering cells/genes, HVG selection, and L2 normalization before trainability analysis) — https://github.com/nitzanlab/Annotatability
- **Annotatability** (Consumes the QC-filtered AnnData to train a DNN and compute confidence/variability training-dynamics scores) — https://github.com/nitzanlab/Annotatability
- **Python** (Execution environment for the filtering and preprocessing pipeline (python >3.0))

## Examples

```
import scanpy as sc; adata = adata[(adata.obs['pct_counts_mt'] < 5) & (adata.obs['n_genes'] <= 4000) & (adata.obs['n_genes'] >= 200)]; sc.pp.filter_genes(adata, min_cells=3); sc.pp.highly_variable_genes(adata, n_top_genes=3000); adata = adata[:, adata.var['highly_variable']]; sc.pp.scale(adata, zero_center=False)
```

## Evaluation signals

- No retained cell has mitochondrial gene fraction ≥5% of total counts, more than 4,000 expressed genes, or fewer than 200 expressed genes
- Gene matrix is restricted to genes expressed in >3 cells and contains 3,000 highly variable genes
- Each retained gene has variance 1 (L2 normalization) in the final expression matrix
- Downstream training on the filtered data (iterNum=50, batch_size=256, num_layers=3) yields well-defined per-cell 'conf' and 'var' scores in adata.obs

## Limitations

- Thresholding on scores/cutoffs introduces arbitrary cutoffs that could influence data interpretation
- Hyperparameters such as training time are not standardized and may influence downstream confidence and variability scores
- Annotatability does not directly handle convolved or aggregated data, such as that obtained from certain spatial transcriptomics technologies

## Evidence

- [discussion] EMT dataset filtering thresholds: [evidence span withheld — anchor status: not_found]
