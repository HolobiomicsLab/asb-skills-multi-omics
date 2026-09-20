---
name: modality-attention-weight-normalization
description: Use when chained modality-specific spatial representations (expression,
  spatial adjacency, morphological image features) share the same n_spots dimension
  and must be fused into a single unified embedding with per-spot attention weights.
when_to_use_negative:
- Only one modality-specific embedding is available (e.g. MCGAE is run in its no-image
  mode), so no multi-view attention fusion is defined
- The three modality-specific embeddings do not share the same n_spots dimension,
  so per-spot row-wise softmax weighting is undefined
tools:
- name: MCGAE
  role: Provides the self-attention fusion module (Eqs. 3-5) that integrates gene
    expression, spatial position, and image embeddings into a unified embedding Z
  repo: github.com/yiwen-yang/MCGAE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:62e8cd49f189edc90051cee5a7a644040be84edd38908861be2d2df548a71e93
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# modality-attention-weight-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs MCGAE's terminal fusion phase (Eqs. 3-5), in which attention values for the modality-specific embeddings Z^X, Z^A, and Z^morph are computed via a shared attention vector, softmax-normalized row-wise, and used to weight an attention-weighted sum into a unified multimodal embedding Z.

## When to use

Use when chained modality-specific spatial representations (expression, spatial adjacency, morphological image features) share the same n_spots dimension and must be fused into a single unified embedding with per-spot attention weights.

## When NOT to use

- Only one modality-specific embedding is available (e.g. MCGAE is run in its no-image mode), so no multi-view attention fusion is defined
- The three modality-specific embeddings do not share the same n_spots dimension, so per-spot row-wise softmax weighting is undefined

## Inputs

- Modality-specific embedding Z^X (n_spots x d) from the contrastive-GCN expression branch
- Modality-specific embedding Z^A (n_spots x d) from the contrastive-GCN spatial adjacency branch
- Morphological image embedding Z^morph (n_spots x d) from the image-embedding sub-task

## Outputs

- Unified multimodal embedding Z (n_spots x d), stored in fused_embedding_Z.npz
- Per-spot attention coefficient vector (normalized, in [0, 1], summing to 1 across modalities), stored in fused_embedding_Z.npz

## How to apply

Load the chained modality-specific embeddings Z^X and Z^A from the upstream contrastive-GCN sub-task together with the morphological image embedding Z^morph from the image-embedding sub-task, verifying all three share the same n_spots dimension. Compute pairwise similarity scores among the three modality-specific embeddings using the dot-product scoring of Eq. (3), applied via a shared attention vector. Apply the softmax function of Eq. (4) row-wise so each spot's attention coefficients lie in [0, 1] and sum to 1 across modalities, making coefficients comparable across spots. Form the unified embedding Z as the attention-weighted sum of Z^X, Z^A, and Z^morph per Eq. (5), yielding an (n_spots x d) matrix, and persist both Z and the attention coefficient vector under named fields in fused_embedding_Z.npz.

## Related tools

- **MCGAE** (Provides the self-attention fusion module (Eqs. 3-5) that integrates gene expression, spatial position, and image embeddings into a unified embedding Z) — github.com/yiwen-yang/MCGAE

## Evaluation signals

- Assert every attention coefficient is in [0, 1]
- Assert each spot's attention coefficients sum to 1 across modalities
- Confirm the fused embedding Z has shape (n_spots, d)
- Confirm the three input embeddings share the same n_spots dimension before fusion
- Confirm fused_embedding_Z.npz contains both the fused embedding Z and the attention coefficient vector under named fields

## Limitations

- Adjacency matrix computation for spatial position information has O(n^2) complexity, which may exhaust computational resources with large sample counts
- The model depends on a pre-trained ResNet50 network for image feature extraction whose version is not specified

## Evidence

- [methods] MCGAE computes attention values for each modality-specific embedding via a shared attention vector and normalizes them with a softmax function: [evidence span withheld — anchor status: not_found]
