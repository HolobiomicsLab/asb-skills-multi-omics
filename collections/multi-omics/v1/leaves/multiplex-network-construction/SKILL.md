---
name: multiplex-network-construction
description: Use when you have multiple edge-list files representing distinct relationship
  types (e.g. coexpression, PPI, regulation layers) and need a single multiplex network
  object with supra-adjacency and transition matrices for RWR analysis.
when_to_use_negative:
- 'You only need the aggregated network: aggregation is a summarization that loses
  the unique topological information carried by each layer, so do not treat an aggregate
  as a substitute for the multiplex'
- Input is a single homogeneous network (monoplex) — use monoplex RWR rather than
  multiplex construction
- Network size and layer count are small enough that the original sequential RandomWalkRestartMH
  construction is fast; the parallelized path adds no benefit there
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: RWRtoolkit
  role: Provides RWR_make_multiplex plus create.multiplex, compute.adjacency.matrix
    (parallelized supra-adjacency computation), and compute.transition.matrix
  repo: github.com/dkainer/RWRtoolkit
- name: RandomWalkRestartMH
  role: Original package whose sequential create.multiplex / compute.adjacency.matrix
    / compute.transition.matrix chain serves as the timing baseline; its implementation
    weighted tau seeds equally across layers
  repo: https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- name: igraph
  role: Generates random Erdős–Rényi layer networks for benchmarking construction
    across the (L,N) grid
- name: RWRtoolkit-data
  role: Source of the nine Arabidopsis layer edge lists and the preassembled comprehensive
    multiplex RData
  repo: dkainer/RWRtoolkit-data
merged_aliases:
- multiplex-network-construction-from-edge-lists
merged_alias_records:
- alias: multiplex-network-construction-from-edge-lists
  slug: multiplex-network-construction-from-edge-lists
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:48e312b08568f8fadb24fb77a7073b422f5467aaddf794d4504f34484dfbf675
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkainer/RWRtoolkit
  tools:
  - R
  - RWRtoolkit
  - RWRtoolkit-data
  - RandomWalkRestartMH
  - igraph
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# multiplex-network-construction

## Summary

Constructs a multiplex network from multiple homogeneous edge-list layers using RWRtoolkit (create.multiplex, compute.adjacency.matrix, compute.transition.matrix), preserving layer-specific topology while producing the supra-adjacency and normalized transition matrices needed for random walks with restart. This replaces the unoptimized sequential RandomWalkRestartMH construction, whose computational load grows exponentially with layer count and network size.

## When to use

Use when you have multiple edge-list files representing distinct relationship types (e.g. coexpression, PPI, regulation layers) and need a single multiplex network object with supra-adjacency and transition matrices for RWR analysis.

## When NOT to use

- You only need the aggregated network: aggregation is a summarization that loses the unique topological information carried by each layer, so do not treat an aggregate as a substitute for the multiplex
- Input is a single homogeneous network (monoplex) — use monoplex RWR rather than multiplex construction
- Network size and layer count are small enough that the original sequential RandomWalkRestartMH construction is fast; the parallelized path adds no benefit there

## Inputs

- flist descriptor file with rows of {file_path, layer name, layer code/group} delimitable by comma, tab, pipe, or semicolon
- Per-layer edge-list files (unweighted edges), e.g. the nine Arabidopsis layer files from dkainer/RWRtoolkit-data
- delta parameter (0.5 used in the comprehensive Arabidopsis multiplex)

## Outputs

- Multiplex mpo object (nw.mpo) with Pool_of_Nodes and per-layer igraph objects
- Supra-adjacency matrix (nw.adj) and normalized transition matrix (nw.adjnorm)
- Saved RData object, e.g. Comprehensive_Network_AT_d0.5_v02.RData
- Optional netstats file with multiplex metadata and pairwise between-layer Jaccard similarity

## How to apply

