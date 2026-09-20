---
name: igraph-toy-network-generation
description: Use when you need a small, fully controlled multiplex network to isolate
  parameter effects (e.g., tau per-layer seed weighting) in RWR, or to reproduce regression
  tests against an updated RWRtoolkit implementation.
when_to_use_negative:
- When analyzing real multi-omic biological networks — build the multiplex from an
  flist of edge files via RWR_make_multiplex instead of hand-built igraph toys
- When the goal is gene-set scoring or cross-validation (RWR_CV, RWR_LOE) rather than
  diagnosing parameter sensitivity or implementation correctness
- When only a single monoplex network is involved — use the monoplex RWR functions
  rather than multiplex construction
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: RWRtoolkit
  role: Provides create.multiplex to assemble igraph layers into the multiplex, and
    compute.adjacency.matrix / compute.transition.matrix to derive the supra-adjacency
    and transition matrices for the toy network
  repo: http://github.com/dkainer/RWRtoolkit
- name: RandomWalkRestartMH
  role: Original implementation whose get.seed.scoresMultiplex seed-weighting behavior
    is tested against the updated RWRtoolkit Random.Walk.Restart.Multiplex on the
    toy multiplex
  repo: https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- name: igraph
  role: Used to construct the tiny graph objects (two-node path, four-node clique)
    that populate each multiplex layer
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:72abe29b85f924d4574cfcab45afe093ac03533523266120efc3f9143f5523da
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - RWRtoolkit
  - RandomWalkRestartMH
  - igraph
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# igraph-toy-network-generation

## Summary

Constructs small hand-built igraph networks (e.g., a two-node path layer and a four-node clique layer) and assembles them into a multiplex with create.multiplex so that RWR parameter behavior, such as tau seed weighting, can be tested where ground truth for every node is known.

## When to use

Use when you need a small, fully controlled multiplex network to isolate parameter effects (e.g., tau per-layer seed weighting) in RWR, or to reproduce regression tests against an updated RWRtoolkit implementation.

## When NOT to use

- When analyzing real multi-omic biological networks — build the multiplex from an flist of edge files via RWR_make_multiplex instead of hand-built igraph toys
- When the goal is gene-set scoring or cross-validation (RWR_CV, RWR_LOE) rather than diagnosing parameter sensitivity or implementation correctness
- When only a single monoplex network is involved — use the monoplex RWR functions rather than multiplex construction

## Inputs

- Two or more small igraph graph objects with explicit node labels (e.g., path A–B; clique A, B, C, D)
- Seed node names shared across layers (e.g., A, B)
- Tau sweep values (e.g., 2 down to 0 for one layer)

## Outputs

- Multiplex network object (list of igraph layers) from create.multiplex
- Supra-adjacency matrix and normalized transition matrix
- Per-tau RWR node score vectors (100-iteration Random.Walk.Restart.Multiplex runs)
- Combined score table (implementation, tau, node, score), e.g., tau_weighting_verification.csv

## How to apply

Build tiny igraph objects where every node's role is known: one layer as a two-node path (nodes A, B) and one layer as a four-node clique (nodes A, B, C, D), sharing seed nodes across layers. Pass these igraph objects to create.multiplex to construct the multiplex list, then compute the supra-adjacency matrix with compute.adjacency.matrix and normalize it to a transition matrix with compute.transition.matrix. Run Random.Walk.Restart.Multiplex (e.g., 100 iterations) with seeds A and B while sweeping tau for the second layer over values from 2 (maximum importance) down to 0 (no importance). The small size makes the expected behavior auditable: node scores for layer-unique nodes (C, D) must change as tau decreases under the corrected RWRtoolkit weighting, whereas the original RandomWalkRestartMH implementation weights seeds equally across layers and produces tau-invariant scores. Record the (implementation, tau, node, score) table, e.g., to tau_weighting_verification.csv, and compare per-tau score vectors across implementations.

## Related tools

- **RWRtoolkit** (Provides create.multiplex to assemble igraph layers into the multiplex, and compute.adjacency.matrix / compute.transition.matrix to derive the supra-adjacency and transition matrices for the toy network) — http://github.com/dkainer/RWRtoolkit
- **RandomWalkRestartMH** (Original implementation whose get.seed.scoresMultiplex seed-weighting behavior is tested against the updated RWRtoolkit Random.Walk.Restart.Multiplex on the toy multiplex) — https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- **igraph** (Used to construct the tiny graph objects (two-node path, four-node clique) that populate each multiplex layer)

## Examples

```
library(RWRtoolkit); mp <- create.multiplex(list(Layer1 = igraph::make_graph(~ A -- B), Layer2 = igraph::make_full_graph(4)))
```

## Evaluation signals

- Updated RWRtoolkit implementation shows tau-dependent score variation for layer-unique nodes (C, D) across the tau sweep (tau 0–2)
- Original RandomWalkRestartMH implementation produces identical score vectors for all tau values on the same toy multiplex, exposing its equal-across-layer seed weighting
- Seed nodes A and B receive correctly weighted seed scores per layer in the updated implementation (get.seed.scoresMultiplex fix)
- Supra-adjacency and transition matrices computed without error for the two-layer toy multiplex (path layer plus clique layer)
- Output CSV contains rows keyed by (implementation, tau, node, score) covering every node and every tau in the sweep

## Limitations

- Toy networks verify parameter sensitivity and seed weighting but say nothing about runtime on large networks, where multiplex construction load increases exponentially with network size and layer count in the original package
- No version number is stated for RWRtoolkit or RandomWalkRestartMH in the article, so reproducing the exact original-vs-updated comparison requires pinning the repository state yourself
- Findings are specific to homogeneous multiplex networks; heterogeneous or full multiplex-heterogeneous networks are not covered by this toy setup

## Evidence

- [methods] create.multiplex, which constructs the multiplex network list of igraph objects: "create.multiplex, which constructs the multiplex network list of igraph objects"
- [methods] For each combination (L,N), we generated L random networks of size N using the Erdős–Rényi model in the igraph package: "we generated L random networks of size N using the Erdős–Rényi model in the igraph package"
- [discussion] In the RandomWalkRestartMH implementation, tau incorrectly weights each seed equally across all layers rather than differently per layer: "the implementation in the RandomWalkRestartMH package was not correctly weighting the seeds within the multiplex network"
- [readme] RWRtoolkit extends RandomWalkRestartMH for RWR on multiplex networks: "RWRtoolkit enables easy use of RandomWalk with Restart on multiplex networks. These functions are an extension to the"
