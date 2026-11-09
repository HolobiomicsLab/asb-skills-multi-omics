---
name: umap-embedding-visualization-of-spatial-domains
description: Use when multi-section spatial transcriptomics embeddings (e.g., MCGAE
  unified embeddings from seven Slide-seq hippocampal sections with inter-batch adjacency)
  must be visually verified for coherent 3D spatial domain organization.
when_to_use_negative:
- Datasets too large for the O(n^2) adjacency-matrix computation on spatial positions
  — the article notes this rapidly exhausts computational resources; subgraph sampling
  would be needed first
- Datasets whose batch effects are too complex to be addressed by the distance-between-neighboring-regions
  strategy used in the 3D example
- Expression data with no spatial position information (e.g., dissociated single-cell
  data), where no spatial adjacency structure exists to embed
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MCGAE
  role: Multi-view contrastive graph autoencoder whose unified embeddings are clustered
    into 3D spatial domain labels and projected into 2D with UMAP for visualization
  repo: github.com/yiwen-yang/MCGAE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:afa68d420e59b89e2df65a347bb0cfce8eeec9003746dded3858fb115b588218
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# UMAP embedding visualization of spatial domains

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Projects MCGAE's modality-specific and unified embeddings into 2D with UMAP and overlays 3D spatial domain labels so that cross-section tissue structure (e.g., cord-like hippocampal regions across seven consecutive Slide-seq sections) can be visually assessed.

## When to use

Use when multi-section spatial transcriptomics embeddings (e.g., MCGAE unified embeddings from seven Slide-seq hippocampal sections with inter-batch adjacency) must be visually verified for coherent 3D spatial domain organization.

## When NOT to use

- Datasets too large for the O(n^2) adjacency-matrix computation on spatial positions — the article notes this rapidly exhausts computational resources; subgraph sampling would be needed first
- Datasets whose batch effects are too complex to be addressed by the distance-between-neighboring-regions strategy used in the 3D example
- Expression data with no spatial position information (e.g., dissociated single-cell data), where no spatial adjacency structure exists to embed

## Inputs

- Processed Slide-seq expression matrices and bead spatial coordinates for seven consecutive hippocampal sections (github.com/yiwen-yang/MCGAE)
- Intra-section spatial adjacency graphs built from bead coordinates
- Inter-batch adjacency distances linking neighboring sections
- Trained MCGAE modality-specific and unified embeddings
- Per-spot 3D spatial domain labels from clustering the unified embedding

## Outputs

- 2D UMAP embedding coordinates per spot
- UMAP plot with 3D spatial domain labels overlaid, delineating hippocampal tissue structures
- Per-spot 3D domain label table (one row per spot with section id, coordinates, and 3D domain)

## How to apply

After training the MCGAE multi-view contrastive graph autoencoder on gene expression views plus intra-section spatial adjacency graphs and inter-batch adjacency distance linking neighboring sections, take the unified embedding (and optionally modality-specific embeddings) and reduce it to 2D with UMAP. Overlay the per-spot 3D spatial domain labels (assigned by clustering the unified embedding) onto the UMAP plot so that each domain forms a distinguishable region. Rationale: MCGAE enhances similarity between adjacent slices by calculating the adjacency distance of neighboring batches before reconstructing 3D spatial domains, so a correct run shows spots from adjacent sections with the same 3D domain clustering together in UMAP and delineates cord-like hippocampal tissue structures rather than separating slices by batch.

## Related tools

- **MCGAE** (Multi-view contrastive graph autoencoder whose unified embeddings are clustered into 3D spatial domain labels and projected into 2D with UMAP for visualization) — github.com/yiwen-yang/MCGAE

## Examples

```
import umap; embedding_2d = umap.UMAP(n_components=2).fit_transform(mcgae_unified_embedding)
```

## Evaluation signals

- Each spot in the label table has a section id, spatial coordinates, and a 3D domain label (one row per spot across all seven sections)
- Spots assigned the same 3D domain across adjacent sections co-cluster in the UMAP plot rather than separating by batch
- Cord-like hippocampal tissue structures are visually delineated on the UMAP overlay and different regions are distinguishable
- Modality-specific embeddings, when projected separately, remain consistent with the unified-embedding view
- UMAP projection reduces the unified embedding to exactly 2 dimensions without reordering or dropping spots

## Limitations

- Adjacency matrix computation for spatial position information has O(n^2) complexity, which may rapidly exhaust computational resources on large samples
- Batch effect handling via distance between neighboring regions may not suffice for other complex datasets
- Large-scale datasets require future subgraph sampling or deep multichannel GCN strategies not yet implemented

## Evidence
