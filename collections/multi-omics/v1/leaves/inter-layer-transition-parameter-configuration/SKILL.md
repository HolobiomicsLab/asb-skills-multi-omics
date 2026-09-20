---
name: inter-layer-transition-parameter-configuration
description: Use when building a multiplex network from multiple layer edge-list files
  and you must set the inter-layer transition probability (delta) and layer codes
  before running RWR.
when_to_use_negative:
- Your input is a preassembled multiplex RData object (e.g. Close_To_Phenotype_AT_d0.5_v01.RData)
  — load it directly instead of reconstructing.
- You only have a single homogeneous network (monoplex); multiplex construction and
  delta configuration are unnecessary.
- Your analysis requires layer-specific edge weights — the preassembled RWRtoolkit-data
  multiplexes contain unweighted edges only.
tools:
- name: RWRtoolkit
  role: Provides RWR_make_multiplex, which consumes the flist and delta parameter
    to build the multiplex object, adjacency, and normalized transition matrices in
    parallel.
  repo: https://github.com/dkainer/RWRtoolkit
- name: RandomWalkRestartMH
  role: Base package whose multiplex-construction functionality RWRtoolkit extends
    and parallelizes; supplies create.multiplex, compute.adjacency.matrix, and compute.transition.matrix.
  repo: https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- name: R
  role: Runtime environment; RWRtoolkit requires R >= 4.1.0 and is run as an R package
    or via Rscript command-line scripts.
  repo: https://github.com/dkainer/RWRtoolkit
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:bf34b9f458d6e739b6a57e92e5778dbd96f4a86259c54501da5439c470330942
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkainer/RWRtoolkit
  tools:
  - R
  - RWRtoolkit
  - RandomWalkRestartMH
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# inter-layer-transition-parameter-configuration

## Summary

Configures the inter-layer transition parameter (delta) and the layer descriptor file (flist) when constructing a multiplex network with RWRtoolkit's RWR_make_multiplex, so that random walkers can correctly hop between layers.

## When to use

Use when building a multiplex network from multiple layer edge-list files and you must set the inter-layer transition probability (delta) and layer codes before running RWR.

## When NOT to use

- Your input is a preassembled multiplex RData object (e.g. Close_To_Phenotype_AT_d0.5_v01.RData) — load it directly instead of reconstructing.
- You only have a single homogeneous network (monoplex); multiplex construction and delta configuration are unnecessary.
- Your analysis requires layer-specific edge weights — the preassembled RWRtoolkit-data multiplexes contain unweighted edges only.

## Inputs

- flist descriptor file listing each layer's edge-list path, layer name, and layer code (e.g. DU, GA, KS, PP, PX, PY, RE, RP, RX)
- per-layer edge-list files (e.g. the nine A. thaliana layer edge lists from the dkainer/RWRtoolkit-data repository)
- delta parameter value (inter-layer transition weight; 0.5 in the preassembled Arabidopsis multiplexes)

## Outputs

- multiplex object (nw.mpo) including the Pool_of_Nodes
- supra-adjacency matrix (nw.adj)
- normalized transition matrix (nw.adjnorm)
- RData file such as Comprehensive_Network_AT_d0.5_v02.RData

## How to apply

Write an flist descriptor file where each row is a triple {file_path, name, group} defining one network layer (e.g. the nine Arabidopsis thaliana layers with codes DU, GA, KS, PP, PX, PY, RE, RP, RX), delimited by comma, tab, pipe, or semicolon. Choose delta to weight intra- versus inter-layer transitions; the RWRtoolkit preassembled multiplexes use delta = 0.5 with unweighted edges, and the comprehensive 9-layer Arabidopsis multiplex is saved as Comprehensive_Network_AT_d0.5_v02.RData encoding that choice in its filename. Call RWRtoolkit::RWR_make_multiplex on the flist with the chosen delta, which parallelizes construction (reducing build time from hours to seconds — the full Arabidopsis multiplex built in 59 seconds) and returns the multiplex object, supra-adjacency matrix, and normalized transition matrix. Save the result as an RData object containing nw.mpo, nw.adj, and nw.adjnorm for downstream RWR_LOE, RWR_CV, and RWR_ShortestPaths analyses. Validate by loading the RData and checking the Pool_of_Nodes and aggregated edge counts against expected values.

## Related tools

- **RWRtoolkit** (Provides RWR_make_multiplex, which consumes the flist and delta parameter to build the multiplex object, adjacency, and normalized transition matrices in parallel.) — https://github.com/dkainer/RWRtoolkit
- **RandomWalkRestartMH** (Base package whose multiplex-construction functionality RWRtoolkit extends and parallelizes; supplies create.multiplex, compute.adjacency.matrix, and compute.transition.matrix.) — https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- **R** (Runtime environment; RWRtoolkit requires R >= 4.1.0 and is run as an R package or via Rscript command-line scripts.) — https://github.com/dkainer/RWRtoolkit

## Examples

```
library(RWRtoolkit); nw.mpo <- RWRtoolkit::RWR_make_multiplex(flist = "flist.tsv", delta = 0.5)
```

## Evaluation signals

- The saved RData object loads with exactly three components: nw.mpo, nw.adj, and nw.adjnorm.
- For the nine-layer comprehensive Arabidopsis multiplex at delta = 0.5, nw.mpo's Pool_of_Nodes contains 26,605 unique nodes and the aggregated network contains 918,640 edges.
- The flist parses into the expected number of layer triples (9 rows for the comprehensive network) with the correct layer codes (DU, GA, KS, PP, PX, PY, RE, RP, RX).
- The chosen delta is recorded in the output filename convention (e.g. Comprehensive_Network_AT_d0.5_v02.RData matches delta = 0.5).
- Construction completes in seconds rather than hours, confirming the parallelized RWR_make_multiplex path was used.

## Limitations

- The delta choice trades off intra- versus inter-layer transition probability; RWRtoolkit preassembled networks fix it at 0.5, and no per-layer weighting scheme for delta is provided here.
- Multiplex construction is computationally demanding: without the parallelized RWRtoolkit path, computational load increases exponentially as network size and layer count grow.
- The resulting aggregated network is a summarization of the input layers and loses the unique topological information carried by each layer.

## Evidence

- [readme] The first step in RWRtoolkit is to build the RData object that represents the multiplex network using `RWR_make_multiplex`.: [evidence span withheld — anchor status: not_found]
- [readme] Each row in the flist is a triple defining the network: {file_path, name, group}.: [evidence span withheld — anchor status: not_found]
- [readme] Contains the following network layers with unweighted edges and delta value = 0.5:: "Contains the following network layers with unweighted edges and delta value = 0.5:"
- [discussion] With RWRtoolkit we parallelized this process to dramatically reduce the time to construct a large multiplex network from hours to seconds: "With RWRtoolkit we parallelized this process to dramatically reduce the time to construct a large multiplex network from hours to seconds"
- [results] The full comprehensive Arabidopsis multiplex, built in 59 seconds, contains 26,605 unique nodes and 918,640 edges.: [evidence span withheld — anchor status: not_found]
