---
name: spot-level-latent-embedding-training
description: Use when spatial transcriptomics spot-level data (expression matrix plus
  a spatial adjacency matrix built from spot coordinates) must be converted into low-dimensional
  latent spot embeddings before spatial domain clustering or downstream multimodal
  integration.
when_to_use_negative:
- Input lacks spatial coordinate information, so no spatial adjacency matrix can be
  constructed and the multi-view graph structure underlying MCGAE is unavailable
- The analysis goal is expression-only clustering without spatial relationships —
  non-spatial linear dimensionality reduction (e.g. Seurat-style) is the article's
  own baseline for that case
- Very large sample counts where the O(n^2) adjacency matrix computation may rapidly
  exhaust computational resources
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3473
tools:
- name: MCGAE
  role: the multi-view contrastive graph autoencoder framework whose GCN encoder and
    Deep Graph Infomax objective produce the spot-level latent embeddings
  repo: github.com/yiwen-yang/MCGAE
- name: PyTorch
  role: framework in which the Eq. 1 GCN encoder and Eq. 2 DGI contrastive objective
    are implemented and trained
- name: ResNet50
  role: pre-trained network used for extracting morphological image features that
    enrich the multimodal latent representation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:392ef0f98a77b39f4b0b90fff8c18affc5b26f39169fc03be22c57fa6470dee8
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
  - PyTorch
  - ResNet50
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# spot-level-latent-embedding-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a contrastive graph convolutional autoencoder (MCGAE-style) that converts a spot-by-gene expression matrix and a spatial adjacency matrix into modality-specific latent spot embeddings (Z^X and Z^A) via Deep Graph Infomax self-supervision, forming the basis for spatial domain identification.

## When to use

Use when spatial transcriptomics spot-level data (expression matrix plus a spatial adjacency matrix built from spot coordinates) must be converted into low-dimensional latent spot embeddings before spatial domain clustering or downstream multimodal integration.

## When NOT to use

- Input lacks spatial coordinate information, so no spatial adjacency matrix can be constructed and the multi-view graph structure underlying MCGAE is unavailable
- The analysis goal is expression-only clustering without spatial relationships — non-spatial linear dimensionality reduction (e.g. Seurat-style) is the article's own baseline for that case
- Very large sample counts where the O(n^2) adjacency matrix computation may rapidly exhaust computational resources

## Inputs

- spot-by-gene expression feature matrix for the slice
- spatial adjacency matrix computed from spot coordinates
- optional morphological image features (e.g. ResNet50-derived) for the multimodal view

## Outputs

- modality-specific latent embedding matrix Z^X (expression view), one row per spot with fixed latent dimension
- modality-specific latent embedding matrix Z^A (adjacency view), one row per spot with fixed latent dimension
- mcgae_spot_embeddings.csv containing both per-spot embedding matrices

## How to apply

Load the chained multi-view graph inputs: the spot-by-gene expression feature matrix and the spatial adjacency matrix for the slice. Implement the GCN encoder of Eq. 1 in PyTorch, propagating spot features through the normalized spatial adjacency matrix, and implement the Deep Graph Infomax objective of Eq. 2, using a sigmoid-of-neighbor-average readout to obtain the graph-level summary. Negative pairs are produced by perturbing the data: gene expression profiles are randomized (shuffled) while the network topology is kept constant. Train the encoder on the multi-view graphs until the contrastive loss converges, then extract the modality-specific latent representations Z^X (expression view) and Z^A (adjacency view), each with one row per spot and a fixed latent dimension. These modality-specific spatial representations can subsequently be integrated into a unified multimodal embedding, optionally enriched with morphological image features from a pre-trained ResNet50.

## Related tools

- **MCGAE** (the multi-view contrastive graph autoencoder framework whose GCN encoder and Deep Graph Infomax objective produce the spot-level latent embeddings) — github.com/yiwen-yang/MCGAE
- **PyTorch** (framework in which the Eq. 1 GCN encoder and Eq. 2 DGI contrastive objective are implemented and trained)
- **ResNet50** (pre-trained network used for extracting morphological image features that enrich the multimodal latent representation)

## Examples

```
import torch; Z_X, Z_A = mcgae_dgi_train(torch.tensor(X_expr), torch.tensor(A_adj)); pd.DataFrame(Z_X, columns=['latent_%d'%i for i in range(Z_X.shape[1])]).to_csv('mcgae_spot_embeddings.csv')
```

## Evaluation signals

- The training script runs without error end-to-end on the slice's expression matrix and spatial adjacency matrix
- The output file mcgae_spot_embeddings.csv exists and its row count equals the number of spots
- The latent-dimension field is present in the output embedding file
- Both modality-specific embeddings Z^X (expression view) and Z^A (adjacency view) are produced with one row per spot and a fixed latent dimension
- The Deep Graph Infomax contrastive loss converges during training on the multi-view graphs

## Limitations

- Computation of the adjacency matrix for spatial position information incurs O(n^2) computational complexity, which may rapidly exhaust computational resources with large sample counts
- Batch effect handling via adjacency distance of neighboring slices may not suffice for other complex datasets
- No version is specified for the pre-trained ResNet50 network used for image feature extraction
- Scalability to large-scale datasets would require subgraph sampling or deep multichannel GCN strategies, not yet implemented

## Evidence
