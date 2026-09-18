---
name: layer-pairwise-jaccard-similarity-computation
description: Use when you have a preassembled multiplex network object (e.g., the
  9-layer Arabidopsis thaliana multiplex with delta = 0.5) and need per-layer node/edge
  counts plus edge-wise overlap between every pair of layers.
when_to_use_negative:
- When the analysis target is the aggregated (summarized) network, which 'has lost
  the unique topological information carried by each layer' — layer-pair Jaccard is
  meaningless there
- When you need gene-set validation or ranking (RWR_CV / RWR_LOE) rather than multiplex
  metadata; RWR_netstats does not score seeds
- When the input is raw edge list files rather than a constructed multiplex object
  — build it first with RWR_make_multiplex/create.multiplex
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0605
tools:
- name: RWRtoolkit
  role: 'Provides RWR_netstats, run twice: once with the basic_statistics option and
    once with the pairwise_between_layers option using the default Jaccard similarity
    score'
  repo: https://github.com/dkainer/RWRtoolkit
- name: RandomWalkRestartMH
  role: Underlying package providing core functionality for constructing the multiplex
    network object that RWR_netstats analyzes
  repo: https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- name: R
  role: Execution environment for loading the multiplex object and calling RWRtoolkit::RWR_netstats
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:1290e40a4f98dd459b7ca049f816bf93153c039174c8aef4e00d2c0554309526
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkainer/RWRtoolkit
  tools:
  - R
  - RWRtoolkit
  - RandomWalkRestartMH
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# layer-pairwise-jaccard-similarity-computation

## Summary

Compute per-layer basic statistics and pairwise between-layer edge-overlap (Jaccard) similarities for a multiplex network using RWRtoolkit's RWR_netstats function, yielding per-layer node/edge counts and a full layer-pair Jaccard similarity table.

## When to use

Use when you have a preassembled multiplex network object (e.g., the 9-layer Arabidopsis thaliana multiplex with delta = 0.5) and need per-layer node/edge counts plus edge-wise overlap between every pair of layers.

## When NOT to use

- When the analysis target is the aggregated (summarized) network, which 'has lost the unique topological information carried by each layer' — layer-pair Jaccard is meaningless there
- When you need gene-set validation or ranking (RWR_CV / RWR_LOE) rather than multiplex metadata; RWR_netstats does not score seeds
- When the input is raw edge list files rather than a constructed multiplex object — build it first with RWR_make_multiplex/create.multiplex

## Inputs

- Preassembled multiplex network RData object (e.g., Comprehensive_Network_AT_d0.5_v02.RData, 9 unweighted layers, delta = 0.5)
- R environment with RWRtoolkit (>= R 4.1.0) installed, building on RandomWalkRestartMH

## Outputs

- base_stats.tsv — tab-separated table of per-layer basic statistics (node counts, edge counts, multiplex metadata)
- pairwise_between_mpo_layer_jaccard.tsv — tab-separated table of between-layer Jaccard similarities for all layer pairs

## How to apply

Load the preassembled multiplex RData object (e.g., Comprehensive_Network_AT_d0.5_v02.RData from the dkainer/RWRtoolkit-data deposit, 9 unweighted layers, delta = 0.5) in R. Run RWRtoolkit::RWR_netstats once with the basic_statistics option to compute per-layer node and edge counts and multiplex metadata, and write the result to base_stats.tsv. Re-run RWR_netstats with the pairwise_between_layers option using the default Jaccard similarity score to compute edge-wise overlap between every pair of the 9 layers, writing pairwise_between_mpo_layer_jaccard.tsv. The Jaccard default is used as-is; no additional threshold is applied by this skill. Validate that both tab-separated outputs cover all 9 layers and that per-layer counts match known layer sizes.

## Related tools

- **RWRtoolkit** (Provides RWR_netstats, run twice: once with the basic_statistics option and once with the pairwise_between_layers option using the default Jaccard similarity score) — https://github.com/dkainer/RWRtoolkit
- **RandomWalkRestartMH** (Underlying package providing core functionality for constructing the multiplex network object that RWR_netstats analyzes) — https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- **R** (Execution environment for loading the multiplex object and calling RWRtoolkit::RWR_netstats)

## Examples

```
RWRtoolkit::RWR_netstats(multiplex = mpo, basic_statistics = TRUE); RWRtoolkit::RWR_netstats(multiplex = mpo, pairwise_between_layers = TRUE)
```

## Evaluation signals

- Both base_stats.tsv and pairwise_between_mpo_layer_jaccard.tsv exist and are valid tab-separated tables
- Per-layer node/edge counts match reported layer sizes, e.g., PPI-6merged: 19,191 nodes / 317,787 edges (Table 2 of the article)
- The comprehensive multiplex totals check out: 9 layers, 26,605 unique nodes, 918,640 edges
- The pairwise Jaccard table spans all pairs of the 9 layers with similarity values in the valid [0, 1] Jaccard range
- Per-layer statistics are consistent with the multiplex being constructed with delta = 0.5 and all layers unweighted

## Limitations

- The preassembled Arabidopsis multiplex layers are unweighted; weighted-layer statistics are not exercised by this demonstrated workflow
- No version number is stated for RWRtoolkit or RandomWalkRestartMH in the article, complicating reproducibility pinning
- netstats outputs metadata/statistics only; it does not evaluate gene-set topology (that requires RWR_CV) or seed rankings (RWR_LOE)

## Evidence

- [methods] RWR_netstats basic statistics: "we used the RWR_netstats function to extract basic statistics using the option basic_statistics"
- [discussion] netstats output to file: "The netstats functionality allows for users to concisely generate multiplex metadata and statistics and output those data to file."
- [readme] comprehensive multiplex definition: "Contains the following network layers with unweighted edges and delta value = 0.5:"
- [abstract] RWRtoolkit package purpose: "We introduce RWRtoolkit, a multiplex generation, exploration, and statistical package built for R and command-line users."
