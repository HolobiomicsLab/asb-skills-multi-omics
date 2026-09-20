---
name: multiplex-network-random-walk-scoring
description: Use when you have a set of seed genes (e.g. from a pathway or GWAS) and
  a preassembled or custom multiplex network (multiple omic layers), and need ranked
  topological context around the seeds.
when_to_use_negative:
- You need to test whether a gene set is unusually interconnected (statistical validation)
  rather than rank neighbors — use RWR_CV cross-validation instead of RWR_LOE
- Your input is a single aggregated network rather than a multiplex — aggregation
  loses the unique topological information carried by each layer, which this method
  depends on
- You want shortest-path distances between two gene sets — use RWR_ShortestPaths rather
  than the RWR scoring of RWR_LOE
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0605
tools:
- name: RWRtoolkit
  role: Provides RWR_make_multiplex to build the multiplex and RWR_LOE to score and
    rank genes around the seed set, plus Cytoscape export (cyto parameter)
  repo: http://github.com/dkainer/RWRtoolkit
- name: RandomWalkRestartMH
  role: Underlying R package supplying core random walk with restart on multiplex
    network functionality used by RWRtoolkit
  repo: https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- name: Cytoscape
  role: Visualization target for the top-ranked seed-neighborhood subnetwork via the
    RCy3 API, with edges colored by layer
- name: KBase
  role: Alternative reproducible-workflow GUI route for running RWR_LOE and visualizing
    subnetworks
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:e907f7d92376e3781389ff16deaff64b98f0cf9429ac2096387ae532f3d3d74a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cytoscape
  - KBase
  - R
  - RWRtoolkit
  - RandomWalkRestartMH
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# multiplex-network-random-walk-scoring

## Summary

Scores and ranks all genes in a multi-layer (multiplex) biological network with respect to a seed gene set using random walk with restart (RWR_LOE from RWRtoolkit, built on RandomWalkRestartMH), then exports a top-ranked subnetwork (e.g. top 50 nodes) for visualization in Cytoscape via RCy3.

## When to use

Use when you have a set of seed genes (e.g. from a pathway or GWAS) and a preassembled or custom multiplex network (multiple omic layers), and need ranked topological context around the seeds.

## When NOT to use

- You need to test whether a gene set is unusually interconnected (statistical validation) rather than rank neighbors — use RWR_CV cross-validation instead of RWR_LOE
- Your input is a single aggregated network rather than a multiplex — aggregation loses the unique topological information carried by each layer, which this method depends on
- You want shortest-path distances between two gene sets — use RWR_ShortestPaths rather than the RWR scoring of RWR_LOE

## Inputs

- Multiplex network RData object or flist file (file list of {file_path, name, group} triples) defining network layers, e.g. Comprehensive_Network_AT_d0.5_v02.RData
- Seed geneset file (plain text of gene IDs, e.g. AT2G44810, AT1G17420)
- tau vector of per-layer seed weights

## Outputs

- Ranked gene output table with random walk score and rank for every gene in the multiplex
- Top-N (e.g. 50-node) seed-neighborhood subnetwork exported to Cytoscape (via RCy3) with edges colored by layer
- Optionally the multiplex object from RWR_make_multiplex

## How to apply

Build or load a multiplex network object with RWR_make_multiplex (e.g. the 9-layer preassembled Arabidopsis thaliana comprehensive multiplex with delta = 0.5). Provide a seed set file listing the genes of interest (e.g. jasmonate-pathway genes AT2G44810 and AT1G17420), optionally with per-layer tau weights. Run RWR_LOE with a restart value (the exemplar run used 0.7) so the walk scores every gene in the multiplex by connectivity to the seeds across all layers; write the full ranked output table (score and rank for each gene). Set cyto = 50 to extract the 50 highest-ranked nodes around the seeds and export that subnetwork to Cytoscape via RCy3, with edges colored by layer name, or use the KBase web app route.

## Related tools

- **RWRtoolkit** (Provides RWR_make_multiplex to build the multiplex and RWR_LOE to score and rank genes around the seed set, plus Cytoscape export (cyto parameter)) — http://github.com/dkainer/RWRtoolkit
- **RandomWalkRestartMH** (Underlying R package supplying core random walk with restart on multiplex network functionality used by RWRtoolkit) — https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- **Cytoscape** (Visualization target for the top-ranked seed-neighborhood subnetwork via the RCy3 API, with edges colored by layer)
- **KBase** (Alternative reproducible-workflow GUI route for running RWR_LOE and visualizing subnetworks)

## Examples

```
Rscript <LIBPATHS_DIRECTORY>/RWRtoolkit/scripts/run_loe.R --data <LIBPATHS_DIRECTORY>/RWRtoolkit/example_data/string_interactions.Rdata --seed_geneset <LIBPATHS_DIRECTORY>/RWRtoolkit/example_data/geneset1.txt --tau "1.0,1.0" -o ./outdir
```

## Evaluation signals

- The ranked output file exists and contains a score and rank for every gene in the multiplex (not just the seeds)
- The exported Cytoscape subnetwork contains exactly the requested number of top-ranked nodes (50 in the exemplar run, via cyto=50)
- Top-ranked genes are functionally related to the seeds (e.g. jasmonate-associated genes rank high for jasmonate-pathway seeds AT2G44810 and AT1G17420)
- The multiplex object was built with the intended delta value (0.5 in the exemplar) and contains the expected 9 layers
- In the visualization, edges are colored by layer name so each line of evidence (coexpression, PPI, regulation, etc.) is distinguishable

## Limitations

- In the RandomWalkRestartMH implementation, tau incorrectly weighted each seed equally across all layers rather than differently per layer, leading Valdeolivas et al. to conclude tau had minimal effect on output rankings; RWRtoolkit updates this underlying functionality
- RandomWalkRestartMH multiplex construction was not optimized and computational load increases exponentially with network size and layer count (RWRtoolkit parallelizes construction, reducing hours to seconds)
- No version number was stated for RWRtoolkit or RandomWalkRestartMH in the article's discussion

## Evidence

- [discussion] RWR_LOE ranks genes around a seed set: "RWRtoolkit uses RWR to explore and score topological connectivity from multiple lines of evidence between seed genes and other genes."
- [methods] Exemplar seed set for RWR_LOE: "We used the RWR_LOE function with a seed set file containing the seed genes of AT2G44810 and AT1G17420."
- [discussion] Subnetwork visualization routes: "The software offers multiple routes for visualizing these subnetsworks via the RCy3 API to Cytoscape or the KBase web app."
- [readme] CLI invocation of the LOE script from the README: "Rscript <LIBPATHS_DIRECTORY>/RWRtoolkit/scripts/run_loe.R 
    --data            <LIBPATHS_DIRECTORY>/RWRtoolkit/example_data/string_interactions.Rdata \"
- [readme] Multiplex must be built first with RWR_make_multiplex: [evidence span withheld — anchor status: not_found]
