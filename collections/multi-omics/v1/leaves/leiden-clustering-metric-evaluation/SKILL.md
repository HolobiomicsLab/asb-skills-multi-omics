---
name: leiden-clustering-metric-evaluation
description: Use when you have SSGATE (or comparable) integrated embeddings from paired
  transcriptome/proteome single-cell data (e.g. BMNC, SLN111_D1) and need to quantitatively
  compare integration quality across hyperparameter settings via label-annotated clustering.
when_to_use_negative:
- When only spatial multi-omics data without ground-truth cell labels is available
  — the evaluation metrics require reference labels to compute ARI/NMI
- When evaluating raw expression profiles rather than integrated joint embeddings
  — the skill scores integration output, not input normalization quality
- When the research question concerns spatial coordinate-driven neighborhood graphs
  (Radius model on obsm['spatial']) rather than expression-based single-cell graphs
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SSGATE
  role: Produces the integrated joint embedding (dual-path graph attention auto-encoder,
    out_dims=30) whose Leiden clusterings are evaluated
  repo: https://github.com/Linliu-Bioinf/SSGATE
- name: SCANPY
  role: Provides Leiden clustering and single-cell analysis utilities used to cluster
    embeddings and handle AnnData objects
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae450
    title: Multi-omics integration for both single-cell and spatially resolved data
      based on dual-path graph attention auto-encoder
  source_content_hash: sha256:13b56e58401c72251eba37c8f57d214870840306c8c4227733729266f242f764
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Linliu-Bioinf/SSGATE
  tools:
  - SCANPY
  - SSGATE
  - Seurat v5
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# leiden-clustering-metric-evaluation

## Summary

Benchmark protocol for SSGATE-style multi-omics integration: cluster the integrated joint embeddings with the Leiden algorithm across resolutions 0.1–1.0 and score each clustering against ground-truth cell labels using Purity, Homogeneity Score, ARI, and NMI, as done to produce Table 3-style sensitivity tables.

## When to use

Use when you have SSGATE (or comparable) integrated embeddings from paired transcriptome/proteome single-cell data (e.g. BMNC, SLN111_D1) and need to quantitatively compare integration quality across hyperparameter settings via label-annotated clustering.

## When NOT to use

- When only spatial multi-omics data without ground-truth cell labels is available — the evaluation metrics require reference labels to compute ARI/NMI
- When evaluating raw expression profiles rather than integrated joint embeddings — the skill scores integration output, not input normalization quality
- When the research question concerns spatial coordinate-driven neighborhood graphs (Radius model on obsm['spatial']) rather than expression-based single-cell graphs

## Inputs

- Integrated joint embedding matrix (SSGATE output, embedding dimension 30) for cells from paired transcriptome/proteome data (e.g. BMNC, SLN111_D1) in AnnData format
- Ground-truth cell-type labels for the same cells
- Cluster resolution sweep range (0.1 to 1.0) and number of independent repeats (10)

## Outputs

- Per-resolution clustering assignments (Leiden labels) for each embedding and repeat
- Metric table of Purity, Homogeneity Score, ARI, NMI per hyperparameter setting and dataset
- Aggregate ranking table identifying the best hyperparameter setting (paper: balance parameter 0.1 with neighbors fixed at 15)

## How to apply

Train the dual-path graph attention auto-encoder to obtain the joint embedding (README defaults: out_dims=30, n_epochs=300, lr=0.001, hidden_dims1/2=128), then for each hyperparameter setting cluster the integrated embeddings with the Leiden algorithm while sweeping resolutions from 0.1 to 1.0, repeating over 10 independent experiments to control stochasticity. For each resolution and repeat, compute Purity, Homogeneity Score, ARI, and NMI of the Leiden clusters against ground-truth cell-type labels. Fix non-swept parameters (e.g. number of neighbors at 15 in the neighbor-network construction) so the swept parameter is the only variable. Aggregate per-setting metric values/rankings into a summary table per dataset; a setting is judged best only if it wins across all four metrics on both datasets, mirroring the paper's balance-parameter=0.1 confirmation.

## Related tools

- **SSGATE** (Produces the integrated joint embedding (dual-path graph attention auto-encoder, out_dims=30) whose Leiden clusterings are evaluated) — https://github.com/Linliu-Bioinf/SSGATE
- **SCANPY** (Provides Leiden clustering and single-cell analysis utilities used to cluster embeddings and handle AnnData objects)

## Examples

```
import scanpy as sc; for res in [0.1,0.3,0.5,0.7,1.0]: sc.tl.leiden(adata_int, resolution=res, key_added=f'leiden_{res}')
```

## Evaluation signals

- For the paper's confirmation case (neighbors=15), the balance parameter 0.1 yields the best values in all four metrics (Purity, Homogeneity, ARI, NMI) on both BMNC and SLN111_D1, matching Table 3
- All four metrics fall in the valid [0, 1] range for every resolution and repeat; NMI and ARI should be near zero for random labelings
- Results are stable across 10 independent experiments (low variance per setting); a single-run best result is not acceptable evidence
- Metric curves are reported across the full resolution sweep 0.1–1.0, not at a single resolution, so the summary table reflects the sweep

## Limitations

- SSGATE hyperparameters are defaults 'based on experience' rather than systematically derived, so the swept optimum may not generalize to new datasets
- The paper's evaluation covers transcriptome/proteome integration; normalization and dimensionality reduction for other omics types are not yet available
- No software versions are reported for compared methods, complicating exact reproduction of the metric tables

## Evidence

- [abstract] SSGATE shows better performance and stronger robustness than competitive methods and facilitates downstream analysis.: "SSGATE shows better performance and stronger robustness than competitive methods and facilitates downstream analysis."
- [abstract] Neighborhood graph construction from expression profiles: "It can construct the neighborhood graphs based on single-cell expression profiles or spatial coordinates"
- [readme] README training defaults with embedding dimension and epochs: "hidden_dims1 = 128, hidden_dims2 = 128, out_dims = 30, cluster_update_epoch = 20, epochs_init = 50, n_epochs=300, lr=0.001"
- [discussion] Hyperparameters are experience-based defaults: "Based on experience, we provide a set of hyperparameters suitable for most datasets as default settings"
