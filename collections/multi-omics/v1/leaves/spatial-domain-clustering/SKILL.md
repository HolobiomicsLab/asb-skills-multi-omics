---
name: spatial-domain-clustering
description: Use when clustering 10x Visium (or similar SRT) count matrices into spatial
  domains and H&E histology image features, spatial coordinates, or a spatial adjacency
  graph are available to boost clustering accuracy beyond expression-only methods.
when_to_use_negative:
- Input is non-spatial scRNA-seq without spot coordinates — no spatial adjacency matrix
  can be built and the method's multi-view spatial design does not apply
- Sample count is very large — the O(n^2) spatial adjacency matrix computation may
  rapidly exhaust computational resources
- Goal is single-modality expression clustering where spatial relationships are irrelevant
  (expression-only tools like Seurat suffice)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: MCGAE
  role: 'Primary method: multi-view contrastive graph autoencoder producing spatial
    domain clusters, with and without image integration'
  repo: github.com/yiwen-yang/MCGAE
- name: SpaGCN
  role: Baseline method for ARI comparison (image-integrated graph convolutional clustering)
- name: STAGATE
  role: Baseline method for ARI comparison (graph attention autoencoder)
- name: ResNet50
  role: Pre-trained network used to extract morphological image features from H&E
    histology
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:f2a8d7854620c5f796dd98ab76d541232ee683ad47c31e87b1beeb99b2a1d1e0
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spatial-domain-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify spatial domains in spatially-resolved transcriptomics (SRT) data using MCGAE, a multi-view contrastive graph autoencoder that fuses gene expression, spatial adjacency, and (optionally) H&E morphological image features extracted with a pre-trained ResNet50 into a unified multimodal embedding. On the Human Breast Cancer 10x Visium dataset, image integration lifts mean ARI from ~0.53 to ~0.64.

## When to use

Use when clustering 10x Visium (or similar SRT) count matrices into spatial domains and H&E histology image features, spatial coordinates, or a spatial adjacency graph are available to boost clustering accuracy beyond expression-only methods.

## When NOT to use

- Input is non-spatial scRNA-seq without spot coordinates — no spatial adjacency matrix can be built and the method's multi-view spatial design does not apply
- Sample count is very large — the O(n^2) spatial adjacency matrix computation may rapidly exhaust computational resources
- Goal is single-modality expression clustering where spatial relationships are irrelevant (expression-only tools like Seurat suffice)

## Inputs

- 10x Visium gene-expression count matrix (e.g., Human Breast Cancer section)
- Spatial coordinates of spots (for adjacency matrix construction)
- H&E stained histological image (optional; morphological features via pre-trained ResNet50)
- Ground-truth spatial domain annotations for ARI evaluation

## Outputs

- Spatial domain cluster assignments per spot
- Multimodal latent embedding (unified multi-view representation)
- ARI comparison table (e.g., breast_cancer_ari_comparison.csv) for MCGAE-with-image, MCGAE-without-image, SpaGCN, STAGATE

## How to apply

Load the 10x Visium count matrix, spatial coordinates, and H&E image; extract morphological features with a pre-trained ResNet50 network. Train MCGAE in image-free mode first, building multi-view representations from gene expression and the spatial adjacency matrix, then train the image-integrated mode, fusing gene expression, spatial positions, and morphological features through the self-attention mechanism into a unified multimodal embedding. Run SpaGCN and STAGATE on the same section as baselines. Compute the Adjusted Rand Index (ARI) of each method's domain assignments against ground-truth annotations and tabulate results (e.g., breast_cancer_ari_comparison.csv). Rationale: MCGAE's contrastive multi-view design avoids reliance on a single predefined similarity metric and works even without image information, while image fusion enriches the embedding with morphology.

## Related tools

- **MCGAE** (Primary method: multi-view contrastive graph autoencoder producing spatial domain clusters, with and without image integration) — github.com/yiwen-yang/MCGAE
- **SpaGCN** (Baseline method for ARI comparison (image-integrated graph convolutional clustering))
- **STAGATE** (Baseline method for ARI comparison (graph attention autoencoder))
- **ResNet50** (Pre-trained network used to extract morphological image features from H&E histology)

## Evaluation signals

- MCGAE-with-image ARI on the Human Breast Cancer Visium section falls within [0.60, 0.68] (~0.64 expected)
- Relative ARI lift of image-integrated over image-free MCGAE is ~20.75% (0.53 -> 0.64)
- MCGAE baselines comparison: SpaGCN ~0.56 and STAGATE ~0.57 ARI on the same section should be reproduced or exceeded
- Image-free MCGAE mode still produces competitive spatial domains (the paper states it functions effectively without image information)
- Output CSV contains ARI rows for all four conditions (MCGAE ± image, SpaGCN, STAGATE) against ground-truth annotations

## Limitations

- Adjacency matrix computation for spatial position information has O(n^2) complexity, which may rapidly exhaust computational resources with large sample counts
- No version specified for the pre-trained ResNet50 network used for image feature extraction
- Batch-effect handling via distance between neighboring regions may not suffice for other complex datasets
- Scalability to future large-scale datasets will require subgraph sampling or deep multichannel GCN strategies

## Evidence