Write an flist descriptor file in which each row is a triple {file_path, name, group} defining one layer (e.g. the nine A. thaliana layers DU, GA, KS, PP, PX, PY, RE, RP, RX). Run RWR_make_multiplex (or the underlying create.multiplex → compute.adjacency.matrix → compute.transition.matrix chain, which constructs the multiplex list of igraph objects, generates the supra-adjacency matrix, and normalizes it into a transition matrix) with delta = 0.5, the value used for the Arabidopsis comprehensive network. For large networks, rely on RWRtoolkit's parallelized supra-adjacency computation (32 cores): a 50-layer, 5000-node-per-layer network that timed out at 6 hours under the original sequential implementation completed construction in 17.69 seconds. Save the result as an RData object (nw.mpo, nw.adj, nw.adjnorm) and optionally profile layer composition with RWR_netstats, which reports pairwise between-layer Jaccard similarity. Benchmark correctness by timing construction on a grid of (L,N) combinations, generating L random Erdős–Rényi networks of size N in igraph for each.

## Related tools

- **RWRtoolkit** (Provides RWR_make_multiplex plus create.multiplex, compute.adjacency.matrix (parallelized supra-adjacency computation), and compute.transition.matrix) — github.com/dkainer/RWRtoolkit
- **RandomWalkRestartMH** (Original package whose sequential create.multiplex / compute.adjacency.matrix / compute.transition.matrix chain serves as the timing baseline; its implementation weighted tau seeds equally across layers) — https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- **igraph** (Generates random Erdős–Rényi layer networks for benchmarking construction across the (L,N) grid)
- **RWRtoolkit-data** (Source of the nine Arabidopsis layer edge lists and the preassembled comprehensive multiplex RData) — dkainer/RWRtoolkit-data

## Examples

```
nw <- RWRtoolkit::RWR_make_multiplex(input_flist_path="flist.tsv", delta=0.5)
```

## Evaluation signals

- Loaded RData object's Pool_of_Nodes contains 26,605 unique nodes and the aggregated network contains 918,640 edges for the nine-layer comprehensive Arabidopsis multiplex at delta = 0.5
- Construction timing on a 50-layer, 5000-node-per-layer benchmark completes in approximately 17.69 seconds with 32 cores, versus the original implementation exceeding a 6-hour timeout
- Supra-adjacency matrix dimension equals the expected sum of layer node blocks with inter-layer identity links at delta; compute.transition.matrix output is row-normalized (a valid transition matrix)
- netstats output file contains basic statistics and pairwise between-layer Jaccard similarity for all layer pairs
- Each flist row parses as a triple {file_path, name, group} with all edge-list files resolvable at the given paths

## Limitations

- The original RandomWalkRestartMH multiplex construction was not optimized and its computational load increases exponentially as network size and layer count grow
- No version number is stated for RWRtoolkit or RandomWalkRestartMH, complicating exact reproduction; the comprehensive network is versioned as d0.5_v02
- Installation can fail on SSL certificate issues accessing Bioconductor (fixable via cacert.pem and CURL_CA_BUNDLE) and on missing harfbuzz/fribidi/libgit2 system libraries for textshaping and gert
- Aggregated-network views lose layer-specific topology, so summary statistics alone cannot validate per-layer structure

## Evidence

- [methods] create.multiplex builds layer list: "create.multiplex, which constructs the multiplex network list of igraph objects"
- [methods] supra-adjacency matrix: "compute.adjacency.matrix, which generates the supra-adjacency matrix for the multiplex"
- [methods] transition matrix normalization: "compute.transition.matrix, which normalizes the adjacency matrix to create a transition matrix"
- [discussion] parallelized construction speedup: "With RWRtoolkit we parallelized this process to dramatically reduce the time to construct a large multiplex network from hours to seconds"
- [readme] flist triple format: [evidence span withheld — anchor status: not_found]
- [methods] Erdős–Rényi benchmark layers: "For each combination (L,N), we generated L random networks of size N using the Erdős–Rényi model in the igraph package"
- [discussion] exponential load limitation: "the construction of the multiplex was not optimized. As the size of networks and the total number of layers increases, the computational load for constructing the multiplex increases exponentially."
