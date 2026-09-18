---
name: layer-edge-list-aggregation-across-omics-networks
description: Use when you have ≥2 edge-list files describing different relationship
  types between a shared node set and need a combined multiplex network object for
  random-walk analysis or layer-similarity statistics.
when_to_use_negative:
- The analysis requires layer-specific topological information to be preserved in
  the query graph — the aggregated network is a summarization of the input layers
  and has lost that information
- You need a multiplex of heterogeneous (bipartite-linked) networks of different node
  natures rather than a homogeneous node-set multiplex
- Layers carry weighted edges — the preassembled comprehensive multiplex layers use
  unweighted edges and the workflow is built around that
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0602
tools:
- name: RWRtoolkit
  role: Provides RWR_make_multiplex (R function and run_make_multiplex.R command-line
    script) to build the multiplex from an flist, parallelized to reduce construction
    from hours to seconds
  repo: http://github.com/dkainer/RWRtoolkit
- name: RandomWalkRestartMH
  role: Underlying package supplying create.multiplex (list of igraph objects), compute.adjacency.matrix,
    and compute.transition.matrix functionality
  repo: https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- name: RWRtoolkit-data
  role: Repository of preassembled multiplex RData objects and source layer edge lists
    (e.g. Comprehensive_Network_AT_d0.5_v02.RData with its nine layer versions/codes)
  repo: https://github.com/dkainer/RWRtoolkit-data
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:674a005069b77df2ff4e2801cfc04c87f050ea23d28907df046da42e75ba070e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkainer/RWRtoolkit
  tools:
  - R
  - RWRtoolkit
  - RWRtoolkit-data
  - RandomWalkRestartMH
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# layer-edge-list-aggregation-across-omics-networks

## Summary

Aggregates multiple heterogeneous omics edge-list layers (e.g. coexpression, PPI, regulation, metabolic) into a single multiplex network object with a supra-adjacency and normalized transition matrix using RWRtoolkit's parallelized RWR_make_multiplex at delta = 0.5.

## When to use

Use when you have ≥2 edge-list files describing different relationship types between a shared node set and need a combined multiplex network object for random-walk analysis or layer-similarity statistics.

## When NOT to use

- The analysis requires layer-specific topological information to be preserved in the query graph — the aggregated network is a summarization of the input layers and has lost that information
- You need a multiplex of heterogeneous (bipartite-linked) networks of different node natures rather than a homogeneous node-set multiplex
- Layers carry weighted edges — the preassembled comprehensive multiplex layers use unweighted edges and the workflow is built around that

## Inputs

- flist descriptor file: one row per layer with {file_path, name, group}, delimiters , \t | ; accepted
- Per-layer edge-list files (e.g. the nine A. thaliana layers CoEvolution-DUO, Coexpression Gene-Atlas, Knockout Similarity, PPI-6merged, PEN-Diversity, Predictive CG Methylation, Regulation-ATRM, Regulation-Plantregmap, Metabolic-AraCyc from dkainer/RWRtoolkit-data)
- delta value controlling inter-layer connectivity (0.5 in the reference workflow)

## Outputs

- Multiplex object (nw.mpo) with Pool_of_Nodes (26,605 unique nodes for the 9-layer Arabidopsis multiplex)
- Supra-adjacency matrix (nw.adj) and normalized transition matrix (nw.adjnorm)
- Aggregated network edge count (918,640 edges for the reference multiplex)
- RData file (e.g. Comprehensive_Network_AT_d0.5_v02.RData) suitable for downstream RWR_LOE / RWR_CV / RWR_netstats

## How to apply

Write an flist descriptor file where each row is a triple {file_path, name, group} (delimiters , \t | ; are accepted), listing each layer's edge list, layer name, and short code (e.g. DU, GA, KS, PP, PX, PY, RE, RP, RX for the nine Arabidopsis layers). Call RWR_make_multiplex on the flist with delta = 0.5 (the inter-layer jump weight used in the comprehensive Arabidopsis multiplex); the function constructs the multiplex list of igraph objects, the supra-adjacency matrix, and the normalized transition matrix. The paper parallelized this construction to reduce build time from hours to seconds — the 9-layer comprehensive Arabidopsis multiplex built in 59 seconds. Save the result as an RData object holding nw.mpo, nw.adj, and nw.adjnorm (e.g. Comprehensive_Network_AT_d0.5_v02.RData). Note that the aggregated network is a summarization of the input layers and has lost the unique topological information carried by each layer, so retain the multiplex object (not just an aggregated graph) if layer-specific structure matters downstream.

## Related tools

- **RWRtoolkit** (Provides RWR_make_multiplex (R function and run_make_multiplex.R command-line script) to build the multiplex from an flist, parallelized to reduce construction from hours to seconds) — http://github.com/dkainer/RWRtoolkit
- **RandomWalkRestartMH** (Underlying package supplying create.multiplex (list of igraph objects), compute.adjacency.matrix, and compute.transition.matrix functionality) — https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- **RWRtoolkit-data** (Repository of preassembled multiplex RData objects and source layer edge lists (e.g. Comprehensive_Network_AT_d0.5_v02.RData with its nine layer versions/codes)) — https://github.com/dkainer/RWRtoolkit-data

## Examples

```
library(RWRtoolkit); mpo <- RWRtoolkit::RWR_make_multiplex(flist = "flist.tsv", delta = 0.5)
```

## Evaluation signals

- Loaded RData object's nw.mpo Pool_of_Nodes contains 26,605 unique nodes for the nine-layer comprehensive Arabidopsis multiplex
- Aggregated network edge count equals 918,640 for the reference comprehensive multiplex at delta = 0.5
- Saved RData contains all three objects: nw.mpo, nw.adj, and nw.adjnorm
- nw.adjnorm is a proper (row-)normalized transition matrix derived from nw.adj
- All flist rows resolve to existing edge-list files and each layer carries a distinct name/group code (DU, GA, KS, PP, PX, PY, RE, RP, RX)

## Limitations

- In the original RandomWalkRestartMH implementation multiplex construction was not optimized and computational load increases exponentially with network size and layer count; RWRtoolkit's parallelization mitigates but does not remove scaling limits
- The aggregated network alone loses the unique topological information carried by each layer, so aggregation is a summarization step, not a replacement for the multiplex
- Requires R >= 4.1.0; installation can take ~20 minutes and is prone to Bioconductor SSL-certificate and devtools dependency (harfbuzz, fribidi, libgit2) failures
- No RWRtoolkit or RandomWalkRestartMH version number is stated in the article; pin via the Zenodo records (10.5281/zenodo.8315832, 10.5281/zenodo.13964143)

## Evidence

- [readme] Each row in the flist is a triple defining the network: {file_path, name, group}: [evidence span withheld — anchor status: not_found]
- [discussion] Parallelized construction reduces build time from hours to seconds: "With RWRtoolkit we parallelized this process to dramatically reduce the time to construct a large multiplex network from hours to seconds"
- [methods] create.multiplex constructs the multiplex network list of igraph objects: "create.multiplex, which constructs the multiplex network list of igraph objects"
- [intro] Aggregation is a summarization that loses layer-specific topology: "The aggregated network is a summarization of the input layers, and as such has lost the unique topological information carried by each layer."
- [discussion] Prior implementation scaled exponentially with network size and layer count: "the construction of the multiplex was not optimized. As the size of networks and the total number of layers increases, the computational load for constructing the multiplex increases exponentially."
