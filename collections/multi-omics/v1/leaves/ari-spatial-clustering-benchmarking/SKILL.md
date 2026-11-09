---
name: ari-spatial-clustering-benchmarking
description: Use when evaluating spatial domain detection on 10x Visium H&E-stained
  sections (e.g., Human Breast Cancer) where you must quantify ARI gain from adding
  image features and compare algorithms by ARI.
when_to_use_negative:
- Input lacks an H&E histology image or spatial coordinates, so no image-integrated
  or spatial-adjacency multi-view embedding can be built
- The goal is general expression-only clustering (e.g., non-spatial Seurat-style clustering),
  where spatial relationships are intentionally ignored
- Dataset is very large, given MCGAE's O(n^2) adjacency matrix computation may exhaust
  computational resources
edam_operation: http://edamontology.org/operation_3432
tools:
- name: MCGAE
  role: Primary spatial domain clustering framework run in both image-free and image-integrated
    modes
  repo: github.com/yiwen-yang/MCGAE
- name: SpaGCN
  role: Baseline integrating image data into a neighborhood graph using graph convolutional
    networks
- name: STAGATE
  role: Baseline using graph-based and attention mechanisms for spatial domain identification
- name: ResNet50
  role: Pre-trained network used to extract morphological image features from the
    H&E stained section
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:a1ffc611a75bf7fd28ad37a97f4489a8142b03260be13de280d0f48c76bce99e
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
  - ResNet50
  - STAGATE
  - SpaGCN
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# ari-spatial-clustering-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark spatial-domain clustering accuracy of MCGAE with and without H&E histological image integration on 10x Visium data, using Adjusted Rand Index (ARI) against ground-truth annotations and comparing against baselines like SpaGCN and STAGATE.

## When to use

Use when evaluating spatial domain detection on 10x Visium H&E-stained sections (e.g., Human Breast Cancer) where you must quantify ARI gain from adding image features and compare algorithms by ARI.

## When NOT to use

- Input lacks an H&E histology image or spatial coordinates, so no image-integrated or spatial-adjacency multi-view embedding can be built
- The goal is general expression-only clustering (e.g., non-spatial Seurat-style clustering), where spatial relationships are intentionally ignored
- Dataset is very large, given MCGAE's O(n^2) adjacency matrix computation may exhaust computational resources

## Inputs

- 10x Visium Human Breast Cancer count matrix (gene expression)
- 10x Visium spatial coordinates
- H&E stained histology image
- Pre-trained ResNet50 network for image feature extraction
- Ground-truth spatial domain annotations

## Outputs

- breast_cancer_ari_comparison.csv (ARI table for MCGAE-without-image, MCGAE-with-image, SpaGCN, STAGATE)
- Spatial domain cluster assignments per method
- Multimodal (image-integrated) MCGAE embedding

## How to apply

Load the 10x Visium Human Breast Cancer count matrix, spatial coordinates, and H&E image, and extract morphological features using a pre-trained ResNet50 network. Train MCGAE in image-free mode (multi-view representations from gene expression and the spatial adjacency matrix), then retrain with image integration, fusing gene expression, spatial positions, and morphological features through the self-attention mechanism into a unified multimodal embedding. Run SpaGCN and STAGATE on the same breast cancer section as baselines, since SpaGCN integrates image data into a neighborhood graph using graph convolutional networks and STAGATE uses graph-based attention mechanisms. Compute the Adjusted Rand Index of each method's spatial domain assignments against the ground-truth annotations, tabulate MCGAE-without-image (~0.53), MCGAE-with-image (~0.64), SpaGCN (~0.56), STAGATE (~0.57), and save as breast_cancer_ari_comparison.csv.

## Related tools

- **MCGAE** (Primary spatial domain clustering framework run in both image-free and image-integrated modes) — github.com/yiwen-yang/MCGAE
- **SpaGCN** (Baseline integrating image data into a neighborhood graph using graph convolutional networks)
- **STAGATE** (Baseline using graph-based and attention mechanisms for spatial domain identification)
- **ResNet50** (Pre-trained network used to extract morphological image features from the H&E stained section)

## Evaluation signals

- MCGAE-with-image ARI on the Human Breast Cancer Visium section falls within [0.60, 0.68] (expected ~0.64)
- Relative ARI lift over the image-free condition (~0.53) reproduces the ~20.75% improvement
- ARI values are computed against ground-truth annotations, not pseudo-labels
- Baseline ARIs are in expected ranges: SpaGCN ~0.56 and STAGATE ~0.57 on the same section
- Results are tabulated and saved as breast_cancer_ari_comparison.csv with all four methods compared

## Limitations

- Adjacency matrix computation for spatial position information has O(n^2) computational complexity, which may rapidly exhaust computational resources on large samples
- Batch effect handling based on distance between neighboring regions may not suffice for other complex datasets
- No version specified for the pre-trained ResNet50 network used for image feature extraction

## Evidence

- [readme] readme workflow: "MCGAE is a novel deep computational framework designed for comprehensive analysis of spatial transcriptomics data across platforms, integrating morphological images."
- [intro] self-attention multimodal representation: "MCGAE generates a multimodal, multi-view biological representation that is robust to noise"
- [discussion] ResNet50 feature extraction: "The model also utilizes a pre-trained ResNet50 network for image feature extraction, enhancing its capability to handle complex tumor data."
- [intro] SpaGCN baseline description: "SpaGCN [22] integrates image data into a neighborhood graph using graph convolutional networks"
- [abstract] multi-view representations: "MCGAE advances the field by creating multi-view representations from gene expression and spatial adjacency matrices."
- [discussion] scalability limitation: [evidence span withheld — anchor status: not_found]
