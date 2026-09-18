---
name: softmax-self-attention-fusion-of-multimodal-embeddings
description: Use when you already hold n_spots-dimensional modality-specific embeddings
  (expression, spatial adjacency, and/or morphological image features) for the same
  spots and need to merge them into one multimodal representation with learned, normalized
  per-spot attention weights.
when_to_use_negative:
- Inputs are raw gene-expression counts or H&E images rather than pre-trained modality-specific
  embeddings — the upstream contrastive-GCN and image-embedding sub-tasks must run
  first.
- You need a single-modality spatial clustering baseline (e.g., Seurat-style expression-only
  reduction); fusion with attention weights over multiple views is unnecessary.
- Spots across modalities do not share a common n_spots alignment — softmax-weighted
  summing of misaligned embeddings is invalid.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MCGAE
  role: Reference implementation of the self-attention fusion of Z^X, Z^A, and Z^morph
    (Eqs. 3-5) producing the unified embedding
  repo: github.com/yiwen-yang/MCGAE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:3e2a66e00737f8480795026d768a01125fa3fcf7b3a74575f25baa2afd51ccf8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/yiwen-yang/MCGAE
  repo_url: https://github.com/yiwen-yang/MCGAE
  tools:
  - MCGAE
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# softmax-self-attention-fusion-of-multimodal-embeddings

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Terminal fusion phase of MCGAE that combines modality-specific embeddings — the expression embedding Z^X, the spatial-adjacency embedding Z^A from the contrastive-GCN, and the ResNet50-derived morphological embedding Z^morph — into a unified (n_spots x d) embedding Z using a shared-attention vector, dot-product scores (Eq. 3), row-wise softmax (Eq. 4), and an attention-weighted sum (Eq. 5).

## When to use

Use when you already hold n_spots-dimensional modality-specific embeddings (expression, spatial adjacency, and/or morphological image features) for the same spots and need to merge them into one multimodal representation with learned, normalized per-spot attention weights.

## When NOT to use

- Inputs are raw gene-expression counts or H&E images rather than pre-trained modality-specific embeddings — the upstream contrastive-GCN and image-embedding sub-tasks must run first.
- You need a single-modality spatial clustering baseline (e.g., Seurat-style expression-only reduction); fusion with attention weights over multiple views is unnecessary.
- Spots across modalities do not share a common n_spots alignment — softmax-weighted summing of misaligned embeddings is invalid.

## Inputs

- Modality-specific embedding Z^X (gene expression view) from the contrastive-GCN sub-task
- Modality-specific embedding Z^A (spatial adjacency view) from the contrastive-GCN sub-task
- Morphological image embedding Z^morph (e.g., pre-trained ResNet50 features)
- n_spots dimension check across all three embeddings

## Outputs

- Unified multimodal embedding Z of shape (n_spots, d)
- Per-spot attention coefficient vector with coefficients in [0, 1] summing to 1 across modalities
- fused_embedding_Z.npz with named fields for Z and the attention coefficients

## How to apply

Load the chained embeddings Z^X and Z^A from the upstream contrastive-GCN sub-task plus Z^morph from the image-embedding sub-task, verifying all three share the same n_spots dimension. Compute pairwise similarity scores among the three modality-specific embeddings via the dot-product scoring of Eq. (3), applied through a shared attention vector so coefficients are comparable across spots. Apply the softmax of Eq. (4) row-wise so each spot's attention coefficients lie in [0, 1] and sum to 1 across modalities. Form the unified embedding Z as the attention-weighted sum of Z^X, Z^A, and Z^morph per Eq. (5); persist Z and the attention coefficients in fused_embedding_Z.npz. If image data are absent, MCGAE is designed to still function effectively, so the fusion degrades gracefully to the available views.

## Related tools

- **MCGAE** (Reference implementation of the self-attention fusion of Z^X, Z^A, and Z^morph (Eqs. 3-5) producing the unified embedding) — github.com/yiwen-yang/MCGAE

## Examples

```
import numpy as np; sim = np.stack([np.sum(v['attn_vec']*e, axis=1) for e in (Z_X, Z_A, Z_morph)], axis=1); c = np.exp(sim - sim.max(1, keepdims=True)); c /= c.sum(1, keepdims=True); Z = c[:,0:1]*Z_X + c[:,1:2]*Z_A + c[:,2:3]*Z_morph; np.savez('fused_embedding_Z.npz', Z=Z, attn=c)
```

## Evaluation signals

- Every attention coefficient lies in [0, 1].
- For each spot, the attention coefficients across modalities sum to 1 (softmax normalization of Eq. 4).
- Fused embedding Z has shape (n_spots, d), matching the input embedding dimensions.
- fused_embedding_Z.npz contains both the fused Z and the attention coefficient vector under named fields.
- Per-spot Z values reconstruct exactly as the attention-weighted sum of Z^X, Z^A, and Z^morph (Eq. 5 round-trip check).

## Limitations

- Upstream adjacency-matrix computation has O(n^2) complexity, limiting scalability of the embeddings fed into fusion to large spot counts.
- The pre-trained ResNet50 network used for image feature extraction (source of Z^morph) is specified without a version, so morphological embeddings may not be reproducible across environments.
- MCGAE is designed to function even without image information, so attention over a missing modality must be handled (degrade to Z^X and Z^A only).

## Evidence
