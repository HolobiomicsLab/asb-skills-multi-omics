---
name: implementation-behavior-regression-comparison
description: Use when an algorithm library has been updated/forked (e.g., RWRtoolkit
  vs original RandomWalkRestartMH) and you must confirm that a parameter now behaves
  as documented rather than being silently ignored.
when_to_use_negative:
- When the goal is biological discovery on a real large multiplex — this skill uses
  deliberately tiny toy graphs purely to expose implementation behavior
- When the algorithm has no tunable seed-weighting parameter (tau-like) whose effect
  can be differentially probed across layers
- When both implementations are already known to produce identical outputs and no
  behavior change is suspected
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: RWRtoolkit
  role: Updated implementation of RWR on multiplex networks; provides create.multiplex,
    compute.adjacency.matrix, compute.transition.matrix, and Random.Walk.Restart.Multiplex
    used to run the tau sweep
  repo: github.com/dkainer/RWRtoolkit
- name: RandomWalkRestartMH
  role: Original implementation (RandomWalkRestartMHOriginal baseline) whose seed-weighting
    behavior is regressed against the updated version
  repo: github.com/alberto-valdeolivas/RandomWalkRestartMH
- name: R
  role: Execution environment for building the toy multiplex and running both implementations
- name: igraph
  role: Used to construct the toy graph layers (two-node path and four-node clique)
    before multiplex creation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:9d0158bdd369facaef92e34d66e44f55f5f5dc98fc2fcd4ab60b7f7e35b0d388
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - RWRtoolkit
  - RandomWalkRestartMH
  - igraph
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# implementation-behavior-regression-comparison

## Summary

A verification skill that detects silent behavioral regressions or corrections between two implementations of the same algorithm by sweeping a key parameter (here tau) on a small toy multiplex and diffing per-node scores across implementations.

## When to use

Use when an algorithm library has been updated/forked (e.g., RWRtoolkit vs original RandomWalkRestartMH) and you must confirm that a parameter now behaves as documented rather than being silently ignored.

## When NOT to use

- When the goal is biological discovery on a real large multiplex — this skill uses deliberately tiny toy graphs purely to expose implementation behavior
- When the algorithm has no tunable seed-weighting parameter (tau-like) whose effect can be differentially probed across layers
- When both implementations are already known to produce identical outputs and no behavior change is suspected

## Inputs

- Toy multiplex network layers as igraph objects (e.g., two-node path graph and four-node clique)
- Seed node set (e.g., nodes A and B)
- Tau sweep values for the second layer (2 down to 0)
- Original implementation source ('RandomWalkRestartMHOriginal') and updated RWRtoolkit implementation

## Outputs

- Per-tau node score vectors for each implementation
- Combined score table written to tau_weighting_verification.csv (columns: implementation, tau, node, score)

## How to apply

Build a minimal toy multiplex with igraph and RWRtoolkit's create.multiplex — one layer as a two-node path (nodes A, B) and one layer as a four-node clique (nodes A, B, C, D) so that layer-unique nodes exist. Compute the supra-adjacency and normalized transition matrices with compute.adjacency.matrix and compute.transition.matrix. Run 100 iterations of Random.Walk.Restart.Multiplex with nodes A and B as seeds under both the original ('RandomWalkRestartMHOriginal') and updated implementations, sweeping tau for the second layer from 2 (maximum importance) down to 0 (no importance). The rationale: the RandomWalkRestartMH implementation weighted each seed equally across all layers rather than per layer, which led Valdeolivas et al. to conclude tau had minimal effect; a correct implementation must therefore show tau-dependent score variation for nodes unique to the tau-weighted layer (C, D), while a broken one yields identical score vectors across all tau values. Write the combined score table (implementation, tau, node, score) to tau_weighting_verification.csv for side-by-side comparison.

## Related tools

- **RWRtoolkit** (Updated implementation of RWR on multiplex networks; provides create.multiplex, compute.adjacency.matrix, compute.transition.matrix, and Random.Walk.Restart.Multiplex used to run the tau sweep) — github.com/dkainer/RWRtoolkit
- **RandomWalkRestartMH** (Original implementation (RandomWalkRestartMHOriginal baseline) whose seed-weighting behavior is regressed against the updated version) — github.com/alberto-valdeolivas/RandomWalkRestartMH
- **R** (Execution environment for building the toy multiplex and running both implementations)
- **igraph** (Used to construct the toy graph layers (two-node path and four-node clique) before multiplex creation)

## Examples

```
Rscript <LIBPATHS_DIRECTORY>/RWRtoolkit/scripts/run_loe.R --data <LIBPATHS_DIRECTORY>/RWRtoolkit/example_data/string_interactions.Rdata --seed_geneset <LIBPATHS_DIRECTORY>/RWRtoolkit/example_data/geneset1.txt --tau "1.0,1.0" -o ./outdir
```

## Evaluation signals

- Updated implementation's scores for layer-unique nodes (C, D) change monotonically as tau decreases from 2 to 0
- Original RandomWalkRestartMHOriginal implementation produces identical score vectors for all tau values in the sweep (demonstrating tau had no effect)
- Supra-adjacency and transition matrices are computed for the multiplex before the walk (compute.adjacency.matrix then compute.transition.matrix)
- Combined output table tau_weighting_verification.csv contains one row per (implementation, tau, node) with a numeric score
- Both implementations run for exactly 100 iterations with identical seeds (A, B) so that only the tau weighting differs

## Limitations

- The original implementation's get.seed.scoresMultiplex weighted each seed equally across all layers rather than differently per layer, so results from it cannot distinguish tau values
- No version number is stated for RWRtoolkit or RandomWalkRestartMH in the article, so exact version pinning of the baseline is not documented
- This is a toy-network regression check (two-node path + four-node clique); its pass/fail signal is about implementation behavior, not biological validity

## Evidence

- [discussion] the implementation in the RandomWalkRestartMH package was not correctly weighting the seeds within the multiplex network, leading Valdeolivas et al. [13] to conclude that tau had a minimal effect on: "the implementation in the RandomWalkRestartMH package was not correctly weighting the seeds within the multiplex network, leading Valdeolivas et al. [13] to conclude that tau had a minimal effect on"
- [methods] We ran 100 iterations of Random.Walk.Restart.Multiplex function from RandomWalkRestartMH: "We ran 100 iterations of Random.Walk.Restart.Multiplex function from RandomWalkRestartMH"
- [methods] create.multiplex, which constructs the multiplex network list of igraph objects: "create.multiplex, which constructs the multiplex network list of igraph objects"
- [readme] RWRtoolkit enables easy use of RandomWalk with Restart on multiplex networks. These functions are an extension to the RandomWalkRestartMH: "RWRtoolkit enables easy use of RandomWalk with Restart on multiplex networks. These functions are an extension to the RandomWalkRestartMH"
