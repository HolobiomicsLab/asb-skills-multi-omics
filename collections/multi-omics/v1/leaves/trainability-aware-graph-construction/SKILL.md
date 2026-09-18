---
name: trainability-aware-graph-construction
description: Use when annotated single-cell AnnData carries per-cell confidence ('conf')
  and variability ('var') training-dynamics scores and downstream embedding/clustering
  should capture the annotated signal (e.g. E/M state, disease vs healthy-like) rather
  than confounds like treatment condition.
when_to_use_negative:
- Input AnnData lacks per-cell 'conf'/'var' training-dynamics scores (e.g. DNN training
  with follow_training_dyn_neural_net has not been run) — build a standard sc.pp.neighbors
  graph instead
- You need the standard expression-driven neighborhood graph, since calling sc.pp.neighbors(adata)
  after graph construction will overwrite adata.obsp['connectivities']
- Data are convolved or aggregated spatial transcriptomics measurements — Annotatability
  does not directly handle such data
tools:
- name: Annotatability
  role: Provides metrics.make_conf_graph, which constructs the trainability-aware
    connectivities and distance graphs blending expression distance with confidence-score
    difference
  repo: https://github.com/nitzanlab/Annotatability
- name: Scanpy
  role: Runs sc.tl.umap, sc.pl.umap, Louvain clustering and the baseline sc.pp.neighbors
    graph that must not overwrite the trainability-aware graph
  repo: https://scanpy.readthedocs.io/en/stable/
- name: scipy
  role: Stores the trainability-aware graph via scipy.sparse (sp.csr_matrix) into
    adata.obsp['connectivities']
- name: Annotatability_notebooks
  role: Reproducibility notebooks (e.g. EMT, mSTZ) demonstrating trainability-aware
    graph construction and comparison against the baseline expression graph
  repo: https://github.com/nitzanlab/Annotatability_notebooks
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:aea0c34df208066d3babb2549395714298427c108d728e42db0395530f69aff6
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
  - Annotatability_notebooks
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
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# trainability-aware-graph-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a signal-aware neighborhood graph that blends gene-expression similarity with similarity of Annotatability training-dynamics confidence scores (alpha=0.9, k=15 Gaussian-kernel affinity), enabling downstream UMAP embedding and Louvain clustering that track signals like EMT progression or disease-state gradients instead of treatment-driven clustering.

## When to use

Use when annotated single-cell AnnData carries per-cell confidence ('conf') and variability ('var') training-dynamics scores and downstream embedding/clustering should capture the annotated signal (e.g. E/M state, disease vs healthy-like) rather than confounds like treatment condition.

## When NOT to use

- Input AnnData lacks per-cell 'conf'/'var' training-dynamics scores (e.g. DNN training with follow_training_dyn_neural_net has not been run) — build a standard sc.pp.neighbors graph instead
- You need the standard expression-driven neighborhood graph, since calling sc.pp.neighbors(adata) after graph construction will overwrite adata.obsp['connectivities']
- Data are convolved or aggregated spatial transcriptomics measurements — Annotatability does not directly handle such data

## Inputs

- Annotated single-cell AnnData object (e.g. .h5ad) with per-cell training-dynamics scores stored in adata.obs['conf'] and adata.obs['var']
- Baseline expression neighbors graph from sc.pp.neighbors (for comparison, kept in a separate AnnData copy)
- alpha (edge-weight blend between expression distance and confidence difference, default 0.9) and k (nearest neighbors, 15)

## Outputs

- Trainability-aware connectivities graph written into adata.obsp['connectivities'] (scipy.sparse CSR matrix)
- Distance graph from make_conf_graph
- UMAP embedding colored by 'conf' and by E/M or disease markers (e.g. emt_conf_graph_umap.png, mstz_beta_conf_umap.png)
- Louvain communities over the trainability-aware graph (e.g. disease/intermediate/healthy-like beta-cell clusters) and annotated AnnData (e.g. mstz_beta_trainability.h5ad)

## How to apply

First ensure the AnnData object has adata.obs['conf'] and adata.obs['var'] computed upstream via models.follow_training_dyn_neural_net followed by models.probability_list_to_confidence_and_var. Call metrics.make_conf_graph(adata.copy(), alpha=0.9, k=15) to obtain a connectivities_graph and distance_graph in which edge weights blend PCA-reduced expression distance with confidence-score difference (alpha=0.9 controls the blend) and edges are formed by Gaussian-kernel affinity over the k=15 nearest neighbors. Critically, store the result as adata.obsp['connectivities']=sp.csr_matrix(connectivities_graph); note that a later call to sc.pp.neighbors would overwrite this graph with the standard expression graph, so keep a separate baseline copy if a comparison is desired. Then run sc.tl.umap(adata) (and optionally Louvain community detection) on the trainability-aware graph, and optionally score genes with metrics.rank_genes_conf_min_counts to read out conf_score_high/conf_score_low marker associations. The rationale is that the resulting graph integrates both expression and training-dynamics statistics, so confidence in labels changes gradually along the graph structure.

## Related tools

- **Annotatability** (Provides metrics.make_conf_graph, which constructs the trainability-aware connectivities and distance graphs blending expression distance with confidence-score difference) — https://github.com/nitzanlab/Annotatability
- **Scanpy** (Runs sc.tl.umap, sc.pl.umap, Louvain clustering and the baseline sc.pp.neighbors graph that must not overwrite the trainability-aware graph) — https://scanpy.readthedocs.io/en/stable/
- **scipy** (Stores the trainability-aware graph via scipy.sparse (sp.csr_matrix) into adata.obsp['connectivities'])
- **Annotatability_notebooks** (Reproducibility notebooks (e.g. EMT, mSTZ) demonstrating trainability-aware graph construction and comparison against the baseline expression graph) — https://github.com/nitzanlab/Annotatability_notebooks

## Examples

```
connectivities_graph, distance_graph = metrics.make_conf_graph(adata.copy(), alpha=0.9, k=15); adata.obsp['connectivities']=sp.csr_matrix(connectivities_graph); sc.tl.umap(adata); sc.pl.umap(adata, color='conf')
```

## Evaluation signals

- Confirm adata.obsp['connectivities'] holds the trainability-aware graph (a scipy.sparse CSR matrix) and was not overwritten by sc.pp.neighbors
- On EMT data, the baseline expression graph clusters cells by treatment (mock vs TGF-beta) while the trainability-aware graph shows E/M marker expression varying gradually along its structure, reproducing the Fig. 4g–j pattern
- On mSTZ beta cells, Louvain communities on the graph split into disease/intermediate/healthy-like states, with insulin-treated cells enriched in the healthy-like cluster
- metrics.rank_genes_conf_min_counts yields conf_score_high (positive) and conf_score_low (negative) gene associations consistent with the cluster marker expression
- alpha and k used match the reported values (alpha=0.9, k=15), and UMAP coloring by 'conf' shows gradual confidence change along the graph

## Limitations

- Hyperparameters such as alpha, k, and training time influence the graph and the confidence/variability scores and are not standardized
- Running sc.pp.neighbors(adata) after graph construction stores the expression neighbors graph in adata.obsp['connectivities'] instead of the trainability-aware graph
- Annotatability treats input annotations as nominal categories only and does not directly handle convolved or aggregated spatial transcriptomics data
- Thresholding on confidence and variability scores introduces arbitrary cutoffs that could influence data interpretation

## Evidence

- [results] finding on trainability-aware graphs capturing EMT progression: [evidence span withheld — anchor status: not_found]
