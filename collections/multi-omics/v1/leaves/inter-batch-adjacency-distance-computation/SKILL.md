---
name: inter-batch-adjacency-distance-computation
description: Use when jointly analyzing consecutive spatial transcriptomics sections
  (e.g., seven Slide-seq hippocampal slices) where per-section bead coordinates must
  be linked by cross-batch adjacency edges before 3D spatial domain clustering.
when_to_use_negative:
- 'Single-section 2D spatial domain detection: with no consecutive slices there are
  no neighboring batches to link, so only the intra-section adjacency graph is needed.'
- Datasets whose sections are not physically consecutive or reliably aligned — the
  authors warn that handling batch effects via distance between neighboring regions
  may not suffice for other complex datasets.
- 'Very large spot counts: adjacency matrix computation has O(n^2) complexity and
  may rapidly exhaust computational resources, so subgraph sampling alternatives are
  advised instead.'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MCGAE
  role: Multi-view contrastive graph autoencoder that consumes the combined intra-section
    plus inter-batch adjacency to learn modality-specific and unified embeddings and
    assign 3D spatial domain labels across the seven Slide-seq sections
  repo: github.com/yiwen-yang/MCGAE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:307fa2ea6cb8145700c7a7f8cdbe013f26973c077b7ac51e099a2e2c210a6b7e
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
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# inter-batch-adjacency-distance-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the inter-batch adjacency distance between neighboring spatial transcriptomics sections to create cross-batch edges that link consecutive slices (e.g., seven consecutive Slide-seq hippocampal sections), which MCGAE uses to enhance cross-section similarity and reconstruct 3D spatial domains.

## When to use

Use when jointly analyzing consecutive spatial transcriptomics sections (e.g., seven Slide-seq hippocampal slices) where per-section bead coordinates must be linked by cross-batch adjacency edges before 3D spatial domain clustering.

## When NOT to use

- Single-section 2D spatial domain detection: with no consecutive slices there are no neighboring batches to link, so only the intra-section adjacency graph is needed.
- Datasets whose sections are not physically consecutive or reliably aligned — the authors warn that handling batch effects via distance between neighboring regions may not suffice for other complex datasets.
- Very large spot counts: adjacency matrix computation has O(n^2) complexity and may rapidly exhaust computational resources, so subgraph sampling alternatives are advised instead.

## Inputs

- Processed expression matrix for each of the seven consecutive Slide-seq hippocampal sections (one per batch)
- Spatial bead coordinates (x, y) with section/batch id for each section
- Section ordering defining which batches are physically adjacent neighbors

## Outputs

- Inter-batch adjacency distances/edges connecting spots across neighboring sections
- Combined spatial adjacency graph (intra-section plus inter-batch edges) consumed by the MCGAE autoencoder
- Downstream per-spot 3D spatial domain label table (one row per spot with section id, coordinates, and 3D domain) and UMAP embeddings after MCGAE clustering

## How to apply

Load the processed expression matrices and spatial bead coordinates for each of the seven consecutive Slide-seq hippocampal sections from github.com/yiwen-yang/MCGAE, and build the intra-section spatial adjacency graph for every section from its bead coordinates. Then compute the inter-batch adjacency distance between spots of neighboring sections — MCGAE's stated device for enhancing similarity between adjacent slices — and append these cross-batch edges to the per-section adjacency matrices to form the combined spatial structure. This step also serves as the paper's batch-effect handling for the 3D case: distances between neighboring regions tie sections together so spatial domains align across batches. Feed the combined adjacency together with the expression views into the MCGAE multi-view contrastive graph autoencoder, then cluster the unified embedding so every spot across all seven sections receives a 3D spatial domain label. Validate by projecting the learned embeddings with UMAP and checking that cord-like hippocampal tissue structures are delineated and different regions distinguished rather than splitting by batch.

## Related tools

- **MCGAE** (Multi-view contrastive graph autoencoder that consumes the combined intra-section plus inter-batch adjacency to learn modality-specific and unified embeddings and assign 3D spatial domain labels across the seven Slide-seq sections) — github.com/yiwen-yang/MCGAE

## Evaluation signals

- Every spot in sections 2–7 gains at least one inter-batch edge to its neighboring section, confirming adjacent slices are linked in the combined adjacency graph.
- Clustering the MCGAE unified embedding assigns a 3D spatial domain label to every spot across all seven sections with no per-batch islands.
- UMAP overlay of the 3D domain labels delineates cord-like hippocampal tissue structures and distinguishes different hippocampal regions, matching the paper's 3D Slide-seq result.
- The saved output table has one row per spot with section id, coordinates, and 3D domain, and UMAP embeddings are exported alongside it.
- Runtime and memory stay feasible: adjacency computation scales O(n^2) in spot count, so the seven-section spot total must fit the reference workstation (README: 64 GB RAM, RTX 3060).

## Limitations

- Adjacency matrix computation for spatial position information has O(n^2) computational complexity and may rapidly exhaust computational resources as sample counts grow; the authors propose subgraph sampling or deep multichannel GCN strategies for future large-scale datasets.
- The batch-effect strategy of considering distance between neighboring regions was designed for the 3D example and may not suffice for other complex datasets.
- The approach was demonstrated on the consecutive Slide-seq hippocampal 3D example; the repository README provides only hardware specifications (Intel i9-12900K, 64 GB RAM, NVIDIA RTX 3060; Windows/Linux), not per-step parameter defaults.

## Evidence

- [discussion] Adjacency matrix computation is O(n^2), limiting scalability: [evidence span withheld — anchor status: not_found]
