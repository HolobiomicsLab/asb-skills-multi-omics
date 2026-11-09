---
name: disease-state-characterization-in-beta-cells
description: Use when single-cell RNA-seq data from disease/treatment conditions (e.g.
  mSTZ β cells annotated by condition) must be partitioned into disease-related vs.
  healthy-like cell states using training-dynamics scores rather than expression alone.
when_to_use_negative:
- Data has not been through the upstream Annotatability DNN training step, so 'conf'
  and 'var' scores are absent — a standard expression-only Scanpy neighbors graph
  would then be the appropriate substrate, not make_conf_graph
- Data is convolved or aggregated, such as that obtained from certain spatial transcriptomics
  technologies, which Annotatability does not directly handle
- The analysis goal is standard cell-type clustering by expression alone rather than
  characterizing subpopulations (e.g. hard-to-learn cells) relative to an existing
  annotation signal
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3512
tools:
- name: Annotatability
  role: Provides make_conf_graph for the trainability-aware graph (alpha=0.9, k=15)
    and rank_genes_conf_min_counts for annotation-trainability gene scoring
  repo: https://github.com/nitzanlab/Annotatability
- name: Scanpy
  role: Runs Louvain community detection and UMAP visualization (sc.tl.umap, sc.pl.umap)
    on the trainability-aware graph stored in adata.obsp['connectivities']
  repo: https://scanpy.readthedocs.io/en/stable/
- name: PyTorch
  role: Backend for the DNN training-dynamics monitoring that generates the conf/var
    scores upstream of clustering
- name: scipy
  role: Stores the trainability-aware connectivities graph as a sparse matrix in adata.obsp
    via scipy.sparse
- name: pandas
  role: Tabulates cluster-by-treatment composition fractions for cluster_composition.csv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:aa5b4cd5bf3b01104ee4d6f557a93676713a46b6b3863c873fa06b6b49ef6fd6
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
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# disease-state-characterization-in-beta-cells

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Characterizes disease-related β cell states in the mSTZ-induced diabetic mouse model by building a trainability-aware graph from DNN training-dynamics scores (confidence 'conf' and variability 'var') with Annotatability, then running Louvain clustering to separate disease, intermediate, and healthy-like β cell populations and quantify treatment-specific composition.

## When to use

Use when single-cell RNA-seq data from disease/treatment conditions (e.g. mSTZ β cells annotated by condition) must be partitioned into disease-related vs. healthy-like cell states using training-dynamics scores rather than expression alone.

## When NOT to use

- Data has not been through the upstream Annotatability DNN training step, so 'conf' and 'var' scores are absent — a standard expression-only Scanpy neighbors graph would then be the appropriate substrate, not make_conf_graph
- Data is convolved or aggregated, such as that obtained from certain spatial transcriptomics technologies, which Annotatability does not directly handle
- The analysis goal is standard cell-type clustering by expression alone rather than characterizing subpopulations (e.g. hard-to-learn cells) relative to an existing annotation signal

## Inputs

- AnnData (h5ad) of mSTZ β cells with per-cell training-dynamics scores in adata.obs['conf'] and adata.obs['var'] from an upstream Annotatability DNN training run
- Per-cell treatment/condition annotation (control, STZ, insulin-treated) stored as an observation column
- Gene expression matrix (Scanpy-compatible counts) for graph construction and marker scoring

## Outputs

- Louvain communities on the trainability-aware graph separating disease, intermediate, and healthy-like β cell states
- UMAP embedding of the trainability-aware graph colored by confidence score (mstz_beta_conf_umap.png)
- Per-cluster treatment composition table (cluster_composition.csv) quantifying the fraction of control/STZ/insulin-treated cells per community
- Ranked genes with annotation-trainability association scores in adata.var['conf_score_high'] and adata.var['conf_score_low']
- Annotated AnnData written to mstz_beta_trainability.h5ad

## How to apply

Start from the β-cell AnnData carrying per-cell confidence ('conf') and variability ('var') scores computed from DNN training dynamics via Annotatability's models.follow_training_dyn_neural_net and probability_list_to_confidence_and_var. Build the trainability-aware graph with metrics.make_conf_graph(adata.copy(), alpha=0.9, k=15), which integrates gene expression with training-dynamics statistics, and store the returned connectivities graph via scipy.sparse into adata.obsp['connectivities'], replacing the standard Scanpy neighbors graph (running sc.pp.neighbors would overwrite it with the expression-only graph). Run Louvain community detection on this graph to partition cells into three clusters corresponding to disease, intermediate, and healthy-like states, then visualize with sc.tl.umap(adata) and sc.pl.umap(adata, color='conf'), coloring communities by treatment condition and key marker genes. Score genes for annotation-trainability association with metrics.rank_genes_conf_min_counts(adata), reading adata.var['conf_score_high'] (positive association) and adata.var['conf_score_low'] (negative association) to identify marker-expression patterns distinguishing the clusters. Tabulate the fraction of cells from each treatment (control, STZ, insulin-treated) per cluster to extract hard-to-learn insulin-treated β cells showing healthy-like marker expression; persist results as mstz_beta_trainability.h5ad, mstz_beta_conf_umap.png, and cluster_composition.csv.

## Related tools

- **Annotatability** (Provides make_conf_graph for the trainability-aware graph (alpha=0.9, k=15) and rank_genes_conf_min_counts for annotation-trainability gene scoring) — https://github.com/nitzanlab/Annotatability
- **Scanpy** (Runs Louvain community detection and UMAP visualization (sc.tl.umap, sc.pl.umap) on the trainability-aware graph stored in adata.obsp['connectivities']) — https://scanpy.readthedocs.io/en/stable/
- **PyTorch** (Backend for the DNN training-dynamics monitoring that generates the conf/var scores upstream of clustering)
- **scipy** (Stores the trainability-aware connectivities graph as a sparse matrix in adata.obsp via scipy.sparse)
- **pandas** (Tabulates cluster-by-treatment composition fractions for cluster_composition.csv)

## Examples

```
connectivities_graph, distance_graph = metrics.make_conf_graph(adata.copy(), alpha=0.9, k=15); adata.obsp['connectivities']=sp.csr_matrix(connectivities_graph); sc.tl.louvain(adata)
```

## Evaluation signals

- The three Louvain communities on the trainability-aware graph reproduce the disease/intermediate/healthy-like β cell separation
- Insulin-treated mSTZ β cells are enriched in the healthy-like cluster compared with control and untreated STZ groups
- Hard-to-learn insulin-treated β cells exhibit healthy-like marker expression, as identified via conf_score_high/conf_score_low gene rankings
- adata.obsp['connectivities'] holds the make_conf_graph output (scipy.sparse CSR), not the expression-only Scanpy neighbors graph — verify sc.pp.neighbors was not run afterwards
- Outputs mstz_beta_trainability.h5ad, mstz_beta_conf_umap.png, and cluster_composition.csv are produced with expected schema (per-cluster treatment fractions summing to 1)

## Limitations

- Thresholding on confidence and variability scores for categorizing cells introduces arbitrary cutoffs that could influence interpretation of the data
- Hyperparameters such as training time (epoch_num) are not standardized and may influence the confidence and variability scores feeding the graph
- The percentile q for hard-to-learn thresholds is chosen dataset-dependently (25%–90%), a manual, non-standardized choice
- Annotatability currently treats input annotations as nominal categories only, so continuous or probabilistic condition labels cannot be used directly

## Evidence

- [discussion] Optional trainability-aware graph embedding integrating expression and training dynamics: [evidence span withheld — anchor status: not_found]
