---
name: cytoscape-subnetwork-export
description: 'Use when an RWR_LOE ranking over a multiplex network must be visualized:
  you have seed genes, a ranked gene output, and need the top-N (e.g., 50) highest-ranked
  nodes extracted and sent to Cytoscape for exploration.'
when_to_use_negative:
- When the goal is statistical validation of gene-set connectivity rather than neighborhood
  context — use RWR_CV (leave-one-out cross-validation) instead
- When exact topology-based distances between seed and target sets are needed — use
  RWR_ShortestPaths instead of RWR ranking plus visualization
- When the network layers should be collapsed into a single aggregated network — the
  aggregated network loses the unique topological information carried by each layer
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: RWRtoolkit
  role: Provides RWR_make_multiplex and RWR_LOE with the cyto parameter that extracts
    the top-ranked subnetwork for export
  repo: https://github.com/dkainer/RWRtoolkit
- name: RandomWalkRestartMH
  role: Underlying package providing core functionality for constructing multiplex
    networks and implementing random walk with restart
  repo: https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- name: RCy3
  role: R API through which the top-N subnetwork is exported to Cytoscape
- name: Cytoscape
  role: External visualization target where the subnetwork is loaded and edges colored
    by layer name for evidence exploration
- name: KBase
  role: Alternative reproducible-workflow route for running RWR_LOE and visualizing
    subnetworks via a web app GUI
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:7fb234fac1723bffcaa335794ac8c3586dc53c4157d420c410e980c6f1f304af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dkainer/RWRtoolkit
  tools:
  - Cytoscape
  - KBase
  - R
  - RCy3
  - RWRtoolkit
  - RandomWalkRestartMH
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# cytoscape-subnetwork-export

## Summary

Exports the top-ranked neighborhood of seed genes from a Random Walk with Restart (RWR_LOE) run on a multiplex network to Cytoscape, using the cyto parameter (e.g., cyto=50) and RCy3 so the ranked genes are rendered as a subnetwork with edges colored by layer name.

## When to use

Use when an RWR_LOE ranking over a multiplex network must be visualized: you have seed genes, a ranked gene output, and need the top-N (e.g., 50) highest-ranked nodes extracted and sent to Cytoscape for exploration.

## When NOT to use

- When the goal is statistical validation of gene-set connectivity rather than neighborhood context — use RWR_CV (leave-one-out cross-validation) instead
- When exact topology-based distances between seed and target sets are needed — use RWR_ShortestPaths instead of RWR ranking plus visualization
- When the network layers should be collapsed into a single aggregated network — the aggregated network loses the unique topological information carried by each layer

## Inputs

- Multiplex network RData object (e.g., Comprehensive_Network_AT_d0.5_v02.RData, 9 unweighted layers, delta = 0.5)
- Seed gene set file (plain-text gene IDs, e.g., AT2G44810 and AT1G17420)
- Ranked RWR_LOE output table (random walk scores and ranks for every gene in the multiplex)

## Outputs

- Full ranked gene output table with random walk scores and ranks for all multiplex genes
- Top-50 subnetwork (cyto = 50) exported to Cytoscape via RCy3, with edges colored by layer name

## How to apply

Load the preassembled multiplex (e.g., the 9-layer comprehensive Arabidopsis thaliana network built with delta = 0.5) via RWRtoolkit's RWR_make_multiplex, then run RWR_LOE with a seed set file (e.g., jasmonate genes AT2G44810 and AT1G17420) and a restart value of 0.7 to score and rank all genes. Set cyto = 50 so RWR_LOE extracts the subnetwork of the 50 highest-ranked nodes surrounding the seeds and exports it via the RCy3 API to Cytoscape, where edges are colored by layer name (coexpression, protein–protein interactions, etc.). Rationale: random-walk scores provide multi-omic topological context that is hard to read from a ranked table alone; the Cytoscape view lets users inspect the lines of evidence behind each gene-to-gene interaction. An alternative route is the KBase web application, where RWR_LOE and visualization are available as reproducible workflows.

## Related tools

- **RWRtoolkit** (Provides RWR_make_multiplex and RWR_LOE with the cyto parameter that extracts the top-ranked subnetwork for export) — https://github.com/dkainer/RWRtoolkit
- **RandomWalkRestartMH** (Underlying package providing core functionality for constructing multiplex networks and implementing random walk with restart) — https://github.com/alberto-valdeolivas/RandomWalkRestartMH
- **RCy3** (R API through which the top-N subnetwork is exported to Cytoscape)
- **Cytoscape** (External visualization target where the subnetwork is loaded and edges colored by layer name for evidence exploration)
- **KBase** (Alternative reproducible-workflow route for running RWR_LOE and visualizing subnetworks via a web app GUI)

## Examples

```
Rscript <LIBPATHS_DIRECTORY>/RWRtoolkit/scripts/run_loe.R --data <LIBPATHS_DIRECTORY>/RWRtoolkit/example_data/string_interactions.Rdata --seed_geneset <LIBPATHS_DIRECTORY>/RWRtoolkit/example_data/geneset1.txt --tau "1.0,1.0" -o ./outdir
```

## Evaluation signals

- The ranked output file exists and contains random walk scores and ranks for all genes in the multiplex
- The exported subnetwork contains exactly 50 nodes when cyto = 50
- Top-ranked genes are functionally related to the seeds (e.g., jasmonate-associated for AT2G44810 and AT1G17420)
- In Cytoscape, edges are colored by layer name so each interaction can be traced to its evidence layer
- The restart value (r = 0.7) matches the intended random walk configuration for the run

## Limitations

- In the RandomWalkRestartMH implementation, tau incorrectly weights each seed equally across all layers rather than differently per layer, which led Valdeolivas et al. to conclude tau had minimal effect on output rankings
- No version number is stated for RWRtoolkit or RandomWalkRestartMH, complicating reproducibility of the visualization workflow
- Aggregating layers into a single network (instead of keeping the multiplex) loses the unique topological information carried by each layer

## Evidence

- [discussion] RCy3 subnetwork export route: "The software offers multiple routes for visualizing these subnetsworks via the RCy3 API to Cytoscape or the KBase web app."
- [discussion] Cytoscape evidence exploration: "Users can further explore the lines of evidence supporting gene-to-gene interactions (coexpression, protein–protein interactions, etc.) using Cytoscape"
- [methods] RWR_LOE seed set in methods: "We used the RWR_LOE function with a seed set file containing the seed genes of AT2G44810 and AT1G17420."
- [readme] RWRtoolkit usage modes from README: [evidence span withheld — anchor status: not_found]
