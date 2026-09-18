---
name: tau-layer-weighting-parameter-sweep
description: Use when testing or porting an RWR-on-multiplex implementation and you
  must verify that per-layer tau weights (e.g. tau swept from 2 to 0) actually propagate
  into per-node scores, especially for nodes unique to one layer.
when_to_use_negative:
- Your multiplex has no layer-unique nodes (all layers share the same node set), since
  per-layer tau effects on layer-unique scores cannot be observed
- You only need to run a production RWR analysis with fixed tau (e.g. the default
  "1.0,1.0") rather than validate tau sensitivity of an implementation
edam_operation: http://edamontology.org/operation_3927
tools:
- name: RWRtoolkit
  role: Provides the updated create.multiplex, compute.adjacency.matrix, compute.transition.matrix,
    and RandomWalkRestartMH functions used for the corrected tau-weighted random walk
  repo: https://github.com/dkainer/RWRtoolkit
- name: RandomWalkRestartMH
  role: Original package whose Random.Walk.Restart.Multiplex and 'RandomWalkRestartMHOriginal'
    implementation serve as the tau-insensitive baseline for comparison
  repo: https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- name: igraph
  role: Used to construct the toy layer graphs (two-node path and four-node clique)
    before multiplex creation
- name: R
  role: Runtime environment for the sweep script; the CLI also accepts --tau as a
    comma-separated vector (e.g. "1.0,1.0")
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:0885a9ff164fc994ca3f1b001d0656d0d82e5447070a05754ad82ebaf48dcadd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkainer/RWRtoolkit
  tools:
  - R
  - RWRtoolkit
  - RandomWalkRestartMH
  - igraph
  edam_operation: http://edamontology.org/operation_3927
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# tau-layer-weighting-parameter-sweep

## Summary

A verification procedure for the tau (layer-weighting) parameter in Random Walk with Restart on multiplex networks: sweep tau for one layer of a small toy multiplex and confirm that layer-unique node scores change as tau decreases, exposing whether the implementation correctly weights seeds per layer. It exists because the original RandomWalkRestartMH weighting made tau appear to have minimal effect.

## When to use

Use when testing or porting an RWR-on-multiplex implementation and you must verify that per-layer tau weights (e.g. tau swept from 2 to 0) actually propagate into per-node scores, especially for nodes unique to one layer.

## When NOT to use

- Your multiplex has no layer-unique nodes (all layers share the same node set), since per-layer tau effects on layer-unique scores cannot be observed
- You only need to run a production RWR analysis with fixed tau (e.g. the default "1.0,1.0") rather than validate tau sensitivity of an implementation

## Inputs

- Two-layer multiplex built from igraph graphs via create.multiplex (layer 1: two-node path A-B; layer 2: four-node clique A, B, C, D)
- Seed node list (nodes A and B)
- Vector of tau values for the second layer, swept over 2 down to 0
- Both implementations: original 'RandomWalkRestartMHOriginal' and updated RWRtoolkit RandomWalkRestartMH

## Outputs

- tau_weighting_verification.csv with columns (implementation, tau, node, score)
- Per-tau score vectors for each implementation, used to assert tau-dependent variation for nodes C and D

## How to apply

Build a minimal two-layer multiplex with igraph and create.multiplex: one layer as a two-node path (nodes A, B) and one layer as a four-node clique (nodes A, B, C, D), so layer-unique nodes exist. Compute the supra-adjacency with compute.adjacency.matrix and normalize it with compute.transition.matrix. Run 100 iterations of Random.Walk.Restart.Multiplex with nodes A and B as seeds, sweeping tau for the second layer from 2 (maximum importance) down to 0 (no importance), recording per-node scores at each tau; repeat the identical sweep with the original 'RandomWalkRestartMHOriginal' implementation. Compare per-tau score vectors: the updated RWRtoolkit implementation must show tau-dependent score changes for layer-unique nodes (C, D), whereas the original implementation produces identical scores at every tau because it weights each seed equally across all layers. Write the combined table (implementation, tau, node, score) to tau_weighting_verification.csv.

## Related tools

- **RWRtoolkit** (Provides the updated create.multiplex, compute.adjacency.matrix, compute.transition.matrix, and RandomWalkRestartMH functions used for the corrected tau-weighted random walk) — https://github.com/dkainer/RWRtoolkit
- **RandomWalkRestartMH** (Original package whose Random.Walk.Restart.Multiplex and 'RandomWalkRestartMHOriginal' implementation serve as the tau-insensitive baseline for comparison) — https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- **igraph** (Used to construct the toy layer graphs (two-node path and four-node clique) before multiplex creation)
- **R** (Runtime environment for the sweep script; the CLI also accepts --tau as a comma-separated vector (e.g. "1.0,1.0"))

## Examples

```
Rscript <LIBPATHS_DIRECTORY>/RWRtoolkit/scripts/run_loe.R --data string_interactions.Rdata --seed_geneset geneset1.txt --tau "1.0,1.0" -o ./outdir
```

## Evaluation signals

- Acceptance criterion: the updated implementation exhibits tau-dependent score variation for layer-unique nodes (C, D) across the tau sweep (tau in 0-2)
- The original 'RandomWalkRestartMHOriginal' implementation produces identical per-node scores at every tau value in the sweep (invariant output)
- The combined CSV contains one row per (implementation, tau, node) combination with a numeric score column, written to tau_weighting_verification.csv
- Both implementations were run under identical conditions: 100 iterations, seeds A and B, same supra-adjacency and normalized transition matrices
- Supra-adjacency and transition matrices were built via compute.adjacency.matrix and compute.transition.matrix before running the walk

## Limitations

- In the original RandomWalkRestartMH implementation, tau incorrectly weights each seed equally across all layers rather than differently per layer, so a tau sweep there cannot show layer-specific effects
- No version number was stated for RWRtoolkit or RandomWalkRestartMH in the article, so the exact corrected implementation version must be pinned by the practitioner
- The toy two-layer multiplex is a minimal diagnostic; results on it do not by themselves validate tau behavior on large biological multiplexes

## Evidence

- [discussion] Original implementation tau insensitivity: "the implementation in the RandomWalkRestartMH package was not correctly weighting the seeds within the multiplex network, leading Valdeolivas et al. [13] to conclude that tau had a minimal effect on"
- [discussion] Updated base functionality: "we have updated the base underlying functionality of RandomWalkRestartMH. While RandomWalkRestartMH provides core functionality for constructing multiplex networks and implementing random walk"
- [methods] 100 RWR iterations: "We ran 100 iterations of Random.Walk.Restart.Multiplex function from RandomWalkRestartMH"
- [readme] CLI tau parameter format: "--seed_geneset    <LIBPATHS_DIRECTORY>/RWRtoolkit/example_data/geneset1.txt \
    --tau             "1.0,1.0" \"
