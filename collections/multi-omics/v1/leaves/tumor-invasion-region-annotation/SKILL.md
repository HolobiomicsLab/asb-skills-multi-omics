---
name: tumor-invasion-region-annotation
description: Use when applying MCGAE to 10x Visium spatial transcriptomics data (count
  matrix + spot coordinates + H&E image, e.g., colorectal cancer liver metastasis
  sections) to identify tumor-invasion spatial domains spanning tumor and connective
  tissue.
when_to_use_negative:
- Input lacks spot spatial coordinates and a histology image, so no spatial adjacency
  matrix or image features can be constructed (though MCGAE is described as capable
  of functioning without image information, both modalities are expected for this
  annotation task)
- Goal is purely non-spatial expression clustering (the article notes algorithms like
  Seurat overlook vital spatial relationships and are the wrong choice for spatial
  domain identification)
- 'Dataset is very large-scale: the adjacency matrix computation incurs O(n^2) complexity
  and may exhaust computational resources; the authors suggest subgraph sampling or
  deep multichannel GCN strategies for such cases'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MCGAE
  role: Multi-view contrastive graph autoencoder that creates multi-view representations
    from gene expression and spatial adjacency matrices, integrates modality-specific
    representations into a unified embedding, and clusters it to identify tumor-invasion
    spatial domains
  repo: github.com/yiwen-yang/MCGAE
- name: ResNet50
  role: Pre-trained network used by MCGAE to extract morphological image features
    per Visium spot for inclusion in the multimodal integration
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:0d8c254981e1381060f7249702c337f1969099f219754304953b8cf6ee3818d5
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# tumor-invasion-region-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotates spatial transcriptomics spots into spatial domains and flags a tumor-invasion region using the MCGAE multi-view contrastive graph autoencoder, which fuses gene expression, spatial adjacency, and ResNet50-derived morphological image features via a self-attention mechanism. Demonstrated on the 10x Visium colorectal cancer liver metastasis (CRLM) dataset to recover the tumor/connective-tissue-spanning region Y.

## When to use

Use when applying MCGAE to 10x Visium spatial transcriptomics data (count matrix + spot coordinates + H&E image, e.g., colorectal cancer liver metastasis sections) to identify tumor-invasion spatial domains spanning tumor and connective tissue.

## When NOT to use

- Input lacks spot spatial coordinates and a histology image, so no spatial adjacency matrix or image features can be constructed (though MCGAE is described as capable of functioning without image information, both modalities are expected for this annotation task)
- Goal is purely non-spatial expression clustering (the article notes algorithms like Seurat overlook vital spatial relationships and are the wrong choice for spatial domain identification)
- Dataset is very large-scale: the adjacency matrix computation incurs O(n^2) complexity and may exhaust computational resources; the authors suggest subgraph sampling or deep multichannel GCN strategies for such cases

## Inputs

- 10x Visium spatial gene expression count matrix (CRLM dataset)
- Spot spatial coordinates for adjacency matrix construction
- H&E histology image for ResNet50 morphological feature extraction
- Pre-trained ResNet50 network used by MCGAE

## Outputs

- Per-spot spatial domain label table (crlm_spatial_domains.csv) covering all Visium spots
- Annotated tumor-invasion region Y spanning tumor and connective tissue
- t-SNE embedding of region-Y spots resolving three sub-clusters (largest = Cluster 0, mainly tumor cells)

## How to apply

Load the CRLM 10x Visium count matrix, spatial coordinates, and H&E image packaged in github.com/yiwen-yang/MCGAE, and extract morphological features per spot with the pre-trained ResNet50 network. Build the spatial adjacency matrix from spot coordinates and construct multi-view representations from gene expression and spatial adjacency, as MCGAE does to generate modality-specific spatial representations. Train the multi-view contrastive graph autoencoder, letting the self-attention mechanism fuse expression, spatial position, and image features into a unified embedding, then cluster this embedding to assign a spatial domain label to every spot. Annotate the domain spanning tumor and connective tissue as tumor-invasion region Y and confirm by t-SNE that spots within region Y alone split into three main clusters, with the largest cluster (Cluster 0) consisting mainly of tumor cells, and that lymphatic structures separate from tumor domains.

## Related tools

- **MCGAE** (Multi-view contrastive graph autoencoder that creates multi-view representations from gene expression and spatial adjacency matrices, integrates modality-specific representations into a unified embedding, and clusters it to identify tumor-invasion spatial domains) — github.com/yiwen-yang/MCGAE
- **ResNet50** (Pre-trained network used by MCGAE to extract morphological image features per Visium spot for inclusion in the multimodal integration)

## Evaluation signals

- Pipeline script runs end-to-end and writes crlm_spatial_domains.csv covering all Visium spots (row_count_equals) with a domain field present (field_present)
- t-SNE of spots within region Y alone resolves three main clusters (value_in_range), matching the Fig. 4 result reported in the paper
- Largest region-Y cluster (Cluster 0) consists mainly of tumor cells, supporting that region Y reflects tumor invasion into non-tumor domains
- Lymphatic structures separate from tumor domains in the spatial domain map, reproducing Fig. 4
- The annotated region Y spans both tumor and connective tissue in the CRLM sample

## Limitations

- Adjacency matrix computation for spatial position information incurs O(n^2) computational complexity, which may rapidly exhaust computational resources as sample counts increase
- No version is specified for the pre-trained ResNet50 network used for image feature extraction, complicating reproducibility
- The batch effect handling strategy based on distance between neighboring regions may not suffice for other complex datasets
- MCGAE is capable of functioning effectively even in the absence of image information, but the image-enriched route is the one demonstrated for CRLM tumor-invasion identification

## Evidence
