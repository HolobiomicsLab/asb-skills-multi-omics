---
name: gene-set-file-construction
description: Use when preparing seed, source, or target gene sets as input files for
  RWRtoolkit functions (RWR_LOE, RWR_ShortestPaths, RWR_CV) against a multiplex network,
  and genes are given as AGI identifiers.
when_to_use_negative:
- Building the flist of network layer files for RWR_make_multiplex — that file lists
  {file_path, name, group} triples for networks, not gene identifiers
- The gene set is already in RWRtoolkit geneset format and only needs to be passed
  to the command-line tool
- Assigning per-layer seed weights (tau) — tau is a function parameter, not part of
  the gene set file
tools:
- name: RWRtoolkit
  role: Provides the geneset file format and the consuming functions (RWR_LOE, RWR_ShortestPaths)
    whose --seed_geneset / source-target geneset parameters expect these files
  repo: http://github.com/dkainer/RWRtoolkit
- name: R
  role: Runtime environment in which RWRtoolkit gene-set-consuming functions are invoked
    after the file is prepared
- name: RWR_LOE
  role: Consumes a seed set file containing genes such as AT2G44810 and AT1G17420
    to rank all multiplex genes
  repo: http://github.com/dkainer/RWRtoolkit
- name: RWR_ShortestPaths
  role: Consumes separate source and target gene set files to extract shortest paths
    between every source-target pair
  repo: http://github.com/dkainer/RWRtoolkit
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:7a694a2c0e2cfb66356cf8dbcad82424c69288b399d0c44a0a70d13a2e023bc7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - KBase
  - R
  - RWR_LOE
  - RWR_ShortestPaths
  - RWRtoolkit
  - RandomWalkRestartMH
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# gene-set-file-construction

## Summary

Constructs the plain-text gene set / seed set files in RWRtoolkit's geneset format that feed functions like RWR_ShortestPaths (source and target sets) and RWR_LOE (seed sets), e.g. jasmonate-pathway Arabidopsis AGI identifiers such as AT1G13280, AT1G17420, AT2G44810, AT1G48500, and AT3G43440.

## When to use

Use when preparing seed, source, or target gene sets as input files for RWRtoolkit functions (RWR_LOE, RWR_ShortestPaths, RWR_CV) against a multiplex network, and genes are given as AGI identifiers.

## When NOT to use

- Building the flist of network layer files for RWR_make_multiplex — that file lists {file_path, name, group} triples for networks, not gene identifiers
- The gene set is already in RWRtoolkit geneset format and only needs to be passed to the command-line tool
- Assigning per-layer seed weights (tau) — tau is a function parameter, not part of the gene set file

## Inputs

- List of gene identifiers (e.g. Arabidopsis AGI IDs such as AT2G44810, AT1G17420) from a curated annotation set (e.g. MAPMAN jasmonate signalling term)
- Preassembled multiplex RData object (e.g. Comprehensive_Network_AT_d0.5_v02.RData, 9 unweighted layers, delta = 0.5) whose node names the gene IDs must match

## Outputs

- Seed gene set file in RWRtoolkit geneset format (e.g. geneset1.txt) for RWR_LOE
- Source gene set file and target gene set file in RWRtoolkit geneset format for RWR_ShortestPaths

## How to apply

Collect the gene identifiers for the biological question (e.g. MAPMAN jasmonate-signalling genes such as AT2G44810 and AT1G17420) and write them to a plain-text file in RWRtoolkit's geneset format, one gene per row, using the accepted delimiters (tab, comma, or space). For RWR_ShortestPaths, split the set into a separate source gene set file (AT1G13280, AT1G17420) and target gene set file (AT1G48500, AT3G43440) so every source-target pair can be enumerated (2 sources x 2 targets = 4 paths). For RWR_LOE, provide a single seed set file consumed with a restart value (e.g. tau/restart of 0.7). The rationale is that these functions take a file path parameter (--seed_geneset, or source/target geneset arguments) against a preassembled multiplex object (e.g. the 9-layer comprehensive A. thaliana multiplex built with delta = 0.5), so the file format and identifier matching to multiplex node names determine whether the walk can seed at all.

## Related tools

- **RWRtoolkit** (Provides the geneset file format and the consuming functions (RWR_LOE, RWR_ShortestPaths) whose --seed_geneset / source-target geneset parameters expect these files) — http://github.com/dkainer/RWRtoolkit
- **R** (Runtime environment in which RWRtoolkit gene-set-consuming functions are invoked after the file is prepared)
- **RWR_LOE** (Consumes a seed set file containing genes such as AT2G44810 and AT1G17420 to rank all multiplex genes) — http://github.com/dkainer/RWRtoolkit
- **RWR_ShortestPaths** (Consumes separate source and target gene set files to extract shortest paths between every source-target pair) — http://github.com/dkainer/RWRtoolkit

## Examples

```
Rscript <LIBPATHS_DIRECTORY>/RWRtoolkit/scripts/run_loe.R --data <multiplex>.Rdata --seed_geneset <LIBPATHS_DIRECTORY>/RWRtoolkit/example_data/geneset1.txt --tau "1.0,1.0" -o ./outdir
```

## Evaluation signals

- The geneset file exists and each row contains one gene identifier that matches a node name in the target multiplex object
- For RWR_ShortestPaths, the two files partition the genes correctly: 2 source genes (AT1G13280, AT1G17420) and 2 target genes (AT1G48500, AT3G43440), yielding exactly 4 source-target pairs in downstream output
- RWR_LOE run on the seed file returns scores and ranks for all multiplex genes, and top-ranked genes are functionally related (jasmonate-associated) to the seeds
- RWR_ShortestPaths output (shortest_paths_edge_list.tsv) contains edges for all 4 source-target pairs with layer and weight annotations present

## Limitations

- Gene identifiers must match the multiplex node names exactly; unmatched IDs cannot seed the random walk and are silently or explicitly dropped
- The README does not fully document the geneset file schema in the truncated portion, so format details must be confirmed against the example_data/geneset1.txt shipped with the package

## Evidence

- [methods] We used the RWR_LOE function with a seed set file containing the seed genes of AT2G44810 and AT1G17420.: "We used the RWR_LOE function with a seed set file containing the seed genes of AT2G44810 and AT1G17420."
- [methods] We ran RWR_ShortestPaths with the source and target gene sets as parameters.: "We ran RWR_ShortestPaths with the source and target gene sets as parameters."
- [readme] Rscript <LIBPATHS_DIRECTORY>/RWRtoolkit/scripts/run_loe.R --data ... --seed_geneset <LIBPATHS_DIRECTORY>/RWRtoolkit/example_data/geneset1.txt: [evidence span withheld — anchor status: not_found]
