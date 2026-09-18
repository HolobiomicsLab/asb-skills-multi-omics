---
name: multimodal-gene-expression-image-integration
description: Use when analyzing 10x Visium-style spatial transcriptomics data (count
  matrix + spatial coordinates + H&E image) and you want to test whether morphological
  image features improve spatial domain clustering (ARI) over expression-only baselines.
when_to_use_negative:
- No H&E histological image is available for the section — MCGAE is designed to function
  effectively even without image information, but the image-integration lift cannot
  be evaluated
- The dataset lacks ground-truth spatial domain annotations, so ARI-based verification
  of the expected lift (~0.53 to ~0.64, 20.75%) cannot be computed
- Very large sample counts where the O(n^2) adjacency-matrix computation for spatial
  position information would exhaust computational resources
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_6398
- http://edamontology.org/topic_3360
tools:
- name: MCGAE
  role: Multi-view contrastive graph autoencoder that fuses gene expression, spatial
    positions, and ResNet50 image features via self-attention into the multimodal
    embedding
  repo: github.com/yiwen-yang/MCGAE
- name: ResNet50
  role: Pre-trained network used to extract morphological features from the H&E stained
    histological image
- name: SpaGCN
  role: Baseline method integrating image data into a neighborhood graph with graph
    convolutional networks, used for ARI comparison
- name: STAGATE
  role: Graph-attention baseline for spatial domain identification used for ARI comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:17ee85fec1cff332f7302ad5e1952e93151502581ce937a4c0447c0900c4dd8a
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
  - http://edamontology.org/topic_6398
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# multimodal-gene-expression-image-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Integrates H&E histological image features (extracted via a pre-trained ResNet50) with gene expression and spatial adjacency information in MCGAE's self-attention mechanism to improve spatial domain clustering, e.g. lifting ARI from ~0.53 to ~0.64 (a 20.75% gain) on the Human Breast Cancer 10x Visium dataset.

## When to use

Use when analyzing 10x Visium-style spatial transcriptomics data (count matrix + spatial coordinates + H&E image) and you want to test whether morphological image features improve spatial domain clustering (ARI) over expression-only baselines.

## When NOT to use

- No H&E histological image is available for the section — MCGAE is designed to function effectively even without image information, but the image-integration lift cannot be evaluated
- The dataset lacks ground-truth spatial domain annotations, so ARI-based verification of the expected lift (~0.53 to ~0.64, 20.75%) cannot be computed
- Very large sample counts where the O(n^2) adjacency-matrix computation for spatial position information would exhaust computational resources

## Inputs

- 10x Visium gene expression count matrix (e.g. Human Breast Cancer section)
- Spatial coordinates / spatial adjacency matrix
- H&E stained histological image
- Ground-truth spatial domain annotations for ARI evaluation

## Outputs

- Multimodal unified embedding per spatial spot
- Spatial domain cluster assignments per spot
- ARI comparison table (breast_cancer_ari_comparison.csv) with MCGAE-with-image, MCGAE-without-image, SpaGCN, and STAGATE values

## How to apply

First load the 10x Visium count matrix, spatial coordinates, and H&E histology image, and extract morphological features with a pre-trained ResNet50 network. Train MCGAE in image-free mode, building multi-view representations from gene expression and the spatial adjacency matrix, to establish a baseline (ARI ~0.53 on the Human Breast Cancer section). Then retrain with image integration, fusing gene expression, spatial positions, and morphological features through the self-attention mechanism into a unified multimodal embedding (expected ARI ~0.64). Run SpaGCN (~0.56) and STAGATE (~0.57) baselines on the same section and compute the Adjusted Rand Index of each method's domain assignments against ground-truth annotations. Verify the with-image ARI falls within [0.60, 0.68] and the relative lift over the image-free condition approximates 20.75%.

## Related tools

- **MCGAE** (Multi-view contrastive graph autoencoder that fuses gene expression, spatial positions, and ResNet50 image features via self-attention into the multimodal embedding) — github.com/yiwen-yang/MCGAE
- **ResNet50** (Pre-trained network used to extract morphological features from the H&E stained histological image)
- **SpaGCN** (Baseline method integrating image data into a neighborhood graph with graph convolutional networks, used for ARI comparison)
- **STAGATE** (Graph-attention baseline for spatial domain identification used for ARI comparison)

## Examples

```
python train.py --data human_breast_cancer --mode image --image_features resnet50 --adj spatial --out breast_cancer_ari_comparison.csv
```

## Evaluation signals

- MCGAE-with-image ARI on the Human Breast Cancer Visium section falls within [0.60, 0.68] (expected ~0.64)
- Relative ARI lift over the image-free MCGAE condition (~0.53) approximates 20.75%
- Baselines tabulated in breast_cancer_ari_comparison.csv land near SpaGCN ~0.56 and STAGATE ~0.57, below MCGAE-with-image
- Multimodal embedding is produced by fusing gene expression, spatial positions, and morphological image features through the self-attention mechanism (check modality-specific representations were generated and integrated)

## Limitations

- Adjacency matrix computation for spatial position information incurs O(n^2) computational complexity, which may rapidly exhaust computational resources with large sample counts
- No version specified for the pre-trained ResNet50 network used for image feature extraction
- The batch-effect handling strategy (distance between neighboring regions) may not suffice for other complex datasets; large-scale data would need subgraph sampling or deep multichannel GCN strategies

## Evidence

- [results] Integrating H&E stained images with MCGAE on the Human Breast Cancer dataset increased the average ARI from 0.53 to 0.64, a 20.75% improvement: [evidence span withheld — anchor status: not_found]
- [methods] extract morphological image features using the pre-trained ResNet50 network: [evidence span withheld — anchor status: not_found]
- [discussion] the computation of the adjacency matrix for spatial position information incurs a computational complexity of $O({n}^2)$: [evidence span withheld — anchor status: not_found]
