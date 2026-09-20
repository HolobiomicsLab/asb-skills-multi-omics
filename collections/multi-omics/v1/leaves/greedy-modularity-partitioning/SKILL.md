---
name: greedy-modularity-partitioning
description: Use when a program-level integration network (e.g. program_graph.graphml
  plus pearson.df.npz from mosaicMPI cNMF programs) must be partitioned into communities
  to reveal cross-modality biological themes.
when_to_use_negative:
- Input is raw expression data or an unfactorized dataset — program discovery via
  cNMF and network construction must happen first before communities can be detected
- The goal is to keep singleton programs in the partition — this workflow prunes communities
  containing only a single node
- Datasets share samples and features fully, so a direct cross-dataset integration
  tool is more appropriate than program-level mosaic integration
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: mosaicMPI
  role: Framework whose anchor network of cNMF-derived programs is the input graph
    for community detection
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: pandas
  role: Load and write program tables and the node-to-community assignment CSV
- name: NumPy
  role: Read the pearson.df.npz thresholded Pearson correlation matrix of all programs
    within and between datasets
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:7060473ccbb76a782074fce363da09981f466ba62850c54ce811cd83c27f7c5a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/MorrissyLab/mosaicMPI
  repo_url: https://github.com/MorrissyLab/mosaicMPI
  tools:
  - NumPy
  - mosaicMPI
  - pandas
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# greedy-modularity-partitioning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs greedy modularity community detection on the mosaicMPI anchor program network built from thresholded Pearson correlations between multi-omics programs, yielding the C1-C23 community landscape of the CPTAC GBM integration, then prunes single-node communities and reports community-modality composition.

## When to use

Use when a program-level integration network (e.g. program_graph.graphml plus pearson.df.npz from mosaicMPI cNMF programs) must be partitioned into communities to reveal cross-modality biological themes.

## When NOT to use

- Input is raw expression data or an unfactorized dataset — program discovery via cNMF and network construction must happen first before communities can be detected
- The goal is to keep singleton programs in the partition — this workflow prunes communities containing only a single node
- Datasets share samples and features fully, so a direct cross-dataset integration tool is more appropriate than program-level mosaic integration

## Inputs

- program_graph.graphml — anchor program network from the mosaicMPI integration step
- pearson.df.npz — thresholded Pearson correlation matrix of all programs within and between datasets
- program-to-modality/dataset metadata for community composition analysis

## Outputs

- CSV node-to-community assignment table with communities labelled C1-C23
- Fruchterman-Reingold spring-embedded network plot colored by community assignment
- community-modality composition summary (e.g. 17 communities spanning all three modalities)

## How to apply

Load the anchor program network produced by the prior integration step (program_graph.graphml) and the thresholded Pearson correlation matrix (pearson.df.npz) using pandas/NumPy. Run greedy modularity community detection on the anchor network's nodes and edges, then prune communities containing only a single node so the partition reflects multi-program themes rather than isolated programs. Write the node-to-community assignment table as a CSV with communities labelled C1-C23. Render a Fruchterman-Reingold spring-embedded layout of the program network colored by community assignment. Validate that the partition yields 23 communities and that the community-modality composition (17 communities spanning all three modalities, 3 communities found by two modalities) matches Figure 2G of the article.

## Related tools

- **mosaicMPI** (Framework whose anchor network of cNMF-derived programs is the input graph for community detection) — https://github.com/MorrissyLab/mosaicMPI
- **pandas** (Load and write program tables and the node-to-community assignment CSV)
- **NumPy** (Read the pearson.df.npz thresholded Pearson correlation matrix of all programs within and between datasets)

## Examples

```
import networkx as nx; G = nx.read_graphml("program_graph.graphml"); comms = nx.community.greedy_modularity_communities(G)
```

## Evaluation signals

- Partition yields exactly 23 communities labelled C1-C23 after pruning single-node communities
- Community-modality composition matches Figure 2G: 17 communities span all three modalities, 3 communities are found by two modalities, remainder are single-modality
- Every assigned node appears in exactly one community in the exported CSV
- Spring-embedded layout renders without disconnected singleton communities retained from the raw partition

## Limitations

- Communities with only a single node are pruned, so isolated programs are dropped from the labelled C1-C23 assignment
- A potential concern with mosaic integration is loss of relevant information between modalities given that not all features are shared
- No software version information is stated for the tools/methods discussed in the discussion section, complicating reproducibility of the exact partition

## Evidence

- [readme] A flexible network-based approach groups similar programs together across resolutions and datasets.: [evidence span withheld — anchor status: external_doc]
- [readme] Program communities are then interpreted using sample/cell metadata and gene set analyses.: [evidence span withheld — anchor status: external_doc]
