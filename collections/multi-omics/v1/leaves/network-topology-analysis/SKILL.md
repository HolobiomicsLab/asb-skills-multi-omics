---
name: network-topology-analysis
description: Use when you have a preassembled multiplex network (e.g. the 9-layer
  comprehensive Arabidopsis thaliana multiplex, unweighted, delta = 0.5) and two gene
  sets, and you need the actual shortest-path edge lists and path lengths between
  every source-target pair.
when_to_use_negative:
- You need probabilistic relatedness/ranking of candidate genes around a seed set
  — use RWR_LOE or RWR_CV instead, since shortest paths capture only topological distance,
  not RWR-based connectivity scores
- 'Your input is a single aggregated network: the aggregated network is a summarization
  of the input layers and has lost the unique topological information carried by each
  layer, so per-edge layer attribution is impossible'
- You need weighted path optimization — the preassembled comprehensive multiplex is
  constructed with unweighted edges (delta = 0.5), so reported 'weights' do not encode
  layer edge strengths
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0091
tools:
- name: RWRtoolkit
  role: Provides RWR_ShortestPaths to extract shortest paths between source and target
    gene sets on the multiplex, plus RWR_make_multiplex/create.multiplex to build
    the network
  repo: github.com/dkainer/RWRtoolkit
- name: R
  role: Environment in which RWRtoolkit functions (R package or command-line scripts
    via Rscript) are executed
- name: RandomWalkRestartMH
  role: Base package extended by RWRtoolkit; supplies core multiplex construction
    and random-walk functionality underlying the workflow
  repo: alberto-valdeolivas/RandomWalkRestartMH
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:359fb79a8ae50b1a80142b5c01e6ee92905b3539a0df01bef1ecdbbc8efd3bbe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkainer/RWRtoolkit
  tools:
  - R
  - RWR_ShortestPaths
  - RWRtoolkit
  - RandomWalkRestartMH
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# network-topology-analysis

## Summary

Extracts pairwise shortest paths between source and target gene sets on a multiplex network using RWRtoolkit's RWR_ShortestPaths, returning for each path edge the connecting nodes, contributing multiplex layer(s), and edge weights. It grounds gene-set connectivity in concrete, layer-annotated topological routes rather than RWR ranking scores alone.

## When to use

Use when you have a preassembled multiplex network (e.g. the 9-layer comprehensive Arabidopsis thaliana multiplex, unweighted, delta = 0.5) and two gene sets, and you need the actual shortest-path edge lists and path lengths between every source-target pair.

## When NOT to use

- You need probabilistic relatedness/ranking of candidate genes around a seed set — use RWR_LOE or RWR_CV instead, since shortest paths capture only topological distance, not RWR-based connectivity scores
- Your input is a single aggregated network: the aggregated network is a summarization of the input layers and has lost the unique topological information carried by each layer, so per-edge layer attribution is impossible
- You need weighted path optimization — the preassembled comprehensive multiplex is constructed with unweighted edges (delta = 0.5), so reported 'weights' do not encode layer edge strengths

## Inputs

- Preassembled multiplex network object (e.g. Comprehensive_Network_AT_d0.5_v02.RData, 9 layers, unweighted edges, delta = 0.5)
- flist file listing network layers as {file_path, name, group} triples (if building the multiplex from scratch)
- Source gene set file in RWRtoolkit geneset format (e.g. AT1G13280, AT1G17420)
- Target gene set file in RWRtoolkit geneset format (e.g. AT1G48500, AT3G43440)

## Outputs

- shortest_paths_edge_list.tsv — edge list records with connecting nodes, multiplex layer(s) per edge, and edge weight for each path
- Per source-target pair path lengths (e.g. all 4 paths of length 3 in the jasmonate example, Table 5)

## How to apply

First build the multiplex RData object with RWRtoolkit's RWR_make_multiplex from an flist of network layers (create.multiplex constructs the list of igraph objects underlying it). Write the source genes (e.g. AT1G13280, AT1G17420) and target genes (e.g. AT1G48500, AT3G43440) into separate gene-set files in the RWRtoolkit geneset format. Run RWR_ShortestPaths with the multiplex, source and target gene sets as parameters to extract all possible shortest paths between every source-target pair. Collect the returned edge list records, which report for each path edge the connecting nodes, the multiplex layer(s) providing the edge, and the edge weight. Because the comprehensive A. thaliana multiplex is constructed unweighted, path lengths are edge counts (hops), not weighted distances. Validate that edge records exist for all source-target pairs and that each reported path has consistent length with layer and weight annotations present.

## Related tools

- **RWRtoolkit** (Provides RWR_ShortestPaths to extract shortest paths between source and target gene sets on the multiplex, plus RWR_make_multiplex/create.multiplex to build the network) — github.com/dkainer/RWRtoolkit
- **R** (Environment in which RWRtoolkit functions (R package or command-line scripts via Rscript) are executed)
- **RandomWalkRestartMH** (Base package extended by RWRtoolkit; supplies core multiplex construction and random-walk functionality underlying the workflow) — alberto-valdeolivas/RandomWalkRestartMH

## Examples

```
library(RWRtoolkit); RWR_ShortestPaths(multiplex = mp, source_geneset = "source_geneset.txt", target_geneset = "target_geneset.txt")
```

## Evaluation signals

- Output edge list file exists (shortest_paths_edge_list.tsv) and contains edges for all source-target pairs (2 sources x 2 targets = 4 paths in the example)
- Each reported path has a length of 3 edges in the jasmonate example, consistent with unweighted hop counts on the comprehensive multiplex
- Every path edge record carries layer and weight annotations (which multiplex layer(s) supply each edge)
- Multiplex object was built successfully first (RWR_make_multiplex over the flist) and all four gene IDs are present as nodes in the network layers
- Output conforms to the RWRtoolkit geneset/edge-list schemas (delimiters , \t | ; accepted in flist/geneset inputs)

## Limitations

- The preassembled comprehensive Arabidopsis multiplex (Comprehensive_Network_AT_d0.5_v02.RData) uses unweighted edges with delta = 0.5, so path lengths are hop counts and edge weights do not encode interaction strength
- RWRtoolkit requires R >= 4.1.0; installation via devtools may hit Bioconductor SSL certificate issues or missing system libraries (harfbuzz, fribidi, libgit2)
- No version number is stated for RWRtoolkit or RandomWalkRestartMH in the article, complicating exact reproducibility

## Evidence

- [methods] We ran RWR_ShortestPaths with the source and target gene sets as parameters.: "We ran RWR_ShortestPaths with the source and target gene sets as parameters."
- [abstract] Topological distances function range: "A range of functions can be used to find topological distances between biological entities, determine relationships within sets of interest, search for topological context around sets of interest"
- [intro] Aggregated network loses layer information: "The aggregated network is a summarization of the input layers, and as such has lost the unique topological information carried by each layer."
- [readme] Multiplex construction step: [evidence span withheld — anchor status: not_found]
- [readme] Comprehensive multiplex specification: "Contains the following network layers with unweighted edges and delta value = 0.5"
