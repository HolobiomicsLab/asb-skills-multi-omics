---
name: kmeans-module-assignment
description: Use when a co-expression analysis needs module (cluster) assignment for
  genes after spectral embedding, specifically within the SGCRNA pipeline where CGM
  has produced CorData and GradData matrices and spectral clustering must yield per-gene
  cluster labels.
when_to_use_negative:
- The input is raw expression data that has not yet been passed through CGM — k-means
  module assignment requires the CorData/GradData matrices from preprocessing, not
  the raw count matrix
- The analysis goal requires scale-free-topology-based module detection or hierarchical
  clustering as in conventional WGCNA — SGCRNA deliberately replaces these with spectral
  clustering without scale-free constraints
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SGCRNAs
  role: Julia package providing the SpectralClustering function that performs Laplacian
    eigenvector computation and k-means cluster assignment on CGM outputs
  repo: https://github.com/C37H41N2O6/SGCRNAs.jl
- name: Julia
  role: Language runtime in which SGCRNAs' spectral clustering and k-means assignment
    are executed (Julia 1.11 or later recommended)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1101/2025.04.27.650628
    title: 'SGCRNA: Spectral Clustering-Guided Co-Expression Network Analysis Without
      Scale-Free Constraints for Multi-Omic Data'
  source_content_hash: sha256:52b7e6162a53f83140a0a42380f0a05d6646c218e3bb7073a96182de8f9795ba
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/C37H41N2O6/SGCRNAs.jl
  repo_url: https://github.com/C37H41N2O6/SGCRNAs.jl
  tools:
  - Julia
  - SGCRNAs
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.25
n_claims: 8
---

# kmeans-module-assignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Final step of SGCRNA's two-step co-expression workflow: k-means cluster assignment of genes based on approximate eigenvectors of the correlation/regression graph Laplacian, producing the `clust` module labels used for downstream SetNetwork and CorPhenMod analysis.

## When to use

Use when a co-expression analysis needs module (cluster) assignment for genes after spectral embedding, specifically within the SGCRNA pipeline where CGM has produced CorData and GradData matrices and spectral clustering must yield per-gene cluster labels.

## When NOT to use

- The input is raw expression data that has not yet been passed through CGM — k-means module assignment requires the CorData/GradData matrices from preprocessing, not the raw count matrix
- The analysis goal requires scale-free-topology-based module detection or hierarchical clustering as in conventional WGCNA — SGCRNA deliberately replaces these with spectral clustering without scale-free constraints

## Inputs

- CorData: correlation matrix output from SGCRNAs.CGM
- GradData: regression coefficient matrix output from SGCRNAs.CGM

## Outputs

- clust: per-gene cluster (module) assignment vector/matrix
- pos: positional/embedding data for downstream SetNetwork
- edge_data: weighted edge data used for network visualisation and module-phenotype correlation

## How to apply

After CGM preprocessing (optionally with `mode` set to :LESS, :SIGMA, or :FTEST; default :NONE) returns CorData (correlation matrix) and GradData (regression coefficient matrix), call `SpectralClustering(CorData, GradData)` to obtain `clust, pos, edge_data`. Internally, edge weights are derived from both cor and grad, a Laplacian is constructed, approximate eigenvectors are computed via Krylov-subspace methods, and k-means assigns each gene to a module. When running on a small synthetic or test matrix, override the defaults of tNodeNum=100 and pcas=99 with values less than roughly one-tenth of the variable count (but no less than 3). Validate the returned `clust` against known block-correlation ground truth when a synthetic matrix with known modules is used.

## Related tools

- **SGCRNAs** (Julia package providing the SpectralClustering function that performs Laplacian eigenvector computation and k-means cluster assignment on CGM outputs) — https://github.com/C37H41N2O6/SGCRNAs.jl
- **Julia** (Language runtime in which SGCRNAs' spectral clustering and k-means assignment are executed (Julia 1.11 or later recommended))

## Examples

```
clust, pos, edge_data = SpectralClustering(CorData, GradData);
```

## Evaluation signals

- The call `clust, pos, edge_data = SpectralClustering(CorData, GradData)` returns a non-empty `clust` whose maximum equals the number of detected modules (used as `k = maximum(clust[d])` in SetNetwork)
- On a synthetic matrix with a known block-correlation structure, the resulting `clust` assignments match the ground-truth modules
- With tNodeNum and pcas set below ~one-tenth of the variable count (minimum 3) on small inputs, clustering still completes and produces valid module labels
- Genes excluded by the CGM filter mode (near-zero expression variation or negligible regression slopes) do not appear in the clustering result
- Downstream SetNetwork/DrawNetwork accepts the clust labels and edge_data without dimension mismatches

## Limitations

- Clustering operates only on variables retained by the CGM preprocessing step; genes removed by :LESS, :SIGMA, or :FTEST filtering are absent from the final modules
- For small variable counts, the default tNodeNum=100 and pcas=99 must be reduced (to below ~one-tenth of variables, minimum 3), or the eigenvector computation may be invalid
- SGCRNA deliberately does not assume scale-free topology, so results may differ structurally from WGCNA modules and cannot be interpreted with WGCNA's scale-free heuristics

## Evidence
