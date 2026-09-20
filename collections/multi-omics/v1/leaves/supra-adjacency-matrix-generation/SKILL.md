---
name: supra-adjacency-matrix-generation
description: Use when the input is a set of layer edge-list files (an flist of {file_path,
  name, group} triples) that must be assembled into a multiplex object with a supra-adjacency
  matrix for downstream RWR analysis.
when_to_use_negative:
- The input is a single monoplex/homogeneous network with one edge type — no multiplex
  or supra-adjacency structure is needed
- Only an aggregated (summarized) network is required and layer-specific topological
  information can be discarded
- Layers are not expressible as edge-lists loadable as igraph objects per the flist
  triple format
edam_operation: http://edamontology.org/operation_0271
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: RWRtoolkit
  role: Provides RWR_make_multiplex and the parallelized create.multiplex / compute.adjacency.matrix
    / compute.transition.matrix pipeline used to build the supra-adjacency and transition
    matrices
  repo: https://github.com/dkainer/RWRtoolkit
- name: RandomWalkRestartMH
  role: Original sequential implementation of create.multiplex, compute.adjacency.matrix,
    and compute.transition.matrix that RWRtoolkit extends and parallelizes
  repo: https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- name: R
  role: Runtime environment (R >= 4.1.0) in which the multiplex construction functions
    are executed and RData outputs saved
- name: igraph
  role: Used to generate the random Erdős–Rényi layer networks in the benchmarking
    grid and as the graph representation of each layer
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:beee869fc3e7c71cf504b194780db65047369be43eca812c3cc3ada3f4678e84
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkainer/RWRtoolkit
  tools:
  - R
  - RWRtoolkit
  - RandomWalkRestartMH
  - igraph
  edam_operation: http://edamontology.org/operation_0271
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# supra-adjacency-matrix-generation

## Summary

Construct the supra-adjacency matrix (and normalized transition matrix) for a multiplex network by combining layer-specific igraph objects, as implemented by RWRtoolkit's parallelized create.multiplex / compute.adjacency.matrix / compute.transition.matrix pipeline.

## When to use

Use when the input is a set of layer edge-list files (an flist of {file_path, name, group} triples) that must be assembled into a multiplex object with a supra-adjacency matrix for downstream RWR analysis.

## When NOT to use

- The input is a single monoplex/homogeneous network with one edge type — no multiplex or supra-adjacency structure is needed
- Only an aggregated (summarized) network is required and layer-specific topological information can be discarded
- Layers are not expressible as edge-lists loadable as igraph objects per the flist triple format

## Inputs

- flist descriptor file with one {file_path, name, group} triple per layer edge-list
- Layer edge-list files (e.g. the nine A. thaliana layer files from dkainer/RWRtoolkit-data)
- delta parameter controlling inter-layer connection strength (e.g. 0.5)

## Outputs

- Multiplex object (nw.mpo) with Pool_of_Nodes of unique nodes
- Supra-adjacency matrix (nw.adj)
- Normalized transition matrix (nw.adjnorm)
- Saved RData file (e.g. Comprehensive_Network_AT_d0.5_v02.RData)

## How to apply

Write an flist descriptor file where each row is a triple {file_path, layer name, group} (delimiters ',', tab, '|', or ';' accepted) listing each layer edge-list. Run RWR_make_multiplex with a delta parameter (e.g. delta = 0.5 for the 9-layer comprehensive Arabidopsis multiplex with layers DU, GA, KS, PP, PX, PY, RE, RP, RX) to construct the multiplex list of igraph objects (create.multiplex), generate the supra-adjacency matrix (compute.adjacency.matrix, parallelized across cores), and normalize it into a transition matrix (compute.transition.matrix). Save the result as an RData object (e.g. Comprehensive_Network_AT_d0.5_v02.RData) containing nw.mpo, nw.adj, and nw.adjnorm. Rationale: RandomWalkRestartMH's unoptimized sequential construction scales exponentially with network size and layer count, whereas RWRtoolkit parallelizes supra-adjacency computation — a 50-layer network of 5000-node Erdős–Rényi layers that timed out after 6 hours under the original implementation completed in 17.69 seconds using 32 cores.

## Related tools

- **RWRtoolkit** (Provides RWR_make_multiplex and the parallelized create.multiplex / compute.adjacency.matrix / compute.transition.matrix pipeline used to build the supra-adjacency and transition matrices) — https://github.com/dkainer/RWRtoolkit
- **RandomWalkRestartMH** (Original sequential implementation of create.multiplex, compute.adjacency.matrix, and compute.transition.matrix that RWRtoolkit extends and parallelizes) — https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- **R** (Runtime environment (R >= 4.1.0) in which the multiplex construction functions are executed and RData outputs saved)
- **igraph** (Used to generate the random Erdős–Rényi layer networks in the benchmarking grid and as the graph representation of each layer)

## Examples

```
library(RWRtoolkit); mpo <- RWRtoolkit::RWR_make_multiplex(flist=flist, delta=0.5)
```

## Evaluation signals

- The saved RData object contains all three expected components: nw.mpo, nw.adj, and nw.adjnorm
- For the 9-layer comprehensive A. thaliana multiplex (delta = 0.5), the Pool_of_Nodes contains 26,605 unique nodes and the aggregated network 918,640 edges
- The transition matrix (nw.adjnorm) is a normalized version of the supra-adjacency matrix as produced by compute.transition.matrix
- On the (L,N) = (50,5000) benchmark of Erdős–Rényi layers, construction completes in ~17.69 seconds on 32 cores rather than timing out (original sequential implementation exceeded a 6-hour limit)
- flist rows parse correctly with any of the accepted delimiters and each layer maps to its {file_path, name, group} triple

## Limitations

- RandomWalkRestartMH multiplex construction was not optimized; computational load increases exponentially as network size and layer count grow
- No version number is stated for RWRtoolkit or RandomWalkRestartMH in the article, complicating exact reproducibility
- Installation of the R package may hit Bioconductor SSL certificate issues requiring manual CURL_CA_BUNDLE configuration, and clean installs can take ~20 minutes

## Evidence

- [methods] compute.adjacency.matrix, which generates the supra-adjacency matrix for the multiplex: "compute.adjacency.matrix, which generates the supra-adjacency matrix for the multiplex"
- [methods] compute.transition.matrix, which normalizes the adjacency matrix to create a transition matrix: "compute.transition.matrix, which normalizes the adjacency matrix to create a transition matrix"
- [discussion] parallelization reduces construction from hours to seconds: "With RWRtoolkit we parallelized this process to dramatically reduce the time to construct a large multiplex network from hours to seconds"
- [discussion] Sequential implementation not optimized and scales exponentially: "the construction of the multiplex was not optimized. As the size of networks and the total number of layers increases, the computational load for constructing the multiplex increases exponentially."
- [readme] RWR_make_multiplex builds the RData multiplex object from an flist: [evidence span withheld — anchor status: not_found]
