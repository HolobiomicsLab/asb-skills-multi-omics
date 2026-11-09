---
name: tabular-results-file-formatting
description: Reference for persisting RWRtoolkit multiplex statistics — per-layer
  node/edge counts and pairwise between-layer Jaccard similarities — into tab-separated
  .tsv result files for reporting or reuse.
when_to_use_negative:
- The goal is running random walks or scoring gene-gene connectivity (RWR_LOE / Random.Walk.Restart.Multiplex),
  not summarizing layer statistics — netstats does not rank genes
- Layers are weighted or heterogeneous-node networks; the preassembled comprehensive
  multiplex this workflow validates against uses unweighted edges across all 9 layers
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: RWRtoolkit
  role: Provides RWR_netstats, whose basic_statistics and pairwise_between_layers
    options produce the per-layer statistics and between-layer Jaccard tables written
    to file
  repo: https://github.com/dkainer/RWRtoolkit
- name: R
  role: Environment in which RWR_netstats is invoked and from which write.table emits
    the tab-separated output files
- name: RandomWalkRestartMH
  role: Underlying package whose multiplex objects (list of igraph layers) are the
    input structure that netstats summarizes
  repo: https://github.com/alberto-valdeolivas/RandomWalkRestartMH
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:80c3ca7855ad2bef238a55590c57e750b8d5c59913b2888c08b1336b1de20dd2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkainer/RWRtoolkit
  tools:
  - R
  - RWRtoolkit
  - RandomWalkRestartMH
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# tabular-results-file-formatting

## Summary

Generate multiplex network metadata/statistics with RWRtoolkit::RWR_netstats and write them as tab-separated result tables (e.g. base_stats.tsv and pairwise_between_mpo_layer_jaccard.tsv) that downstream tools and readers can consume directly.

## When to use

Reference for persisting RWRtoolkit multiplex statistics — per-layer node/edge counts and pairwise between-layer Jaccard similarities — into tab-separated .tsv result files for reporting or reuse.

## When NOT to use

- The goal is running random walks or scoring gene-gene connectivity (RWR_LOE / Random.Walk.Restart.Multiplex), not summarizing layer statistics — netstats does not rank genes
- Layers are weighted or heterogeneous-node networks; the preassembled comprehensive multiplex this workflow validates against uses unweighted edges across all 9 layers

## Inputs

- Multiplex network RData object (e.g. Comprehensive_Network_AT_d0.5_v02.RData, 9 A. thaliana layers, delta = 0.5)
- flist file defining network layers as {file_path, name, group} triples (if building the multiplex first with RWR_make_multiplex)

## Outputs

- base_stats.tsv — tab-separated per-layer basic statistics (node counts, edge counts, multiplex metadata)
- pairwise_between_mpo_layer_jaccard.tsv — tab-separated pairwise between-layer Jaccard similarity table

## How to apply

Load the preassembled multiplex RData object (e.g. Comprehensive_Network_AT_d0.5_v02.RData from dkainer/RWRtoolkit-data: 9 layers, delta = 0.5, unweighted edges) via RWR_make_multiplex or by loading the .RData directly. Call RWRtoolkit::RWR_netstats with the basic_statistics option to compute per-layer node and edge counts plus multiplex metadata, and write the result to base_stats.tsv with a tab separator. Then re-run RWR_netstats with the pairwise_between_layers option and the default Jaccard similarity score to compute edge-wise overlap for every pair of the 9 layers, writing pairwise_between_mpo_layer_jaccard.tsv. Both outputs should be tab-separated tables covering all layers; the netstats functionality exists precisely to 'concisely generate multiplex metadata and statistics and output those data to file'.

## Related tools

- **RWRtoolkit** (Provides RWR_netstats, whose basic_statistics and pairwise_between_layers options produce the per-layer statistics and between-layer Jaccard tables written to file) — https://github.com/dkainer/RWRtoolkit
- **R** (Environment in which RWR_netstats is invoked and from which write.table emits the tab-separated output files)
- **RandomWalkRestartMH** (Underlying package whose multiplex objects (list of igraph layers) are the input structure that netstats summarizes) — https://github.com/alberto-valdeolivas/RandomWalkRestartMH

## Examples

```
stats <- RWRtoolkit::RWR_netstats(multiplex_object, basic_statistics=TRUE); write.table(stats, 'base_stats.tsv', sep='\t')
```

## Evaluation signals

- Both output files (base_stats.tsv, pairwise_between_mpo_layer_jaccard.tsv) exist and are tab-separated
- base_stats.tsv covers all 9 layers of the comprehensive multiplex with per-layer node/edge counts
- Per-layer counts match the reported sizes, e.g. PPI-6merged: 19,191 nodes / 317,787 edges
- Multiplex totals match 26,605 unique nodes and 918,640 edges
- pairwise_between_mpo_layer_jaccard.tsv contains Jaccard similarities for every pair of the 9 layers (36 pairs)

## Limitations

- RWRtoolkit and RandomWalkRestartMH version numbers are not stated in the article, so exact output schema may vary across versions
- Validation of counts depends on using the same preassembled multiplex (delta = 0.5, unweighted); different layer sets or weighting will produce different statistics
- No changelog was found for the software, complicating reproducibility checks of output formats

## Evidence

- [discussion] netstats outputs written to tab-separated files: "The netstats functionality allows for users to concisely generate multiplex metadata and statistics and output those data to file."
- [methods] RWR_netstats basic_statistics option: "we used the RWR_netstats function to extract basic statistics using the option basic_statistics"
- [readme] preassembled multiplex with delta and unweighted layers: "Contains the following network layers with unweighted edges and delta value = 0.5:"
- [full_text] netstats workflow step including Jaccard output: [evidence span withheld — anchor status: not_found]
