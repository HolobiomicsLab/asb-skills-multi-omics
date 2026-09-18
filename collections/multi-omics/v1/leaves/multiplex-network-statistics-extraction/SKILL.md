---
name: multiplex-network-statistics-extraction
description: 'Use when a multiplex network object (.Rdata from RWR_make_multiplex
  or a preassembled multiplex) needs characterization: per-layer node/edge counts
  and between-layer edge overlap (Jaccard) before running RWR analyses.'
when_to_use_negative:
- Input is a flat aggregate/summarized network rather than a multiplex object — the
  aggregated network 'has lost the unique topological information carried by each
  layer', so per-layer statistics and between-layer Jaccard are undefined
- The goal is running random walks (RWR_LOE/RWR_CV) rather than characterizing layer
  composition and overlap
- The multiplex has not yet been constructed — build it first with RWR_make_multiplex/create.multiplex
  from an flist of layer files
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0091
tools:
- name: RWRtoolkit
  role: Provides the RWR_netstats function with basic_statistics and pairwise_between_layers
    options used to generate per-layer statistics and pairwise Jaccard outputs
  repo: http://github.com/dkainer/RWRtoolkit
- name: R
  role: Execution environment in which RWRtoolkit::RWR_netstats is called on the multiplex
    object
- name: RandomWalkRestartMH
  role: Underlying package providing core multiplex network construction on which
    RWRtoolkit's netstats builds
  repo: https://github.com/alberto-valdeolivas/RandomWalkRestartMH
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:c3ac573a5952272d3dd7778ace9bd4a6ddb1e4cdeeacaff5fdf4e795c4cbfe05
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkainer/RWRtoolkit
  tools:
  - R
  - RWRtoolkit
  - RandomWalkRestartMH
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# multiplex-network-statistics-extraction

## Summary

Generate multiplex metadata and statistics with RWR_netstats, producing per-layer basic statistics (node/edge counts) and pairwise between-layer Jaccard similarity tables for a multiplex network, as demonstrated on the 9-layer comprehensive Arabidopsis thaliana multiplex.

## When to use

Use when a multiplex network object (.Rdata from RWR_make_multiplex or a preassembled multiplex) needs characterization: per-layer node/edge counts and between-layer edge overlap (Jaccard) before running RWR analyses.

## When NOT to use

- Input is a flat aggregate/summarized network rather than a multiplex object — the aggregated network 'has lost the unique topological information carried by each layer', so per-layer statistics and between-layer Jaccard are undefined
- The goal is running random walks (RWR_LOE/RWR_CV) rather than characterizing layer composition and overlap
- The multiplex has not yet been constructed — build it first with RWR_make_multiplex/create.multiplex from an flist of layer files

## Inputs

- Multiplex network RData object (e.g., Comprehensive_Network_AT_d0.5_v02.RData; 9 layers, delta = 0.5, unweighted)
- Option flags: basic_statistics and pairwise_between_layers for RWR_netstats

## Outputs

- base_stats.tsv: tab-separated per-layer basic statistics (node and edge counts, multiplex metadata)
- pairwise_between_mpo_layer_jaccard.tsv: tab-separated between-layer Jaccard similarities for all layer pairs

## How to apply

Load the preassembled multiplex network object (e.g., Comprehensive_Network_AT_d0.5_v02.RData from the dkainer/RWRtoolkit-data deposit: 9 layers, delta = 0.5, unweighted edges). In R, call RWRtoolkit::RWR_netstats with the basic_statistics option to compute per-layer node and edge counts and multiplex metadata, and write the result to base_stats.tsv. Re-run RWR_netstats with the pairwise_between_layers option using the default Jaccard similarity score to compute edge-wise overlap between every pair of layers, and write the result to pairwise_between_mpo_layer_jaccard.tsv. The rationale is that the netstats functionality 'allows for users to concisely generate multiplex metadata and statistics and output those data to file', revealing how much evidence is shared between layers before interpreting downstream RWR rankings.

## Related tools

- **RWRtoolkit** (Provides the RWR_netstats function with basic_statistics and pairwise_between_layers options used to generate per-layer statistics and pairwise Jaccard outputs) — http://github.com/dkainer/RWRtoolkit
- **R** (Execution environment in which RWRtoolkit::RWR_netstats is called on the multiplex object)
- **RandomWalkRestartMH** (Underlying package providing core multiplex network construction on which RWRtoolkit's netstats builds) — https://github.com/alberto-valdeolivas/RandomWalkRestartMH

## Examples

```
Rscript <LIBPATHS>/RWRtoolkit/scripts/run_netstats.R --data Comprehensive_Network_AT_d0.5_v02.RData --basic_statistics --pairwise_between_layers -o ./outdir
```

## Evaluation signals

- Both base_stats.tsv and pairwise_between_mpo_layer_jaccard.tsv exist and are tab-separated tables covering all 9 layers
- Per-layer node/edge counts match reported layer sizes, e.g., PPI-6merged: 19,191 nodes / 317,787 edges (Table 2)
- Whole-multiplex totals match the reported 26,605 unique nodes and 918,640 edges for the comprehensive Arabidopsis multiplex
- Pairwise Jaccard table covers all pairwise combinations of the 9 layers (36 pairs) with default Jaccard similarity scores

## Limitations

- No version number stated for RWRtoolkit or RandomWalkRestartMH, so exact netstats behavior may vary across installs
- Statistics are computed on the multiplex as constructed (delta = 0.5, unweighted layers); weighted-layer variants are not covered by the demonstrated workflow
- Jaccard similarity measures edge overlap only; it does not capture node-level or weight-based layer similarity

## Evidence

- [discussion] The netstats functionality allows for users to concisely generate multiplex metadata and statistics and output those data to file.: "The netstats functionality allows for users to concisely generate multiplex metadata and statistics and output those data to file."
- [methods] we used the RWR_netstats function to extract basic statistics using the option basic_statistics: "we used the RWR_netstats function to extract basic statistics using the option basic_statistics"
- [readme] Contains the following network layers with unweighted edges and delta value = 0.5: "Contains the following network layers with unweighted edges and delta value = 0.5:"
- [discussion] preassembled Arabidopsis thaliana multiplex networks: "we have included RWR_LOE and RWR_CV as applications within KBase and provided preassembled Arabidopsis thaliana multiplex networks."
