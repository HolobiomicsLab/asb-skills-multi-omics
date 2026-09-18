---
name: perturbed-graph-negative-sampling
description: Use when training a self-supervised graph convolutional encoder (e.g.,
  MCGAE's Eq. 2 DGI contrastive objective) on spatial transcriptomics graphs and negative
  examples are needed for the contrastive loss.
when_to_use_negative:
- Very large datasets where the O(n^2) spatial adjacency matrix computation exhausts
  resources — the article notes this limits scalability and suggests subgraph sampling
  or deep multichannel GCN strategies instead
- Settings where the graph topology itself is noisy or unreliable — this negative-sampling
  scheme freezes the topology and only perturbs features, so it cannot teach the encoder
  to distrust the adjacency
- Analyses with supervised/labelled spot annotations available — a purely self-supervised
  DGI negative-sampling setup discards that supervision
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: MCGAE
  role: Framework whose Deep Graph Infomax contrastive objective (Eq. 2) consumes
    the perturbed (shuffled-feature, fixed-topology) graphs as negatives while training
    the Eq. 1 GCN encoder
  repo: github.com/yiwen-yang/MCGAE
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbae608
    title: 'MCGAE: unraveling tumor invasion through integrated multimodal spatial
      transcriptomics'
  source_content_hash: sha256:179c681aafc2fbebf306eaf5a924bddbad8c123d22198f33d2409f5fa8dafc5e
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
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.25
n_claims: 8
---

# perturbed-graph-negative-sampling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates negative training pairs for MCGAE's Deep Graph Infomax (DGI) contrastive objective by perturbing the data: spot-by-gene expression profiles are randomly shuffled while the spatial adjacency (network topology) is held constant, so the encoder learns latent spot representations invariant to feature randomization.

## When to use

Use when training a self-supervised graph convolutional encoder (e.g., MCGAE's Eq. 2 DGI contrastive objective) on spatial transcriptomics graphs and negative examples are needed for the contrastive loss.

## When NOT to use

- Very large datasets where the O(n^2) spatial adjacency matrix computation exhausts resources — the article notes this limits scalability and suggests subgraph sampling or deep multichannel GCN strategies instead
- Settings where the graph topology itself is noisy or unreliable — this negative-sampling scheme freezes the topology and only perturbs features, so it cannot teach the encoder to distrust the adjacency
- Analyses with supervised/labelled spot annotations available — a purely self-supervised DGI negative-sampling setup discards that supervision

## Inputs

- spot-by-gene expression feature matrix (X) for the slice
- spatial adjacency matrix (A) built from spot spatial positions
- GCN encoder weights (Eq. 1) and DGI readout/summary vector (Eq. 2)

## Outputs

- modality-specific latent embedding Z^X (expression view), one row per spot, fixed latent dimension
- modality-specific latent embedding Z^A (adjacency view), one row per spot, fixed latent dimension
- combined embeddings written to mcgae_spot_embeddings.csv

## How to apply

Load the spot-by-gene expression feature matrix X and the spatial adjacency matrix A for the slice. Implement the GCN encoder of Eq. 1 in PyTorch, propagating spot features through the normalized spatial adjacency. For each negative example, draw a random permutation of spot indices (e.g., torch.randperm) to shuffle the rows of X while reusing the unchanged adjacency A, yielding a corrupted graph whose feature–topology correspondence is broken. Apply the same GCN encoder to the corrupted graph, then compute the DGI loss of Eq. 2 with a sigmoid-of-neighbor-average readout as the graph-level summary: positive (X, A) embeddings should score high against the summary, perturbed-graph embeddings should score low. Train until the contrastive loss converges, then extract modality-specific latents Z^X (expression view) and Z^A (adjacency view), one row per spot at a fixed latent dimension.

## Related tools

- **MCGAE** (Framework whose Deep Graph Infomax contrastive objective (Eq. 2) consumes the perturbed (shuffled-feature, fixed-topology) graphs as negatives while training the Eq. 1 GCN encoder) — github.com/yiwen-yang/MCGAE

## Examples

```
idx = torch.randperm(X.size(0)); X_neg = X[idx]  # shuffled features, same adjacency A -> negative graph for DGI Eq. 2
```

## Evaluation signals

- The training script runs without error end-to-end
- Output file mcgae_spot_embeddings.csv exists and its row count equals the number of spots in the slice
- The latent-dimension field is present and each latent matrix Z^X and Z^A has a fixed latent dimension
- The DGI contrastive loss (Eq. 2) converges during encoder training
- Negative graphs are verified to have shuffled feature rows but an adjacency matrix identical to the original topology

## Limitations

- Adjacency matrix computation from spatial positions incurs O(n^2) complexity, which may rapidly exhaust computational resources with an increasing number of samples
- Perturbing only gene expression while keeping topology constant ties the learned representation to the quality of the predefined spatial adjacency metric
- The MCGAE repository README provides hardware notes (i9-12900K, 64 GB RAM, RTX 3060) but no versioned specification of some components (e.g., the pre-trained ResNet50 used for image features)

## Evidence

- [other] negative pairs are produced by perturbing the data: gene expression profiles are randomized while the network topology is kept constant: [evidence span withheld — anchor status: not_found]
- [other] using a sigmoid-of-neighbor-average readout to obtain the graph-level summary and perturbed (shuffled-feature) graphs as negatives: [evidence span withheld — anchor status: not_found]
