---
name: louvain-community-clustering-single-cell
description: Use when single-cell AnnData carries per-cell training-dynamics scores
  ('conf', 'var') from Annotatability and the goal is to cluster cells by annotation-trainability
  signal, e.g. recovering disease/intermediate/healthy-like β cell states and comparing
  treatment subgroups.
when_to_use_negative:
- 'When a standard expression-only neighborhood structure is desired: calling sc.pp.neighbors(adata)
  overwrites adata.obsp[''connectivities''] with the Scanpy neighbors graph, replacing
  the trainability-aware graph'
- Data are convolved or aggregated, e.g. from certain spatial transcriptomics technologies,
  which Annotatability does not directly handle
- Input annotations are probabilistic or continuous rather than nominal categories,
  which Annotatability currently does not support
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Annotatability
  role: Provides metrics.make_conf_graph to build the trainability-aware connectivities/distance
    graphs (alpha=0.9, k=15) and metrics.rank_genes_conf_min_counts for annotation-trainability
    gene scoring
  repo: https://github.com/nitzanlab/Annotatability
- name: Scanpy
  role: Runs Louvain community detection and UMAP visualization (sc.tl.umap, sc.pl.umap)
    on the trainability-aware graph stored in adata.obsp['connectivities']
- name: scipy
  role: Stores the connectivities graph as a sparse CSR matrix via sp.csr_matrix into
    adata.obsp['connectivities']
- name: PyTorch
  role: Executes the upstream DNN training (follow_training_dyn_neural_net) that generates
    the confidence/variability scores feeding the graph
- name: pandas
  role: Tabulates cluster composition by treatment condition (cluster_composition.csv)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:9c047a0437362258d10efb41e5fad89b4ab92551b41b46d1d4463346a1496830
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# louvain-community-clustering-single-cell

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run Louvain community detection on a trainability-aware graph (connectivities built by Annotatability's metrics.make_conf_graph integrating gene expression with DNN training-dynamics confidence/variability scores) to partition single-cell data into signal-associated communities, as applied to mSTZ β cells to separate disease, intermediate and healthy-like states.

## When to use

Use when single-cell AnnData carries per-cell training-dynamics scores ('conf', 'var') from Annotatability and the goal is to cluster cells by annotation-trainability signal, e.g. recovering disease/intermediate/healthy-like β cell states and comparing treatment subgroups.

## When NOT to use

- When a standard expression-only neighborhood structure is desired: calling sc.pp.neighbors(adata) overwrites adata.obsp['connectivities'] with the Scanpy neighbors graph, replacing the trainability-aware graph
- Data are convolved or aggregated, e.g. from certain spatial transcriptomics technologies, which Annotatability does not directly handle
- Input annotations are probabilistic or continuous rather than nominal categories, which Annotatability currently does not support

## Inputs

- AnnData (h5ad) of mSTZ β cells with adata.obs['conf'] and adata.obs['var'] training-dynamics scores from Annotatability DNN training
- Cell-level annotation key (e.g. treatment condition: control, STZ, insulin-treated) used for DNN training and cluster composition tabulation
- Gene expression matrix preprocessed per Annotatability (e.g. highly variable genes)

## Outputs

- Louvain communities (three clusters: disease, intermediate, healthy-like) stored on the AnnData
- mstz_beta_trainability.h5ad with the trainability-aware graph in adata.obsp['connectivities']
- mstz_beta_conf_umap.png UMAP visualization colored by confidence and community
- cluster_composition.csv with fraction of cells per treatment within each cluster
- Gene ranking in adata_ranked.var['conf_score_high'] and ['conf_score_low'] from metrics.rank_genes_conf_min_counts

## How to apply

Start from an mSTZ β-cell AnnData with per-cell 'conf' and 'var' scores in adata.obs computed by follow_training_dyn_neural_net and probability_list_to_confidence_and_var. Build the trainability-aware graph with metrics.make_conf_graph(adata.copy(), alpha=0.9, k=15), which returns connectivities and distance graphs integrating gene expression with training-dynamics statistics, then store it as adata.obsp['connectivities'] via scipy.sparse (replacing the standard Scanpy neighbors graph). Run Louvain community detection on this graph to partition cells into three communities corresponding to disease, intermediate and healthy-like states; visualize with sc.tl.umap(adata) and sc.pl.umap(adata, color='conf'), coloring communities by treatment condition and marker genes. Score genes for annotation-trainability association with metrics.rank_genes_conf_min_counts(adata), reading adata_ranked.var['conf_score_high'] and ['conf_score_low']. Finally tabulate the fraction of cells from each treatment (control, STZ, insulin-treated) per cluster and write mstz_beta_trainability.h5ad, mstz_beta_conf_umap.png and cluster_composition.csv.

## Related tools

- **Annotatability** (Provides metrics.make_conf_graph to build the trainability-aware connectivities/distance graphs (alpha=0.9, k=15) and metrics.rank_genes_conf_min_counts for annotation-trainability gene scoring) — https://github.com/nitzanlab/Annotatability
- **Scanpy** (Runs Louvain community detection and UMAP visualization (sc.tl.umap, sc.pl.umap) on the trainability-aware graph stored in adata.obsp['connectivities'])
- **scipy** (Stores the connectivities graph as a sparse CSR matrix via sp.csr_matrix into adata.obsp['connectivities'])
- **PyTorch** (Executes the upstream DNN training (follow_training_dyn_neural_net) that generates the confidence/variability scores feeding the graph)
- **pandas** (Tabulates cluster composition by treatment condition (cluster_composition.csv))

## Examples

```
connectivities_graph, distance_graph = metrics.make_conf_graph(adata.copy(), alpha=0.9, k=15); adata.obsp['connectivities'] = sp.csr_matrix(connectivities_graph)
```

## Evaluation signals

- Louvain yields exactly three communities reproducing the disease/intermediate/healthy-like separation of mSTZ β cells
- Insulin-treated mSTZ β cells are enriched in the healthy-like cluster relative to control and STZ groups in cluster_composition.csv
- The hard-to-learn insulin-treated β cell subset (low 'conf') exhibits healthy-like marker expression per conf_score_high/conf_score_low gene rankings
- adata.obsp['connectivities'] equals the sparse CSR matrix returned by metrics.make_conf_graph(adata.copy(), alpha=0.9, k=15), not the Scanpy neighbors graph
- UMAP computed on the trainability-aware graph shows coherent structure when colored by 'conf' (sc.pl.umap(adata, color='conf'))

## Limitations

- Running sc.pp.neighbors(adata) overwrites adata.obsp['connectivities'] with the standard Scanpy neighbors graph, destroying the trainability-aware graph
- Thresholding on confidence and variability scores introduces arbitrary cutoffs that could influence data interpretation
- Hyperparameters such as training time (epoch_num) influence the determination of the confidence and variability scores and are not standardized
- The alpha parameter of make_conf_graph can be adjusted, and the percentile q for hard-to-learn thresholds is chosen dataset-dependently (25%–90%), a manual non-standardized choice

## Evidence

- [discussion] Missing signal on thresholding: [evidence span withheld — anchor status: not_found]
