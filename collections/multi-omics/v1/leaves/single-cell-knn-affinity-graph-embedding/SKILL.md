---
name: single-cell-knn-affinity-graph-embedding
description: Use when annotated single-cell data carries per-cell training-dynamics
  scores (adata.obs['conf'], adata.obs['var']) and you need a graph embedding that
  integrates annotation confidence with expression for signal-specific downstream
  analyses.
when_to_use_negative:
- Input annotations are not nominal categories (Annotatability currently treats input
  annotations as nominal categories only; probabilistic and continuous annotations
  are deferred to future work)
- Data is convolved or aggregated, such as that obtained from certain spatial transcriptomics
  technologies, which Annotatability does not directly handle
- No per-cell training-dynamics scores are available and training a DNN is out of
  scope — a standard Scanpy expression graph (sc.pp.neighbors) would be the appropriate
  graph instead
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Annotatability
  role: Provides metrics.make_conf_graph, which builds the connectivities and distance
    graphs blending expression distance with confidence-score difference
  repo: https://github.com/nitzanlab/Annotatability
- name: Scanpy
  role: Used for sc.tl.umap embedding, sc.pl.umap visualization, and the baseline
    sc.pp.neighbors expression graph against which the trainability-aware graph is
    compared
  repo: https://scanpy.readthedocs.io/en/stable/
- name: scipy
  role: sp.csr_matrix is used to write the trainability-aware graph into adata.obsp['connectivities']
- name: numpy
  role: Dependency of the Annotatability package used for array handling of confidence/variability
    scores
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:05767b2660010b1a4ac4457311f2e0f7c6d174736fb9e30b63e76eafa5c170d4
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
  - Python
  - Scanpy
  - numpy
  - scipy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 12
---

# single-cell-knn-affinity-graph-embedding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a trainability-aware kNN graph that blends PCA-reduced gene-expression distance with per-cell Annotatability confidence-score differences (Gaussian-kernel affinity over k=15 neighbors, alpha=0.9) so downstream UMAP embedding captures signal-specific gradients (e.g. E/M markers along EMT) rather than only treatment-driven clustering.

## When to use

Use when annotated single-cell data carries per-cell training-dynamics scores (adata.obs['conf'], adata.obs['var']) and you need a graph embedding that integrates annotation confidence with expression for signal-specific downstream analyses.

## When NOT to use

- Input annotations are not nominal categories (Annotatability currently treats input annotations as nominal categories only; probabilistic and continuous annotations are deferred to future work)
- Data is convolved or aggregated, such as that obtained from certain spatial transcriptomics technologies, which Annotatability does not directly handle
- No per-cell training-dynamics scores are available and training a DNN is out of scope — a standard Scanpy expression graph (sc.pp.neighbors) would be the appropriate graph instead

## Inputs

- Annotated AnnData object (e.g. filtered EMT scRNA-seq data) with per-cell confidence and variability scores stored in adata.obs['conf'] and adata.obs['var']
- alpha parameter controlling the weight of confidence-score difference vs expression distance (default 0.9)
- k parameter for number of nearest neighbors (default 15)

## Outputs

- connectivities_graph and distance_graph returned by metrics.make_conf_graph
- adata.obsp['connectivities'] holding the trainability-aware graph as a scipy.sparse csr_matrix
- UMAP embeddings and plots (e.g. emt_conf_graph_umap.png) contrasting the trainability-aware graph against the baseline Scanpy expression graph

## How to apply

Start from an annotated AnnData object with per-cell training-dynamics scores in adata.obs['conf'] and adata.obs['var'], obtained from upstream DNN training with models.follow_training_dyn_neural_net and models.probability_list_to_confidence_and_var. Call metrics.make_conf_graph(adata.copy(), alpha=0.9, k=15) to obtain connectivities_graph and distance_graph, where the distance blends PCA-reduced expression distance with confidence-score difference (weight alpha=0.9) and edges are formed by Gaussian-kernel affinity over the k=15 nearest neighbors. Overwrite the neighbor graph via adata.obsp['connectivities']=sp.csr_matrix(connectivities_graph) using scipy.sparse, keeping a separate copy of the adata with the standard Scanpy sc.pp.neighbors expression graph as the baseline. Run sc.tl.umap(adata) and sc.pl.umap(adata, color='conf') on both graphs, then overlay E/M marker expression. Note the README warning: calling sc.pp.neighbors(adata) afterwards will overwrite the trainability-aware graph in adata.obsp['connectivities'], so order the calls carefully.

## Related tools

- **Annotatability** (Provides metrics.make_conf_graph, which builds the connectivities and distance graphs blending expression distance with confidence-score difference) — https://github.com/nitzanlab/Annotatability
- **Scanpy** (Used for sc.tl.umap embedding, sc.pl.umap visualization, and the baseline sc.pp.neighbors expression graph against which the trainability-aware graph is compared) — https://scanpy.readthedocs.io/en/stable/
- **scipy** (sp.csr_matrix is used to write the trainability-aware graph into adata.obsp['connectivities'])
- **numpy** (Dependency of the Annotatability package used for array handling of confidence/variability scores)

## Examples

```
import scipy.sparse as sp; connectivities_graph, distance_graph = metrics.make_conf_graph(adata.copy(), alpha=0.9, k=15); adata.obsp['connectivities'] = sp.csr_matrix(connectivities_graph); sc.tl.umap(adata); sc.pl.umap(adata, color='conf')
```

## Evaluation signals

- Confirm the trainability-aware graph was written into adata.obsp['connectivities'] as a scipy csr_matrix rather than being overwritten by a subsequent sc.pp.neighbors call
- Reproduce the reported contrast: the baseline expression graph clusters cells by treatment (mock vs TGF-beta), whereas the trainability-aware graph varies E/M marker expression along its structure (Fig. 4g-j pattern)
- Confidence in the E/M labels should change gradually along the trainability-aware graph structure, consistent with Fig. 4g-i
- Verify alpha=0.9 and k=15 were passed to metrics.make_conf_graph and that a copy of the adata (not the original) was used

## Limitations

- Annotatability does not directly handle convolved or aggregated data, such as that obtained from certain spatial transcriptomics technologies
- Hyperparameters such as alpha and k can be adjusted but their determination is not standardized; thresholding choices may influence interpretation of the data
- Annotatability currently treats input annotations as nominal categories only; generalization to probabilistic and continuous annotations is deferred to future work

## Evidence

- [results] Trainability-aware graphs capture EMT progression: [evidence span withheld — anchor status: not_found]
