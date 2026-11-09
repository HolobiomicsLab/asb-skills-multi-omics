---
name: spatial-transcriptomics-embedding-integration
description: Use when you already hold per-spot embeddings from multiple spatial transcriptomics
  modalities (expression, spatial adjacency, histology image) sharing the same n_spots
  dimension and need a unified attention-weighted embedding.
when_to_use_negative:
- You only have a single modality (e.g. raw gene expression alone with no adjacency
  or image embeddings to fuse) — build modality-specific embeddings first via the
  upstream contrastive-GCN sub-task
- Spot counts are so large that the O(n^2) adjacency computation exhausts resources
  — the paper notes subgraph sampling or deep multichannel GCN strategies are needed
  for large-scale datasets
- Inputs are already a fused/unified embedding — no re-fusion with Eq. 3-5 attention
  is warranted
tools:
- name: MCGAE
  role: Framework implementing the attention-based fusion of Z^X, Z^A, and Z^morph
    into the unified embedding Z (Eqs. 3-5); code at yiwen-yang/MCGAE
  repo: github.com/yiwen-yang/MCGAE
- name: ResNet50 (pre-trained)
  role: Produces the morphological image embedding Z^morph that is one of the three
    fused modality inputs
provenance:
  source_task_ids:
  - task_008
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:7fa70e1544cd0295fd0279fd07b19e50a84309544b88a5e07fc94c79e32e412b
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
  - ResNet50 (pre-trained)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# spatial-transcriptomics-embedding-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Fuses modality-specific spatial embeddings from MCGAE — expression-view Z^X, adjacency-view Z^A, and morphological image embedding Z^morph — into a unified per-spot embedding Z via a shared-vector attention mechanism with softmax-normalized coefficients (Eqs. 3-5), yielding a robust multimodal representation for spatial domain identification.

## When to use

Use when you already hold per-spot embeddings from multiple spatial transcriptomics modalities (expression, spatial adjacency, histology image) sharing the same n_spots dimension and need a unified attention-weighted embedding.

## When NOT to use

- You only have a single modality (e.g. raw gene expression alone with no adjacency or image embeddings to fuse) — build modality-specific embeddings first via the upstream contrastive-GCN sub-task
- Spot counts are so large that the O(n^2) adjacency computation exhausts resources — the paper notes subgraph sampling or deep multichannel GCN strategies are needed for large-scale datasets
- Inputs are already a fused/unified embedding — no re-fusion with Eq. 3-5 attention is warranted

## Inputs

- Modality-specific spot embedding Z^X (expression view, n_spots x d) from the contrastive-GCN encoder
- Modality-specific spot embedding Z^A (spatial adjacency view, n_spots x d) from the contrastive-GCN encoder
- Morphological image embedding Z^morph (n_spots x d) from pre-trained ResNet50 feature extraction
- Assurance that all three matrices share the same n_spots dimension

## Outputs

- Unified fused embedding Z (n_spots x d matrix)
- Per-spot attention coefficient vector (values in [0, 1], summing to 1 across modalities per spot)
- Named-field NumPy archive fused_embedding_Z.npz containing Z and the coefficients

## How to apply

Load the chained embeddings Z^X and Z^A produced by the upstream contrastive-GCN (Deep Graph Infomax) sub-task plus the image embedding Z^morph from the ResNet50-based sub-task, verifying all three have identical n_spots rows. Compute pairwise similarity scores among the three modality-specific embeddings using the dot-product scoring of Eq. (3) with a shared attention vector. Apply softmax row-wise (Eq. 4) so each spot's coefficients lie in [0, 1] and sum to 1 across modalities — this normalization makes coefficients comparable across spots despite modality scale differences. Form the unified embedding Z as the attention-weighted sum of Z^X, Z^A, and Z^morph per Eq. (5), an (n_spots x d) matrix. Persist both Z and the attention coefficient vector as named fields in fused_embedding_Z.npz. Note that MCGAE degrades gracefully: if histology imagery is unavailable, fusion proceeds on Z^X and Z^A alone since the framework 'is capable of functioning effectively even in the absence of image information.'

## Related tools

- **MCGAE** (Framework implementing the attention-based fusion of Z^X, Z^A, and Z^morph into the unified embedding Z (Eqs. 3-5); code at yiwen-yang/MCGAE) — github.com/yiwen-yang/MCGAE
- **ResNet50 (pre-trained)** (Produces the morphological image embedding Z^morph that is one of the three fused modality inputs)

## Examples

```
import numpy as np; S = np.stack([Zx@Zx.T, Za@Za.T, Zm@Zm.T]); W = np.exp(S)/np.exp(S).sum(-1, keepdims=True); Z = W[0]*Zx + W[1]*Za + W[2]*Zm; np.savez('fused_embedding_Z.npz', Z=Z, W=W)
```

## Evaluation signals

- Every attention coefficient lies in [0, 1]
- For each spot, the modality coefficients sum to exactly 1 (softmax row-wise normalization)
- Fused embedding Z has shape (n_spots, d) with row count equal to the number of spots
- fused_embedding_Z.npz exists and exposes named fields for both Z and the attention coefficients
- Fusion converges without error on hardware like the README's reference setup (RTX 3060 GPU, 64 GB RAM)

## Limitations

- Adjacency matrix computation for spatial position information incurs O(n^2) complexity, which may rapidly exhaust computational resources on large samples
- The batch-effect strategy based on distance between neighboring regions may not suffice for other complex datasets
- No version specified for the pre-trained ResNet50 network used for image feature extraction
- MCGAE works without image information, so Z^morph may be omitted — but the multimodal fusion is then reduced to two views

## Evidence
