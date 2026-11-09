---
name: rwr-seed-score-computation
description: Use when seed genes/nodes must be scored against a multiplex network
  and per-layer tau weighting should influence rankings — e.g. to verify that layer
  importance parameters actually change output scores.
when_to_use_negative:
- When the analysis target is a single monoplex network rather than a multiplex of
  layers sharing the same node set
- When tau layer-importance sensitivity is not of interest and equal seed weighting
  across layers is acceptable
- When the multiplex needs downstream statistical validation (RWR_CV) or network statistics
  (RWR_netstats) rather than direct seed scoring
edam_operation: http://edamontology.org/operation_3927
edam_topics: []
tools:
- name: RWRtoolkit
  role: Provides create.multiplex, compute.adjacency.matrix, compute.transition.matrix
    and the corrected Random.Walk.Restart.Multiplex with per-layer tau seed weighting
  repo: https://github.com/dkainer/RWRtoolkit
- name: RandomWalkRestartMH
  role: Original base package whose Random.Walk.Restart.Multiplex serves as the comparison
    implementation; its original seed weighting incorrectly treated tau
  repo: https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- name: R
  role: Environment in which the igraph layers, transition matrices, and 100-iteration
    RWR runs are executed
- name: igraph
  role: Constructs the toy layer graphs (two-node path A-B; four-node clique A,B,C,D)
    used to build the multiplex
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:282f48f7147f50f825565799af87ff1ada7f7fb67ec382f642481eba2601dd8f
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
yes_rate: 0.9166666666666666
n_claims: 12
---

# rwr-seed-score-computation

## Summary

Computes random-walk-with-restart node scores from seed genes on a multiplex network using RWRtoolkit's Random.Walk.Restart.Multiplex over the normalized supra-adjacency transition matrix, with per-layer tau weights that correctly bias seeds within layers (a fix over the original RandomWalkRestartMH seed weighting).

## When to use

Use when seed genes/nodes must be scored against a multiplex network and per-layer tau weighting should influence rankings — e.g. to verify that layer importance parameters actually change output scores.

## When NOT to use

- When the analysis target is a single monoplex network rather than a multiplex of layers sharing the same node set
- When tau layer-importance sensitivity is not of interest and equal seed weighting across layers is acceptable
- When the multiplex needs downstream statistical validation (RWR_CV) or network statistics (RWR_netstats) rather than direct seed scoring

## Inputs

- Multiplex network object built from igraph layer graphs via create.multiplex (or an flist file of {file_path, name, group} triples via RWR_make_multiplex)
- Supra-adjacency matrix and normalized transition matrix (compute.adjacency.matrix / compute.transition.matrix)
- Seed node set (e.g. nodes A and B in a toy two-layer multiplex)
- Tau vector, one weight per layer (e.g. values swept from 2 down to 0)

## Outputs

- Node score vector per seed set (scores for all nodes including layer-unique nodes such as C, D)
- Per-tau score table combining implementation, tau, node, and score (e.g. tau_weighting_verification.csv)

## How to apply

Build the multiplex from igraph layer objects with create.multiplex, then generate the supra-adjacency matrix with compute.adjacency.matrix and normalize it into a transition matrix with compute.transition.matrix. Run 100 iterations of Random.Walk.Restart.Multiplex supplying seed nodes and a tau vector, where each tau entry scales the seed weights for the corresponding layer (tau=2 maximum importance, tau=0 no importance). Because the original RandomWalkRestartMH implementation weighted each seed equally across all layers rather than per layer, use the updated RWRtoolkit implementation so that layer-unique nodes respond to tau changes. Validate by sweeping tau and confirming node scores vary with tau for nodes unique to a layer; identical score vectors across the sweep indicate the incorrect original weighting is in use.

## Related tools

- **RWRtoolkit** (Provides create.multiplex, compute.adjacency.matrix, compute.transition.matrix and the corrected Random.Walk.Restart.Multiplex with per-layer tau seed weighting) — https://github.com/dkainer/RWRtoolkit
- **RandomWalkRestartMH** (Original base package whose Random.Walk.Restart.Multiplex serves as the comparison implementation; its original seed weighting incorrectly treated tau) — https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- **R** (Environment in which the igraph layers, transition matrices, and 100-iteration RWR runs are executed)
- **igraph** (Constructs the toy layer graphs (two-node path A-B; four-node clique A,B,C,D) used to build the multiplex)

## Examples

```
Rscript <LIBPATHS_DIRECTORY>/RWRtoolkit/scripts/run_loe.R --data string_interactions.Rdata --seed_geneset geneset1.txt --tau "1.0,1.0" -o ./outdir
```

## Evaluation signals

- Updated implementation: node scores for layer-unique nodes (C, D) change as tau decreases across the sweep tau=2 to tau=0
- Original RandomWalkRestartMH implementation: per-tau score vectors are identical regardless of tau, confirming the known incorrect seed weighting
- Transition matrix is a proper normalization of the supra-adjacency matrix produced by compute.adjacency.matrix
- Combined CSV (implementation, tau, node, score) written with 100-iteration runs for both implementations over the same tau grid
- Seeds A and B receive nonzero scores consistent with restart probability on the transition matrix

## Limitations

- The original RandomWalkRestartMH implementation was not correctly weighting the seeds within the multiplex network, leading Valdeolivas et al. to conclude tau had minimal effect on output rankings
- No version number is stated for RWRtoolkit or RandomWalkRestartMH, so the corrected behavior must be confirmed empirically via the tau sweep
- RandomWalkRestartMH multiplex construction was not optimized; computational load increases exponentially with network size and layer count (RWRtoolkit parallelizes this)

## Evidence

- [discussion] In the RandomWalkRestartMH implementation, tau incorrectly weights each seed equally across all layers rather than differently per layer, leading Valdeolivas et al. to conclude tau had minimal effect on output rankings: "the implementation in the RandomWalkRestartMH package was not correctly weighting the seeds within the multiplex network, leading Valdeolivas et al. [13] to conclude that tau had a minimal effect on"
- [methods] Filter steps: create.multiplex, compute.adjacency.matrix, compute.transition.matrix: "compute.transition.matrix, which normalizes the adjacency matrix to create a transition matrix"
- [methods] RWR iterations count: "We ran 100 iterations of Random.Walk.Restart.Multiplex function from RandomWalkRestartMH"
- [readme] Tau parameter is exposed on the command line as a comma-separated per-layer vector: [evidence span withheld — anchor status: not_found]
- [readme] RWRtoolkit is installed as an extension to RandomWalkRestartMH: "These functions are an extension to the
[RandomWalkRestartMH](https://github.com/alberto-valdeolivas/RandomWalkRestartMH)
R package."
