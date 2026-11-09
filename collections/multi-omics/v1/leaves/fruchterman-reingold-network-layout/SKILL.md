---
name: fruchterman-reingold-network-layout
description: Use when an integrated program network (e.g. program_graph.graphml built
  from a thresholded Pearson correlation matrix) needs a community-colored visualization
  to inspect modality composition of program communities.
when_to_use_negative:
- Input is a raw expression matrix or h5ad dataset prior to program discovery and
  integration — layout applies only to the already-integrated program network
- The goal is quantitative community detection or modality-composition statistics
  rather than visualization of the program landscape
- The network has not been thresholded from the Pearson correlation matrix, so edges
  are weak/noisy and the spring-embedded layout will be uninterpretable
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: mosaicMPI
  role: Produces the integrated anchor program network and program communities that
    the Fruchterman-Reingold layout visualizes
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: NumPy
  role: Loads the pearson.df.npz thresholded Pearson correlation matrix used to build/verify
    the network
- name: pandas
  role: Reads and writes the node-to-community assignment table as CSV and handles
    program metadata (modality labels)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:ee5f51688e84a262b8f883278d93e2b41572cd0147fb590663c52627d19310de
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# fruchterman-reingold-network-layout

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Renders the mosaicMPI integrated anchor program network as a Fruchterman-Reingold spring-embedded graph, with nodes colored by community assignment (C1-C23) from greedy modularity detection of the CPTAC GBM integration.

## When to use

Use when an integrated program network (e.g. program_graph.graphml built from a thresholded Pearson correlation matrix) needs a community-colored visualization to inspect modality composition of program communities.

## When NOT to use

- Input is a raw expression matrix or h5ad dataset prior to program discovery and integration — layout applies only to the already-integrated program network
- The goal is quantitative community detection or modality-composition statistics rather than visualization of the program landscape
- The network has not been thresholded from the Pearson correlation matrix, so edges are weak/noisy and the spring-embedded layout will be uninterpretable

## Inputs

- program_graph.graphml — anchor program network of integrated cNMF/program nodes and edges (from the mosaicMPI integration step)
- pearson.df.npz — thresholded Pearson correlation matrix of all programs within and between datasets (NumPy-compatible)
- node metadata table mapping programs to source dataset and modality (bulk, single-cell, spatial/proteomics)

## Outputs

- CSV node-to-community assignment table with communities labelled C1-C23 (single-node communities pruned)
- Fruchterman-Reingold spring-embedded network figure colored by community assignment (Figure 2G-style landscape)

## How to apply

Load the anchor program network (program_graph.graphml from the prior integration step) and the thresholded Pearson correlation matrix (pearson.df.npz) using pandas/NumPy. Run greedy modularity community detection on the anchor network's nodes and edges, then prune communities containing only a single node. Write the node-to-community assignment table as a CSV with communities labelled C1-C23. Render the program network with a Fruchterman-Reingold spring-embedded layout, coloring nodes by community, so the cross-modality structure (17 of 23 communities spanning all three modalities, 3 found by two modalities) is visually inspectable against Figure 2G of the article.

## Related tools

- **mosaicMPI** (Produces the integrated anchor program network and program communities that the Fruchterman-Reingold layout visualizes) — https://github.com/MorrissyLab/mosaicMPI
- **NumPy** (Loads the pearson.df.npz thresholded Pearson correlation matrix used to build/verify the network)
- **pandas** (Reads and writes the node-to-community assignment table as CSV and handles program metadata (modality labels))

## Examples

```
import networkx as nx; G = nx.read_graphml('program_graph.graphml'); pos = nx.spring_layout(G, seed=42); nx.draw_networkx_nodes(G, pos, node_color=[comm[n] for n in G.nodes])
```

## Evaluation signals

- The partition yields exactly 23 communities after pruning, labelled C1-C23
- Community-modality composition matches the article: 17 communities contain programs from all three modalities, 3 communities are found by two modalities, remainder single-modality
- No community in the output CSV contains a single node (single-node communities pruned)
- Every node in program_graph.graphml appears exactly once in the node-to-community CSV
- The rendered spring layout shows communities as spatially separated clusters consistent with the Figure 2G mosaicMPI landscape

## Limitations

- Greedy modularity community detection is heuristic, so results may vary with tie-breaking/seed; validation is against the reported 23-community composition rather than an exact-match guarantee
- A potential concern with mosaic integration is loss of relevant information between modalities given that not all features are shared, which can affect community composition being visualized
- No software version numbers are stated in the article's methods, so reproducing the exact layout may depend on library versions

## Evidence

- [other] Render a Fruchterman-Reingold spring-embedded layout of the program network colored by community assignment: [evidence span withheld — anchor status: not_found]
- [readme] README on network-based grouping: [evidence span withheld — anchor status: external_doc]
- [results] Community detection yields 23 communities: [evidence span withheld — anchor status: not_found]
