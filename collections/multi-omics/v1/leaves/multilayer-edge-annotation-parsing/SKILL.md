---
name: multilayer-edge-annotation-parsing
description: Use when you have run RWR_ShortestPaths against an RWRtoolkit multiplex
  (e.g. the 9-layer comprehensive A. thaliana network, delta = 0.5) and need to parse,
  validate, or summarize the returned edge list records for all source-target pairs.
when_to_use_negative:
- You only need node-level similarity rankings from a single seed set — RWR_LOE or
  RWR itself returns ranked genes without path/layer annotation
- Your network is a single monoplex rather than a multiplex, so per-layer edge provenance
  annotations are meaningless
- The gene sets lack a natural source/target distinction — use RWR_CV or RWR_LOE for
  within-set connectivity instead
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: RWRtoolkit
  role: Provides RWR_ShortestPaths and the multiplex infrastructure that produces
    the annotated edge-list output parsed by this skill
  repo: https://github.com/dkainer/RWRtoolkit
- name: RWR_ShortestPaths
  role: RWRtoolkit function run with source and target gene sets as parameters to
    extract all possible shortest paths between every source-target pair, returning
    edges with layer and weight annotations
  repo: https://github.com/dkainer/RWRtoolkit
- name: RandomWalkRestartMH
  role: Base package whose multiplex construction and random-walk functionality RWRtoolkit
    extends
  repo: https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- name: R
  role: Environment in which RWRtoolkit functions are called and the TSV edge list
    is parsed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:8bfee5fd4bdee7e0be6fec8089b7206c9962781d8475c1aae1c83bf4253388f4
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
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# multilayer-edge-annotation-parsing

## Summary

Parse the edge-list output of RWR_ShortestPaths (shortest_paths_edge_list.tsv) from RWRtoolkit multiplex networks, extracting per-edge connecting nodes, the multiplex layer(s) supplying each edge, and edge weights, to reconstruct and validate shortest paths between source and target gene sets.

## When to use

Use when you have run RWR_ShortestPaths against an RWRtoolkit multiplex (e.g. the 9-layer comprehensive A. thaliana network, delta = 0.5) and need to parse, validate, or summarize the returned edge list records for all source-target pairs.

## When NOT to use

- You only need node-level similarity rankings from a single seed set — RWR_LOE or RWR itself returns ranked genes without path/layer annotation
- Your network is a single monoplex rather than a multiplex, so per-layer edge provenance annotations are meaningless
- The gene sets lack a natural source/target distinction — use RWR_CV or RWR_LOE for within-set connectivity instead

## Inputs

- Preassembled comprehensive Arabidopsis thaliana multiplex network object (9 layers, unweighted edges, delta = 0.5, e.g. Comprehensive_Network_AT_d0.5_v02.RData)
- Source gene set file in RWRtoolkit geneset format (e.g. AT1G13280, AT1G17420)
- Target gene set file in RWRtoolkit geneset format (e.g. AT1G48500, AT3G43440)
- Multiplex flist file whose rows are triples {file_path, name, group}

## Outputs

- shortest_paths_edge_list.tsv — edge list with, per path edge, connecting nodes, multiplex layer(s) providing the edge, and edge weight
- Per source-target pair path lengths (e.g. all 4 paths of length 3, as in Table 5)

## How to apply

Load the preassembled comprehensive A. thaliana multiplex (9 unweighted layers, delta = 0.5) and run RWRtoolkit's RWR_ShortestPaths with explicit source and target gene-set files (e.g. sources AT1G13280/AT1G17420, targets AT1G48500/AT3G43440). Collect the returned edge-list records — for each path edge the output reports the connecting nodes, the multiplex layer(s) providing the edge, and the edge weight. Parse the resulting shortest_paths_edge_list.tsv and group edges per source-target pair so that path length can be counted as the number of edges. Validate by checking that every source-target pair (2 sources x 2 targets) is represented and that each reported path has consistent layer and weight annotations; in the reference example all 4 paths had a length of 3 (Table 5).

## Related tools

- **RWRtoolkit** (Provides RWR_ShortestPaths and the multiplex infrastructure that produces the annotated edge-list output parsed by this skill) — https://github.com/dkainer/RWRtoolkit
- **RWR_ShortestPaths** (RWRtoolkit function run with source and target gene sets as parameters to extract all possible shortest paths between every source-target pair, returning edges with layer and weight annotations) — https://github.com/dkainer/RWRtoolkit
- **RandomWalkRestartMH** (Base package whose multiplex construction and random-walk functionality RWRtoolkit extends) — https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- **R** (Environment in which RWRtoolkit functions are called and the TSV edge list is parsed)

## Examples

```
el <- read.delim("shortest_paths_edge_list.tsv")
```

## Evaluation signals

- Output edge list exists as shortest_paths_edge_list.tsv and is non-empty
- Edge list contains edges for all 4 source-target pairs (2 sources x 2 targets)
- Each reported path has a length of 3 edges in the reference jasmonate example
- Every edge record carries layer annotation(s) and edge weight — no missing fields
- Reconstructed paths connect exactly the requested source genes to the requested target genes within the comprehensive multiplex

## Limitations

- Multiplex networks are constructed unweighted (delta = 0.5 in the reference network), so edge weights in the output reflect the multiplex construction rather than experimental edge weights
- The RWRtoolkit version number is not stated in the article and no changelog is provided, complicating reproducibility of exact output
- Layer coverage is limited to the assembled layers (e.g. CoEvolution-DUO, Coexpression Gene-Atlas, Knockout Similarity, PPI-6merged, PEN-Diversity, Predictive CG Methylation, Regulation-ATRM, Regulation-Plantregmap, Metabolic-AraCyc) — paths only reflect those evidence types

## Evidence

- [methods] RWR_ShortestPaths invocation: "We ran RWR_ShortestPaths with the source and target gene sets as parameters."
- [readme] RWRtoolkit scope: "RWRtoolkit enables easy use of RandomWalk with Restart on multiplex networks. These functions are an extension to the"
- [other] Validation of edge list: [evidence span withheld — anchor status: not_found]
- [discussion] Connectivity scoring across evidence layers: "RWRtoolkit uses RWR to explore and score topological connectivity from multiple lines of evidence between seed genes and other genes."
- [readme] Multiplex flist format: [evidence span withheld — anchor status: not_found]
