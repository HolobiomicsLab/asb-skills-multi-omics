---
name: emt-marker-expression-gradient-analysis
description: Use when annotated EMT single-cell data (AnnData with adata.obs['conf']
  and adata.obs['var'] from DNN training dynamics) should be embedded so the graph
  structure signals E/M progression instead of mock vs TGF-β treatment batches.
when_to_use_negative:
- When the goal is standard expression-based clustering or treatment-condition separation
  — sc.pp.neighbors on expression alone is the appropriate baseline, not the confidence-weighted
  graph
- When adata.obsp['connectivities'] already holds the trainability-aware graph and
  a subsequent sc.pp.neighbors call would overwrite it with the expression graph (the
  README explicitly warns of this)
- When the input data is convolved or aggregated (e.g. certain spatial transcriptomics
  technologies), which Annotatability does not directly handle
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Annotatability
  role: Provides metrics.make_conf_graph to build the trainability-aware connectivities/distance
    graphs blending expression distance with confidence-score difference
  repo: https://github.com/nitzanlab/Annotatability
- name: Scanpy
  role: Provides sc.pp.neighbors (baseline expression graph), sc.tl.umap for embedding,
    and sc.pl.umap for visualization colored by 'conf' and E/M markers
  repo: https://scanpy.readthedocs.io/en/stable/
- name: scipy
  role: scipy.sparse.sp.csr_matrix converts the make_conf_graph connectivities output
    for storage in adata.obsp['connectivities']
- name: PyTorch
  role: Runs the upstream DNN training (models.follow_training_dyn_neural_net) on
    cpu or cuda device that generates the confidence/variability scores used by the
    graph
- name: numpy
  role: Converts the returned confidence/variability tensors to per-cell obs values
    (all_conf.detach().numpy(), all_var.detach().numpy())
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s43588-024-00721-5
    title: Interpreting single-cell and spatial omics data using deep neural network
      training dynamics
  source_content_hash: sha256:4ac1b7f5c7ed3a245736d3687b69c816ed0df709cd30fbb1429640d69da9e9b1
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
  - scipy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# emt-marker-expression-gradient-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a trainability-aware graph embedding of EMT single-cell data that blends gene expression distance with Annotatability confidence-score difference (alpha=0.9, k=15), so that UMAP structure reflects E/M marker-expression gradients along epithelial-mesenchymal progression rather than treatment-driven clustering.

## When to use

Use when annotated EMT single-cell data (AnnData with adata.obs['conf'] and adata.obs['var'] from DNN training dynamics) should be embedded so the graph structure signals E/M progression instead of mock vs TGF-β treatment batches.

## When NOT to use

- When the goal is standard expression-based clustering or treatment-condition separation — sc.pp.neighbors on expression alone is the appropriate baseline, not the confidence-weighted graph
- When adata.obsp['connectivities'] already holds the trainability-aware graph and a subsequent sc.pp.neighbors call would overwrite it with the expression graph (the README explicitly warns of this)
- When the input data is convolved or aggregated (e.g. certain spatial transcriptomics technologies), which Annotatability does not directly handle

## Inputs

- Annotated EMT AnnData object (.h5ad) with per-cell training-dynamics scores in adata.obs['conf'] and adata.obs['var']
- E/M marker gene expression matrix in adata (raw counts or normalized expression)
- Hyperparameters: alpha=0.9 (confidence-vs-expression weight), k=15 (nearest neighbors)

## Outputs

- Trainability-aware connectivities and distance graphs stored in adata.obsp['connectivities'] (scipy.sparse csr_matrix)
- Baseline expression-only Scanpy neighbors graph (kept in a separate adata copy)
- UMAP embeddings and figures (e.g. emt_conf_graph_umap.png) colored by 'conf' and E/M markers, reproducing the Fig. 4g–j pattern

## How to apply

Start from the annotated EMT AnnData carrying per-cell training-dynamics scores in adata.obs['conf'] and adata.obs['var'], produced upstream by models.follow_training_dyn_neural_net and models.probability_list_to_confidence_and_var (epoch_num=50 in the README example). Call metrics.make_conf_graph(adata.copy(), alpha=0.9, k=15) to obtain connectivities_graph and distance_graph, where distance blends PCA-reduced expression distance with confidence-score difference weighted by alpha=0.9 and edges come from Gaussian-kernel affinity over the k=15 nearest neighbors. Overwrite the Scanpy graph with the trainability-aware graph via adata.obsp['connectivities']=sp.csr_matrix(connectivities_graph) (import scipy.sparse as sp), and keep a separate copy of the adata with the standard sc.pp.neighbors expression graph as the baseline. Run sc.tl.umap(adata) and sc.pl.umap(adata, color='conf') on both graphs, then overlay E/M marker expression. Validate the contrast reported in Fig. 4g–j: the baseline expression graph clusters cells by treatment (mock vs TGF-β), while the trainability-aware graph varies in E/M marker expression along its structure with confidence changing gradually along the graph.

## Related tools

- **Annotatability** (Provides metrics.make_conf_graph to build the trainability-aware connectivities/distance graphs blending expression distance with confidence-score difference) — https://github.com/nitzanlab/Annotatability
- **Scanpy** (Provides sc.pp.neighbors (baseline expression graph), sc.tl.umap for embedding, and sc.pl.umap for visualization colored by 'conf' and E/M markers) — https://scanpy.readthedocs.io/en/stable/
- **scipy** (scipy.sparse.sp.csr_matrix converts the make_conf_graph connectivities output for storage in adata.obsp['connectivities'])
- **PyTorch** (Runs the upstream DNN training (models.follow_training_dyn_neural_net) on cpu or cuda device that generates the confidence/variability scores used by the graph)
- **numpy** (Converts the returned confidence/variability tensors to per-cell obs values (all_conf.detach().numpy(), all_var.detach().numpy()))

## Examples

```
import scipy.sparse as sp; connectivities_graph, distance_graph = metrics.make_conf_graph(adata.copy(), alpha=0.9, k=15); adata.obsp['connectivities'] = sp.csr_matrix(connectivities_graph); sc.tl.umap(adata); sc.pl.umap(adata, color='conf')
```

## Evaluation signals

- Confirm the trainability-aware graph was written into adata.obsp['connectivities'] via sp.csr_matrix(connectivities_graph) and was not overwritten by any subsequent sc.pp.neighbors call
- Check make_conf_graph was called with the documented hyperparameters (alpha=0.9, k=15) and returns both connectivities_graph and distance_graph
- Verify the baseline (sc.pp.neighbors) UMAP clusters cells by treatment (mock vs TGF-β) while the trainability-aware UMAP shows E/M marker-expression gradients along its structure
- Confirm adata.obs['conf'] varies gradually along the trainability-aware graph structure, matching the reported Fig. 4g–i pattern
- Confirm the saved figure (e.g. emt_conf_graph_umap.png) contains the four-panel contrast (baseline vs trainability-aware, colored by conf and E/M markers)

## Limitations

- Running sc.pp.neighbors(adata) after building the trainability-aware graph will overwrite adata.obsp['connectivities'] with the expression-only graph
- The alpha weight (0.9) can be adjusted but the choice is manual; hyperparameters such as training time influence the confidence and variability scores and are not yet standardized
- Annotatability does not directly handle convolved or aggregated data, such as that obtained from certain spatial transcriptomics technologies

## Evidence
