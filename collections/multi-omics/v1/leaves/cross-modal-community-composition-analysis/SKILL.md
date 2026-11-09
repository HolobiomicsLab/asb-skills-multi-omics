---
name: cross-modal-community-composition-analysis
description: Use when an anchor network of molecular programs has been integrated
  across datasets/modalities (e.g. from a thresholded Pearson correlation matrix)
  and you need its community partition and the modality composition of each community.
when_to_use_negative:
- You have not yet built the anchor network — run program discovery (cNMF) and program
  integration first; this skill operates on the integrated program graph, not raw
  expression matrices (e.g. cptac_snRNA_subsampled.txt).
- You need community-representative program feature scores for downstream pathway/marker
  analysis rather than the partition and its modality composition.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: mosaicMPI
  role: Produces the anchor program network being partitioned; community detection
    and community-based annotation are part of its network-based integration approach
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: pandas
  role: Loads data matrices and writes the node-to-community assignment CSV
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: NumPy
  role: Reads the pearson.df.npz thresholded Pearson correlation matrix of programs
  repo: https://github.com/MorrissyLab/mosaicMPI
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:c54ec83b0ccde026102e3cfccf8396f48387dcdecd16b84b3b3543d638d58252
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
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# cross-modal-community-composition-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs greedy modularity community detection on the mosaicMPI anchor-program network (program_graph.graphml / pearson.df.npz) to partition integrated programs into communities (C1–C23), prunes single-node communities, and characterizes each community's modality composition (bulk/single-cell/spatial) for the CPTAC GBM landscape.

## When to use

Use when an anchor network of molecular programs has been integrated across datasets/modalities (e.g. from a thresholded Pearson correlation matrix) and you need its community partition and the modality composition of each community.

## When NOT to use

- You have not yet built the anchor network — run program discovery (cNMF) and program integration first; this skill operates on the integrated program graph, not raw expression matrices (e.g. cptac_snRNA_subsampled.txt).
- You need community-representative program feature scores for downstream pathway/marker analysis rather than the partition and its modality composition.

## Inputs

- program_graph.graphml — anchor program network (nodes = programs, edges = program-program associations)
- pearson.df.npz — Pearson correlation matrix of all programs within and between datasets (NumPy-compatible)
- dataset/modality metadata for each program (bulk, single-cell, spatial) to compute community modality composition

## Outputs

- node-to-community assignment CSV with communities labelled C1–C23
- community x modality composition table (e.g. counts of communities spanning 1, 2, or 3 modalities)
- spring-embedded (Fruchterman-Reingold) network figure colored by community assignment

## How to apply

Load the anchor program network from the integration step (program_graph.graphml) and the thresholded Pearson correlation matrix (pearson.df.npz) with pandas/NumPy. Run greedy modularity community detection over the network nodes and edges (as in scikit-learn-style network community tooling). Prune communities containing only a single node so singleton programs do not inflate the partition count. Write the node-to-community assignments as a CSV with communities labelled C1–C23, then render a Fruchterman-Reingold spring-embedded layout colored by community assignment. Validate against the reference result: 23 communities, with 17 spanning all three modalities, 3 spanning two modalities, and the remainder single-modality (Figure 2G composition).

## Related tools

- **mosaicMPI** (Produces the anchor program network being partitioned; community detection and community-based annotation are part of its network-based integration approach) — https://github.com/MorrissyLab/mosaicMPI
- **pandas** (Loads data matrices and writes the node-to-community assignment CSV) — https://github.com/MorrissyLab/mosaicMPI
- **NumPy** (Reads the pearson.df.npz thresholded Pearson correlation matrix of programs) — https://github.com/MorrissyLab/mosaicMPI

## Examples

```
import networkx as nx; G = nx.read_graphml('program_graph.graphml'); comms = nx.community.greedy_modularity_communities(G); comms = [c for c in comms if len(c) > 1]
```

## Evaluation signals

- The partition yields exactly 23 communities after pruning single-node communities.
- Communities are labelled C1–C23 in the output CSV.
- Community-modality composition matches Figure 2G: 17 communities span all three modalities, 3 communities span two modalities, remainder are single-modality.
- The Fruchterman-Reingold layout renders with nodes colored by community assignment and no unassigned (pruned-singleton) communities in the legend.
- Every graph node appears exactly once in the node-to-community CSV (assignments are a partition).

## Limitations

- Mosaic integration can lose relevant information between modalities because not all features are shared, which may affect program-program edges feeding the network.
- Communities containing only a single node must be pruned; without this step the community count will not match the reference partition.
- No software version information is stated for the tools/methods; reproduce with the mosaicMPI repo state matching the paper (DOI 10.1093/nar/gkae442).

## Evidence

- [readme] network-based grouping of programs: [evidence span withheld — anchor status: external_doc]
