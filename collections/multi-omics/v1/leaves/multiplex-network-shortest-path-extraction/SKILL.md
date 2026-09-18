---
name: multiplex-network-shortest-path-extraction
description: Use when you have a prebuilt multiplex network (RData) and two gene-set
  files, and you need the concrete shortest paths and path lengths connecting every
  source-target gene pair.
when_to_use_negative:
- You need ranked candidate genes around a seed set rather than explicit node-by-node
  paths — RWR_LOE or plain RWR scoring is the appropriate function
- You only have a single (monoplex) network and layer annotation is irrelevant — RandomWalkRestartMH
  monoplex functions or standard igraph shortest-path calls suffice
- The network has not yet been assembled into a multiplex RData object — build it
  first with RWR_make_multiplex from an flist of layer files
edam_operation: http://edamontology.org/operation_3927
tools:
- name: RWRtoolkit
  role: Provides RWR_ShortestPaths and RWR_make_multiplex used to load the multiplex
    and extract source-target shortest paths with layer and weight annotations
  repo: http://github.com/dkainer/RWRtoolkit
- name: R
  role: Runtime environment; RWRtoolkit is an R package (requires R >= 4.1.0) also
    usable via Rscript command-line scripts
- name: RandomWalkRestartMH
  role: Base package providing underlying functionality for constructing multiplex
    networks and implementing random walks, on which RWRtoolkit builds
- name: RWRtoolkit-data
  role: Supplies preassembled multiplex network RData objects such as Comprehensive_Network_AT_d0.5_v02.RData
    used as the path-search substrate
  repo: https://github.com/dkainer/RWRtoolkit-data
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:1c6c58d66670d30633b7be0834d599e50b36194bd48be055b4928f9ec8c58529
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkainer/RWRtoolkit
  tools:
  - R
  - RWR_ShortestPaths
  - RWRtoolkit
  - RWRtoolkit-data
  - RandomWalkRestartMH
  edam_operation: http://edamontology.org/operation_3927
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# multiplex-network-shortest-path-extraction

## Summary

Extracts all pairwise shortest paths between a source gene set and a target gene set on a multiplex network using RWRtoolkit's RWR_ShortestPaths, returning an edge list annotated with connecting nodes, multiplex layer(s), and edge weights (e.g. the comprehensive 9-layer A. thaliana multiplex, delta = 0.5).

## When to use

Use when you have a prebuilt multiplex network (RData) and two gene-set files, and you need the concrete shortest paths and path lengths connecting every source-target gene pair.

## When NOT to use

- You need ranked candidate genes around a seed set rather than explicit node-by-node paths — RWR_LOE or plain RWR scoring is the appropriate function
- You only have a single (monoplex) network and layer annotation is irrelevant — RandomWalkRestartMH monoplex functions or standard igraph shortest-path calls suffice
- The network has not yet been assembled into a multiplex RData object — build it first with RWR_make_multiplex from an flist of layer files

## Inputs

- Multiplex network RData object (e.g. comprehensive 9-layer A. thaliana multiplex, unweighted, delta = 0.5)
- Source gene set file in RWRtoolkit geneset format (e.g. AT1G13280, AT1G17420)
- Target gene set file in RWRtoolkit geneset format (e.g. AT1G48500, AT3G43440)

## Outputs

- shortest_paths_edge_list.tsv — edge list reporting, for each path edge, the connecting nodes, the multiplex layer(s) providing the edge, and the edge weight
- Per source-target pair path lengths (e.g. all 4 paths of length 3 in Table 5)

## How to apply

Load the preassembled multiplex RData object (e.g. Comprehensive_Network_AT_d0.5_v02.RData: 9 layers, unweighted edges, delta = 0.5) into RWRtoolkit via RWR_make_multiplex. Write the source genes (AT1G13280, AT1G17420) and target genes (AT1G48500, AT3G43440) as separate gene-set files in the RWRtoolkit geneset format. Run RWR_ShortestPaths with the multiplex and the source/target gene sets as parameters so it enumerates all shortest paths for every source-target pair. The result is an edge list in which each record reports the path edge's connecting nodes, the multiplex layer(s) supplying the edge, and the edge weight; validate that the output exists as shortest_paths_edge_list.tsv, covers all source-target pairs (here 2 x 2 = 4), and that each path has the expected number of edges with layer and weight annotations present.

## Related tools

- **RWRtoolkit** (Provides RWR_ShortestPaths and RWR_make_multiplex used to load the multiplex and extract source-target shortest paths with layer and weight annotations) — http://github.com/dkainer/RWRtoolkit
- **R** (Runtime environment; RWRtoolkit is an R package (requires R >= 4.1.0) also usable via Rscript command-line scripts)
- **RandomWalkRestartMH** (Base package providing underlying functionality for constructing multiplex networks and implementing random walks, on which RWRtoolkit builds)
- **RWRtoolkit-data** (Supplies preassembled multiplex network RData objects such as Comprehensive_Network_AT_d0.5_v02.RData used as the path-search substrate) — https://github.com/dkainer/RWRtoolkit-data

## Examples

```
Rscript <LIBPATHS_DIRECTORY>/RWRtoolkit/scripts/run_shortestpaths.R --data Comprehensive_Network_AT_d0.5_v02.Rdata --source_geneset sources.txt --target_geneset targets.txt -o ./outdir
```

## Evaluation signals

- Output file shortest_paths_edge_list.tsv exists and is parseable as a TSV edge list
- Edge list contains paths for all source-target pairs (2 sources x 2 targets = 4 paths in the exemplar)
- Every reported path edge carries multiplex layer annotation(s) and an edge weight
- Path lengths match expectations from the reference run (all 4 paths of length 3, Table 5)
- The geneset files parse in RWRtoolkit geneset format and all listed genes are present as nodes in the multiplex

## Limitations

- RWR_ShortestPaths depends on a preassembled multiplex RData object; layer and edge-weight reporting only make sense for unweighted multiplexes built with the specified delta (0.5 in the exemplar)
- Multiplex construction is computationally heavy if done from scratch; RWRtoolkit parallelizes it, but large networks and many layers can still be demanding (article notes unoptimized construction grows exponentially)
- No version number for RWRtoolkit was stated in the article, so exact function signatures/output columns may vary across releases

## Evidence

- [methods] RWR_ShortestPaths run against comprehensive multiplex: "We ran RWR_ShortestPaths with the source and target gene sets as parameters."
- [abstract] Topological distance functions in RWRtoolkit: "A range of functions can be used to find topological distances between biological entities, determine relationships within sets of interest, search for topological context around sets of interest,"
- [readme] Multiplex build prerequisite: [evidence span withheld — anchor status: not_found]
- [readme] Preassembled comprehensive Arabidopsis multiplex characteristics: "Contains the following network layers with unweighted edges and delta value = 0.5"
- [readme] RWRtoolkit extends RandomWalkRestartMH: "RWRtoolkit enables easy use of RandomWalk with Restart on multiplex networks. These functions are an extension to the"
