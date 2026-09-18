---
name: he-image-feature-extraction
description: Use when a 10x Visium (or similar SRT) dataset ships an H&E histology
  image aligned to spot coordinates, and spatial-domain clustering should exploit
  morphological information to lift clustering accuracy (e.g. MCGAE ARI 0.53→0.64).
when_to_use_negative:
- No H&E image is available or it cannot be aligned to spots — MCGAE is explicitly
  designed to function effectively even in the absence of image information, so use
  image-free mode instead.
- Sections lack ground-truth annotations and the goal is only to verify the image-integration
  ARI gain, which requires annotated data for ARI computation.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ResNet50
  role: Pre-trained convolutional network used to extract morphological image features
    per spot from the H&E histology image
- name: MCGAE
  role: Consumes the ResNet50 image features via its self-attention mechanism, fusing
    them with gene expression and spatial adjacency into a unified embedding
  repo: github.com/yiwen-yang/MCGAE
merged_aliases:
- histology-image-feature-extraction
merged_alias_records:
- alias: histology-image-feature-extraction
  slug: histology-image-feature-extraction
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:4423fe70f43f59e17fd660074b96cbd4b9b4ff0a6ac62b8aa0b7654e16e7d374
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
  - STAGATE
  - SpaGCN
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# H&E image feature extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extract per-spot morphological features from H&E-stained histology images using a pre-trained ResNet50 network, so that MCGAE can fuse image information with gene expression and spatial adjacency into a unified multimodal spatial-domain embedding.

## When to use

Use when a 10x Visium (or similar SRT) dataset ships an H&E histology image aligned to spot coordinates, and spatial-domain clustering should exploit morphological information to lift clustering accuracy (e.g. MCGAE ARI 0.53→0.64).

## When NOT to use

- No H&E image is available or it cannot be aligned to spots — MCGAE is explicitly designed to function effectively even in the absence of image information, so use image-free mode instead.
- Sections lack ground-truth annotations and the goal is only to verify the image-integration ARI gain, which requires annotated data for ARI computation.

## Inputs

- H&E-stained histology image aligned to 10x Visium spot coordinates
- Visium spot spatial coordinates table (for spot-to-patch alignment)
- Pre-trained ResNet50 network (weights as packaged in github.com/yiwen-yang/MCGAE)

## Outputs

- Per-spot morphological image feature matrix (one ResNet50 embedding per Visium spot)
- Downstream MCGAE multimodal embedding with image-fused spatial domain labels per spot (e.g. crlm_spatial_domains.csv)

## How to apply

Load the H&E image shipped with the 10x Visium count matrix and spatial coordinates, crop patches aligned to each spot, and pass them through the pre-trained ResNet50 network used by MCGAE to obtain a morphological feature vector per spot. Note that the article does not specify the ResNet50 version/checkpoint, so record whichever weights the MCGAE repository uses. Feed these features into MCGAE's self-attention mechanism alongside gene expression and the spatial adjacency matrix, generating modality-specific representations that are fused into a unified embedding. If no usable image exists, fall back to MCGAE's image-free mode, which builds multi-view representations from gene expression and spatial adjacency alone. Validate the added value by comparing ARI with and without image features on a section with ground-truth annotations (breast cancer Visium: ~0.53 image-free vs ~0.64 with image, a 20.75% relative lift).

## Related tools

- **ResNet50** (Pre-trained convolutional network used to extract morphological image features per spot from the H&E histology image)
- **MCGAE** (Consumes the ResNet50 image features via its self-attention mechanism, fusing them with gene expression and spatial adjacency into a unified embedding) — github.com/yiwen-yang/MCGAE

## Evaluation signals

- On the Human Breast Cancer 10x Visium dataset, the with-image MCGAE ARI falls within [0.60, 0.68] (expected ~0.64) versus ~0.53 image-free, a ~20.75% relative lift.
- The extracted feature table covers every Visium spot (row count equals spot count) with a feature vector per spot.
- Downstream on CRLM data, region Y (tumor invasion spanning tumor and connective tissue) resolves into 3 sub-clusters in t-SNE, matching Fig. 4 of the paper.
- With-image MCGAE ARI (~0.64) outperforms SpaGCN (~0.56) and STAGATE (~0.57) run on the same breast cancer section.
- The same ResNet50 feature set degrades gracefully: image-free MCGAE still produces valid spatial domains from expression + spatial adjacency alone.

## Limitations

- No version is specified for the pre-trained ResNet50 network used for image feature extraction, so reproducibility depends on the weights packaged in the MCGAE repository.
- Image features add value only when the H&E image is present and aligned; the pipeline must rely on the image-free mode otherwise.
- The downstream MCGAE adjacency-matrix computation is O(n^2), which may rapidly exhaust computational resources on large samples, constraining the scale at which image-fused clustering can run.

## Evidence
