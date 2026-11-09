---
name: 3d-spatial-domain-reconstruction-across-consecutive-sections
description: Use when you have spatial transcriptomics data from multiple consecutive
  sections (section id, expression matrix, spatial coordinates) and need a shared
  3D domain label per spot that delineates tissue structures across batches.
when_to_use_negative:
- Single-section data with no neighboring slices — cross-batch adjacency cannot be
  computed and per-section spatial clustering methods are appropriate instead
- Datasets where spatial image/morphology is the primary modality and section order
  is undefined, since the neighboring-batch distance strategy was designed for consecutive
  slices
- Very large sample counts — adjacency matrix computation is O(n^2) and may exhaust
  computational resources without subgraph sampling
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MCGAE
  role: Multi-view contrastive graph autoencoder trained on expression views and combined
    spatial adjacency (intra-section graphs plus inter-batch adjacency distance) to
    produce modality-specific and unified embeddings and 3D domain labels
  repo: github.com/yiwen-yang/MCGAE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:7d7acc508a5ba140fe5eec39f2584cf842767d48f1837f09168cc90cdcb77b9a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/yiwen-yang/MCGAE
  tools:
  - MCGAE
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# 3d-spatial-domain-reconstruction-across-consecutive-sections

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstruct 3D spatial domains across consecutive tissue sections (e.g., seven Slide-seq hippocampal slices) by training MCGAE, a multi-view contrastive graph autoencoder, on per-section spatial adjacency graphs plus computed adjacency distances between neighboring sections, then clustering the unified embedding per spot/section.

## When to use

Use when you have spatial transcriptomics data from multiple consecutive sections (section id, expression matrix, spatial coordinates) and need a shared 3D domain label per spot that delineates tissue structures across batches.

## When NOT to use

- Single-section data with no neighboring slices — cross-batch adjacency cannot be computed and per-section spatial clustering methods are appropriate instead
- Datasets where spatial image/morphology is the primary modality and section order is undefined, since the neighboring-batch distance strategy was designed for consecutive slices
- Very large sample counts — adjacency matrix computation is O(n^2) and may exhaust computational resources without subgraph sampling

## Inputs

- Processed expression matrices for consecutive sections (e.g., seven Slide-seq hippocampal sections)
- Spatial bead/spot coordinates per section (section id, x/y coordinates)
- Section order / neighbor relationships between consecutive sections

## Outputs

- Per-spot 3D spatial domain label table (one row per spot with section id, coordinates, and 3D domain)
- UMAP projections of the learned unified embeddings overlaid with 3D domain labels

## How to apply

Load the processed expression matrices and spatial bead coordinates for each section (e.g., seven consecutive Slide-seq hippocampal sections). Build an intra-section spatial adjacency graph from bead coordinates for every section. Compute the inter-batch adjacency distance between neighboring sections and add adjacent edges between neighboring slices — this is the key step that enhances similarity between adjacent sections and enables true 3D reconstruction rather than independent per-slice clustering. Train the MCGAE multi-view contrastive graph autoencoder on the expression views plus the combined spatial adjacency structure to obtain modality-specific and unified embeddings; MCGAE works even in the absence of image information. Cluster the unified embedding to assign every spot a 3D spatial domain label, then project embeddings into 2D with UMAP and overlay the 3D domain labels to verify that cord-like hippocampal tissue structures are delineated and different regions separated on the UMAP.

## Related tools

- **MCGAE** (Multi-view contrastive graph autoencoder trained on expression views and combined spatial adjacency (intra-section graphs plus inter-batch adjacency distance) to produce modality-specific and unified embeddings and 3D domain labels) — github.com/yiwen-yang/MCGAE

## Evaluation signals

- Per-spot 3D domain table contains a domain label, section id, and coordinates for every spot across all seven sections (no dropped spots)
- UMMAP of the unified embedding shows cord-like hippocampal structures delineated and distinct regions separated (matching expected hippocampal anatomy)
- Adjacent sections receive consistent domain labels along their shared boundaries, indicating the inter-batch adjacency edges actually coupled neighboring slices
- 3D domain assignments remain accurate when the multi-view training converges (modality-specific and unified embeddings both learned without collapse)
- Reconstruction distinguishes different hippocampal regions on the UMAP plot rather than clustering purely by expression

## Limitations

- Adjacency matrix computation for spatial position information incurs O(n^2) computational complexity, which may rapidly exhaust computational resources with large sample counts
- Handling batch effects by considering the distance between neighboring regions may not suffice for other complex datasets
- Large-scale datasets require future strategies such as subgraph sampling or deep multichannel GCN approaches
- Hardware reported for the reference run: Intel Core i9-12900K, 64 GB RAM, NVIDIA GeForce RTX 3060 (Windows or Linux)

## Evidence

- [intro] 3d-domain-reconstruction-via-neighbor-batch-adjacency: "MCGAE enhances the similarity between adjacent slices by calculating the adjacency distance of neighboring batches, ultimately reconstructing 3D spatial domains."
- [abstract] unified-embedding-generation: "it generates modality-specific spatial representations and integrates them into a unified embedding."
- [readme] framework-scope: "MCGAE is a novel deep computational framework designed for comprehensive analysis of spatial transcriptomics data across platforms, integrating morphological images."
