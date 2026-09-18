---
name: multiview-representation-fusion
description: Use when two or more modality-specific spatial embeddings (expression,
  spatial adjacency, morphological image) sharing the same n_spots dimension must
  be combined into one (n_spots x d) unified embedding with learned per-spot attention
  weights.
when_to_use_negative:
- Input is raw spatial transcriptomics count data rather than learned modality-specific
  embeddings — the upstream contrastive-GCN encoders (producing Z^X and Z^A) and the
  image-embedding sub-task must run first.
- The modality-specific embeddings do not share the same n_spots dimension, so per-spot
  weighted fusion and validation of coefficient sums cannot be performed.
- Only a single view is available and no spatial adjacency or image modality exists
  — there is nothing to fuse and the attention mechanism is unnecessary.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: MCGAE
  role: Framework whose terminal fusion phase implements the Eqs. (3-5) attention-weighted
    multi-view fusion of Z^X, Z^A, and Z^morph into the unified embedding Z
  repo: github.com/yiwen-yang/MCGAE
- name: ResNet50
  role: Pre-trained network producing the morphological image embedding Z^morph that
    serves as the third fused view
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:8e88054009cce50ba829175885b28be5a56fcddba41d28b02ccb8afd23a25aef
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
  - ResNet50
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# multiview-representation-fusion

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Attention-weighted fusion of modality-specific spatial embeddings (gene expression Z^X, spatial adjacency Z^A, and morphological image Z^morph) into a single unified embedding Z using dot-product similarity scoring and a softmax-normalized self-attention mechanism, as implemented in the terminal fusion phase of MCGAE.

## When to use

Use when two or more modality-specific spatial embeddings (expression, spatial adjacency, morphological image) sharing the same n_spots dimension must be combined into one (n_spots x d) unified embedding with learned per-spot attention weights.

## When NOT to use

- Input is raw spatial transcriptomics count data rather than learned modality-specific embeddings — the upstream contrastive-GCN encoders (producing Z^X and Z^A) and the image-embedding sub-task must run first.
- The modality-specific embeddings do not share the same n_spots dimension, so per-spot weighted fusion and validation of coefficient sums cannot be performed.
- Only a single view is available and no spatial adjacency or image modality exists — there is nothing to fuse and the attention mechanism is unnecessary.

## Inputs

- Z^X: modality-specific gene expression embedding (n_spots x d) from the upstream contrastive-GCN sub-task
- Z^A: modality-specific spatial adjacency embedding (n_spots x d) from the upstream contrastive-GCN sub-task
- Z^morph: morphological image embedding (n_spots x d) from ResNet50-based image feature extraction

## Outputs

- Unified multimodal embedding Z of shape (n_spots, d), stored in fused_embedding_Z.npz
- Attention coefficient vector (per-spot, one coefficient per modality, in [0, 1], summing to 1), stored alongside Z in fused_embedding_Z.npz

## How to apply

Load the chained embeddings Z^X and Z^A from the upstream contrastive-GCN sub-task and Z^morph from the image-embedding sub-task (ResNet50-derived features), first verifying all three share the same n_spots dimension. Compute pairwise similarity scores among the three modality-specific embeddings using the dot-product scoring of Eq. (3). Apply softmax row-wise (Eq. 4) so each spot's attention coefficients lie in [0, 1] and sum to 1 across modalities — the softmax normalization is what makes coefficients comparable across spots. Form the unified embedding Z as the attention-weighted sum of Z^X, Z^A, and Z^morph per Eq. (5). Persist Z and the attention coefficient vector under named fields in fused_embedding_Z.npz, then validate the coefficient and shape constraints.

## Related tools

- **MCGAE** (Framework whose terminal fusion phase implements the Eqs. (3-5) attention-weighted multi-view fusion of Z^X, Z^A, and Z^morph into the unified embedding Z) — github.com/yiwen-yang/MCGAE
- **ResNet50** (Pre-trained network producing the morphological image embedding Z^morph that serves as the third fused view)

## Examples

```
from scipy.special import softmax; import numpy as np; S = np.stack([zX@zX, zX@zA, zX@zM], 1); a = softmax(S, 1); Z = a[:,0:1]*zX + a[:,1:2]*zA + a[:,2:3]*zM; np.savez('fused_embedding_Z.npz', Z=Z, alpha=a)
```

## Evaluation signals

- Every attention coefficient lies in [0, 1] after the softmax normalization of Eq. (4).
- For each spot, the three modality attention coefficients sum to exactly 1.
- The fused embedding Z has shape (n_spots, d) and equals the attention-weighted sum of Z^X, Z^A, and Z^morph per Eq. (5).
- Recomputing Z from the stored attention coefficients in fused_embedding_Z.npz reproduces the fused matrix bit-for-bit.
- Coefficients are comparable across spots (softmax over the same shared attention vector), so no spot shows degenerate all-zero or single-modality-saturated weights without cause.

## Limitations

- The computation of the adjacency matrix for spatial position information incurs O(n^2) computational complexity, which may rapidly exhaust computational resources with large sample counts, limiting scalability of the upstream views feeding this fusion.
- No version is specified for the pre-trained ResNet50 network used to extract morphological image features, so Z^morph may vary between reproductions.
- The fusion assumes the shared attention vector is applied consistently; coefficients are only comparable across spots because of the softmax normalization step.

## Evidence

- [other] MCGAE computes attention values for each modality-specific embedding via a shared attention vector and normalizes them with a softmax function, producing comparable coefficients across spots for the weighted fusion.: [evidence span withheld — anchor status: not_found]
- [other] Apply the softmax function of Eq. (4) row-wise to convert the similarity scores into per-spot attention coefficients that lie in [0, 1] and sum to 1 across modalities.: [evidence span withheld — anchor status: not_found]
