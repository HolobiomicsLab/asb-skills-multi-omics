---
name: program-network-community-detection
description: Use when a thresholded program-level correlation/anchor network (e.g.
  GraphML + pearson.df.npz) exists and you need to partition programs across datasets/modalities
  into communities for downstream annotation transfer.
when_to_use_negative:
- You need to discover the programs themselves — program discovery is done upstream
  by cNMF factorization per dataset, not by this community-detection step
- The network has not yet been built or thresholded (no program_graph.graphml / pearson.df.npz
  from the integration step exists)
- The analysis requires subsetting features/genes to a shared intersection — mosaicMPI's
  mosaic integration deliberately avoids feature intersection
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3170
tools:
- name: mosaicMPI
  role: Framework producing the anchor program network and providing the Python/CLI
    environment in which program communities are grouped and interpreted
  repo: https://github.com/MorrissyLab/mosaicMPI
- name: pandas
  role: Loading data matrices and writing the node-to-community assignment CSV
- name: NumPy
  role: Reading the pearson.df.npz thresholded Pearson correlation matrix of all programs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkae442
    title: 'mosaicMPI: a framework for modular data integration across cohorts and
      -omics modalities'
  source_content_hash: sha256:5b3849de8fc83f85194bce3ec694d9d55eb609a620e26a53dacd77550b72e106
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
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# program-network-community-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs greedy modularity community detection on the mosaicMPI anchor-program network (program_graph.graphml + pearson.df.npz) to partition highly connected molecular programs into communities (C1–C23) and characterize their modality composition in the CPTAC GBM integration.

## When to use

Use when a thresholded program-level correlation/anchor network (e.g. GraphML + pearson.df.npz) exists and you need to partition programs across datasets/modalities into communities for downstream annotation transfer.

## When NOT to use

- You need to discover the programs themselves — program discovery is done upstream by cNMF factorization per dataset, not by this community-detection step
- The network has not yet been built or thresholded (no program_graph.graphml / pearson.df.npz from the integration step exists)
- The analysis requires subsetting features/genes to a shared intersection — mosaicMPI's mosaic integration deliberately avoids feature intersection

## Inputs

- program_graph.graphml — anchor program network from the mosaicMPI integration step (nodes = programs, edges = anchor links)
- pearson.df.npz — thresholded Pearson correlation matrix of all programs within and between datasets (NumPy-compatible)
- Program-to-dataset/modality metadata for labeling community composition

## Outputs

- node-to-community assignment table as CSV, with communities labelled C1–C23
- Fruchterman-Reingold spring-embedded network layout colored by community assignment (Figure 2G-style plot)
- Community-modality composition summary (e.g. counts of communities spanning all three modalities)

## How to apply

Load the anchor program network from the prior integration step (program_graph.graphml) and the thresholded Pearson correlation matrix (pearson.df.npz) with pandas/NumPy. Run greedy modularity community detection on the anchor network's nodes and edges (e.g. via networkx community tooling in the mosaicMPI environment). Prune communities containing only a single node, then write the node-to-community assignment table as CSV with communities labelled C1–C23. Render a Fruchterman-Reingold spring-embedded layout colored by community assignment, and cross-tabulate each community's modality composition (in the CPTAC GBM reference result, 17 of 23 communities span all three modalities, 3 are found by two modalities, remainder single-modality). This network grouping is the mosaicMPI step that 'groups similar programs together across resolutions and datasets', enabling community-specific metadata transfer.

## Related tools

- **mosaicMPI** (Framework producing the anchor program network and providing the Python/CLI environment in which program communities are grouped and interpreted) — https://github.com/MorrissyLab/mosaicMPI
- **pandas** (Loading data matrices and writing the node-to-community assignment CSV)
- **NumPy** (Reading the pearson.df.npz thresholded Pearson correlation matrix of all programs)

## Examples

```
import networkx as nx; from networkx.algorithms.community import greedy_modularity_communities; comms = greedy_modularity_communities(nx.read_graphml('program_graph.graphml'))
```

## Evaluation signals

- The partition yields exactly 23 communities labelled C1–C23 after pruning single-node communities
- Community-modality composition matches Figure 2G: 17 communities span all three modalities, 3 communities are found by two modalities, remainder single-modality
- Node-to-community CSV covers all anchor-network nodes with no unassigned programs
- Spring-embedded layout colors every rendered node with its community assignment
- Community assignments are consistent with the input pearson.df.npz edge structure (no community splits across strong anchor edges)

## Limitations

- Communities pruned as single nodes are discarded, so singleton programs are excluded from downstream community-level interpretation
- mosaic integration can lose relevant information between modalities because not all features are shared
- Results depend on the upstream thresholding of the Pearson correlation matrix used to build the anchor network

## Evidence

- [readme] readme network grouping: [evidence span withheld — anchor status: external_doc]
