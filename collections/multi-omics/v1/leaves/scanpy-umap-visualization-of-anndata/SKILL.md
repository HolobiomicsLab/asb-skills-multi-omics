---
name: scanpy-umap-visualization-of-anndata
description: Use when an AnnData object already carries a neighbor graph in adata.obsp['connectivities']
  (expression-based or confidence-based) and you need a 2D UMAP embedding colored
  by confidence scores or markers, e.g. to compare treatment clustering vs E/M marker
  gradients.
when_to_use_negative:
- Annotatability does not directly handle convolved or aggregated data, such as that
  obtained from certain spatial transcriptomics technologies — do not apply this AnnData-UMAP
  workflow to aggregated spatial inputs.
- Do not call sc.pp.neighbors(adata) when the trainability-aware graph is already
  in adata.obsp['connectivities'], since it silently overwrites the confidence graph
  with the expression graph.
- If hyperparameters such as training time (epoch_num) are not standardized, downstream
  confidence scores — and hence the colored UMAP — may not be reproducible.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: Scanpy
  role: Provides sc.tl.umap for embedding the chosen neighbor graph and sc.pl.umap
    for visualization colored by 'conf' or markers; sc.pp.neighbors supplies the baseline
    expression graph.
  repo: https://scanpy.readthedocs.io/en/stable/
- name: Annotatability
  role: metrics.make_conf_graph builds the trainability-aware connectivities/distance
    graphs (alpha=0.9, k=15) that replace adata.obsp['connectivities'] before UMAP.
  repo: https://github.com/nitzanlab/Annotatability
- name: scipy
  role: sp.csr_matrix casts the connectivities graph for storage in adata.obsp['connectivities'].
- name: numpy
  role: Detaches confidence/variability tensors into adata.obs arrays (all_conf.detach().numpy())
    used as UMAP colors.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:575ad5cabcd82309ef7eb931e23e621c05a33223698318930c5d4e0a6581bc6e
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
  - scipy
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# scanpy-umap-visualization-of-anndata

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Embed and visualize annotated single-cell AnnData objects with Scanpy's UMAP on a chosen neighbor graph — here either the standard expression graph from sc.pp.neighbors or the trainability-aware graph written into adata.obsp['connectivities'] by Annotatability's make_conf_graph.

## When to use

Use when an AnnData object already carries a neighbor graph in adata.obsp['connectivities'] (expression-based or confidence-based) and you need a 2D UMAP embedding colored by confidence scores or markers, e.g. to compare treatment clustering vs E/M marker gradients.

## When NOT to use

- Annotatability does not directly handle convolved or aggregated data, such as that obtained from certain spatial transcriptomics technologies — do not apply this AnnData-UMAP workflow to aggregated spatial inputs.
- Do not call sc.pp.neighbors(adata) when the trainability-aware graph is already in adata.obsp['connectivities'], since it silently overwrites the confidence graph with the expression graph.
- If hyperparameters such as training time (epoch_num) are not standardized, downstream confidence scores — and hence the colored UMAP — may not be reproducible.

## Inputs

- AnnData object (adata) with per-cell training-dynamics scores in adata.obs['conf'] and adata.obs['var']
- Neighbor graph stored in adata.obsp['connectivities'] — either from sc.pp.neighbors or from metrics.make_conf_graph with alpha=0.9, k=15
- Color annotation, e.g. adata.obs['conf'] or E/M marker gene expression

## Outputs

- 2D UMAP coordinates added to adata.obsm['X_umap']
- UMAP scatter plot colored by 'conf' and/or E/M markers (e.g. saved as emt_conf_graph_umap.png)

## How to apply

Ensure the desired graph is stored in adata.obsp['connectivities']: either the Scanpy expression graph from sc.pp.neighbors(adata) (baseline) or the trainability-aware graph from metrics.make_conf_graph(adata.copy(), alpha=0.9, k=15) assigned via sp.csr_matrix(connectivities_graph). Crucially, do NOT call sc.pp.neighbors after installing the confidence graph, because it will overwrite adata.obsp['connectivities'] with the expression-only neighbors. Then run sc.tl.umap(adata) to compute the embedding and sc.pl.umap(adata, color='conf') to color cells by the Annotatability confidence score in adata.obs['conf']; overlay E/M marker expression on the same embedding. On the EMT data, validate the contrast: the baseline expression graph clusters cells by treatment (mock vs TGF-β), whereas the trainability-aware graph varies in E/M marker expression along its structure (Fig. 4g–j pattern).

## Related tools

- **Scanpy** (Provides sc.tl.umap for embedding the chosen neighbor graph and sc.pl.umap for visualization colored by 'conf' or markers; sc.pp.neighbors supplies the baseline expression graph.) — https://scanpy.readthedocs.io/en/stable/
- **Annotatability** (metrics.make_conf_graph builds the trainability-aware connectivities/distance graphs (alpha=0.9, k=15) that replace adata.obsp['connectivities'] before UMAP.) — https://github.com/nitzanlab/Annotatability
- **scipy** (sp.csr_matrix casts the connectivities graph for storage in adata.obsp['connectivities'].)
- **numpy** (Detaches confidence/variability tensors into adata.obs arrays (all_conf.detach().numpy()) used as UMAP colors.)

## Examples

```
connectivities_graph, distance_graph = metrics.make_conf_graph(adata.copy(), alpha=0.9, k=15); adata.obsp['connectivities']=sp.csr_matrix(connectivities_graph); sc.tl.umap(adata); sc.pl.umap(adata, color='conf')
```

## Evaluation signals

- Verify the trainability-aware graph was written into adata.obsp['connectivities'] (as sp.csr_matrix) rather than being overwritten by sc.pp.neighbors.
- The baseline expression graph UMAP clusters cells by treatment (mock vs TGF-β), while the trainability-aware graph UMAP shows E/M marker expression gradients (reproduces Fig. 4g–j).
- Confidence in the E/M labels changes gradually along the trainability-aware graph structure.
- The distance graph blends PCA-reduced expression distance with confidence-score difference at weight alpha=0.9 over k=15 nearest neighbors, as specified in make_conf_graph.
- The figure is saved (e.g. emt_conf_graph_umap.png) with both embeddings side by side.

## Limitations

- Using sc.pp.neighbors(adata) after computing the confidence graph overwrites adata.obsp['connectivities'] with the expression-only neighbors graph.
- Thresholding on confidence and variability scores introduces arbitrary cutoffs that could influence data interpretation.
- Hyperparameters such as training time (iterNum, default 100) may influence confidence and variability scores and are not standardized, affecting embedding reproducibility.
- No changelog found for the referenced repository, limiting reproducibility of code versions.

## Evidence

- [results] the trainability-aware graphs integrate information about both training dynamics statistics and gene expression, and they capture the progression of cells along the EMT: [evidence span withheld — anchor status: not_found]
