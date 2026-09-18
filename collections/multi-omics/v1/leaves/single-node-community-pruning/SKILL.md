---
name: single-node-community-pruning
description: Use when a program graph (e.g. program_graph.graphml built from pearson.df.npz
  correlations) has been partitioned into communities and singleton communities must
  be removed before labelling and reporting community-modality composition.
when_to_use_negative:
- When the goal is program discovery itself (cNMF factorization) rather than partitioning
  of an already-built anchor network — this skill assumes the graph exists.
- When every detected community is multi-node or the analysis requires retaining singletons
  for later merging — pruning would discard valid nodes from the partition.
- When inputs are raw expression matrices or AnnData/h5ad objects rather than the
  integrated program graph — mosaicMPI integration must run first.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: mosaicMPI
  role: Produces the integrated anchor program network whose communities are detected
    and pruned; provides the program landscape context for the C1–C23 partition.
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: NumPy
  role: Reads the pearson.df.npz thresholded Pearson correlation matrix underlying
    the network edges.
- name: pandas
  role: Loads program/modality metadata and writes the node-to-community CSV assignment
    table.
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:4f92c8755157e27d011168839e99349a057b7dcda4fb4b4e0756a78c6c7d9d84
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
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# single-node-community-pruning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prunes singleton (single-node) communities from a greedy modularity partition of the mosaicMPI anchor program network, yielding the labelled C1–C23 community landscape reported for the CPTAC GBM integration.

## When to use

Use when a program graph (e.g. program_graph.graphml built from pearson.df.npz correlations) has been partitioned into communities and singleton communities must be removed before labelling and reporting community-modality composition.

## When NOT to use

- When the goal is program discovery itself (cNMF factorization) rather than partitioning of an already-built anchor network — this skill assumes the graph exists.
- When every detected community is multi-node or the analysis requires retaining singletons for later merging — pruning would discard valid nodes from the partition.
- When inputs are raw expression matrices or AnnData/h5ad objects rather than the integrated program graph — mosaicMPI integration must run first.

## Inputs

- program_graph.graphml — anchor program network from the mosaicMPI integration step
- pearson.df.npz — thresholded Pearson correlation matrix of all programs within and between datasets (NumPy-compatible)
- program-to-dataset/modality metadata table (pandas-readable) for computing community-modality composition

## Outputs

- CSV node-to-community assignment table with communities labelled C1–C23
- Spring-embedded (Fruchterman-Reingold) network figure colored by community assignment
- Community-modality composition summary (e.g. 17/23 communities spanning all three modalities)

## How to apply

Load the anchor program network produced by the integration step (program_graph.graphml) together with the thresholded Pearson correlation matrix (pearson.df.npz) using pandas/NumPy. Run greedy modularity community detection on the network's nodes and edges to obtain a candidate partition. Discard any community containing only a single node, since singletons carry no cross-program integration signal. Write the remaining node-to-community assignments as a CSV with communities relabelled C1–C23 (in the CPTAC GBM case this recovers 23 communities, 17 of which span all three modalities, 3 found by two modalities, and the rest single-modality). Optionally render the network in a Fruchterman-Reingold spring-embedded layout colored by community to inspect the mosaicMPI landscape.

## Related tools

- **mosaicMPI** (Produces the integrated anchor program network whose communities are detected and pruned; provides the program landscape context for the C1–C23 partition.) — https://github.com/MorrissyLab/mosaicMPI
- **NumPy** (Reads the pearson.df.npz thresholded Pearson correlation matrix underlying the network edges.)
- **pandas** (Loads program/modality metadata and writes the node-to-community CSV assignment table.)

## Examples

```
import networkx as nx; from networkx.algorithms.community import greedy_modularity_communities; G = nx.read_graphml('program_graph.graphml'); comms = [c for c in greedy_modularity_communities(G) if len(c) > 1]
```

## Evaluation signals

- The pruned partition contains exactly 23 communities relabelled C1–C23.
- Community-modality composition matches Figure 2G: 17 communities span all three modalities, 3 communities are found by two modalities, and the remainder are single-modality.
- No community in the output CSV contains fewer than two nodes.
- Every node in program_graph.graphml either appears in exactly one retained community or was pruned as a singleton — no duplicates or missing assignments.
- The spring-embedded layout renders without disconnected singleton islands, confirming pruning was applied before visualization.

## Limitations

- Greedy modularity community detection is heuristic — the partition may vary with network thresholding choices, so validation against the reported 23-community composition is required.
- Pruning removes single-node communities permanently; those programs are excluded from community-level metadata transfer and downstream community-representative program analyses.
- Community interpretation depends on modality metadata coverage; communities spanning all modalities rely on mosaic integration across datasets with incomplete feature and sample overlap.

## Evidence

- [readme] README network-based grouping: [evidence span withheld — anchor status: external_doc]
- [readme] README program communities interpreted via metadata: [evidence span withheld — anchor status: external_doc]
