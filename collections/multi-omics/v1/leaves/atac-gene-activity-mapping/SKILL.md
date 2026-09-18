---
name: atac-gene-activity-mapping
description: Use when integrating spatial ATAC-seq with mRNA data whose models require
  matched gene-level features — e.g., the peak matrix must be collapsed to gene activity
  before joint dimension reduction or multi-omics fusion.
when_to_use_negative:
- When the downstream method accepts raw peak matrices directly (e.g., chromVAR, cisTopic,
  SCALE operate on ATAC peaks), collapsing to gene activity discards motif/peak-level
  resolution
- When the mRNA modality is not present or the analysis is single-modal ATAC clustering
  — gene activity mapping is only needed to harmonize features across modalities
- 'When spots/cells have not been quality-controlled: mapping reads to genes before
  filtering can propagate low-quality spots into the joint model'
edam_topics:
- http://edamontology.org/topic_3678
- http://edamontology.org/topic_3168
tools:
- name: Signac
  role: Provides the Satija lab protocol functions (e.g., FindTopFeatures v1.4.0 to
    keep peaks detected in at least 200 cells) used to collapse the peak matrix into
    a gene activity matrix
- name: Seurat
  role: Part of the Satija lab workflow for mapping ATAC reads to gene regions and
    preprocessing the resulting gene activity matrix alongside mRNA counts
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:38c0de86a02b8b92961acf02e36ca565570e8fe51926bd02629c5947362bb6be
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/hemberg-lab/SC3
  repo_url: https://github.com/satijalab/seurat
  tools:
  - Seurat
  - Signac
  - SpatialPCA
  - scMDC
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3678
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3
n_claims: 10
---

# atac-gene-activity-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Collapses a sparse ATAC-seq peak-by-spot matrix into a gene-by-spot gene activity matrix by mapping ATAC reads to gene regions, following the Satija lab protocol, so that ATAC and mRNA modalities share a common gene feature space for multimodal integration (e.g., SMOPCA on MISAR-seq E15.5 mouse brain data).

## When to use

Use when integrating spatial ATAC-seq with mRNA data whose models require matched gene-level features — e.g., the peak matrix must be collapsed to gene activity before joint dimension reduction or multi-omics fusion.

## When NOT to use

- When the downstream method accepts raw peak matrices directly (e.g., chromVAR, cisTopic, SCALE operate on ATAC peaks), collapsing to gene activity discards motif/peak-level resolution
- When the mRNA modality is not present or the analysis is single-modal ATAC clustering — gene activity mapping is only needed to harmonize features across modalities
- When spots/cells have not been quality-controlled: mapping reads to genes before filtering can propagate low-quality spots into the joint model

## Inputs

- ATAC-seq peak count matrix (peaks x spots) from spatial ATAC data, e.g., MISAR-seq E15.5 mouse embryonic brain (initial 2500 spots, 105,350 peaks before filtering)
- Genome annotation / gene region definitions used to aggregate reads into gene bodies
- Optional per-spot spatial coordinates to keep spot identities aligned with the mRNA modality

## Outputs

- Gene activity matrix (genes x spots) with spot order matching the mRNA count matrix
- Filtered feature set, e.g., peaks detected in at least 200 cells (47,287 of 105,350 peaks retained via FindTopFeatures)
- Spot-retention record after preprocessing (e.g., 1949 of 2500 MISAR-seq spots retained)

## How to apply

Starting from the ATAC peak matrix, map ATAC reads to gene regions and collapse the peak matrix into a gene activity matrix adhering to the established Satija lab protocol (implemented via Signac/Seurat tooling). Optionally pre-filter peaks, e.g., with the FindTopFeatures function in Signac (v1.4.0) selecting peaks detected in at least 200 cells, which in the article retained 47,287 of 105,350 peaks. The resulting gene activity matrix has the same spots and a gene-level feature space compatible with the mRNA count matrix, enabling shared preprocessing (library-size normalization, log transformation, scaling to zero mean and unit variance) and joint modeling. After preprocessing of the MISAR-seq E15.5 mouse embryonic brain dataset, 1949 of the initial 2500 spots were retained. The decision point is the gene body/region definition used for aggregation; deviating from the Satija protocol changes the feature values and downstream benchmark comparability.

## Related tools

- **Signac** (Provides the Satija lab protocol functions (e.g., FindTopFeatures v1.4.0 to keep peaks detected in at least 200 cells) used to collapse the peak matrix into a gene activity matrix)
- **Seurat** (Part of the Satija lab workflow for mapping ATAC reads to gene regions and preprocessing the resulting gene activity matrix alongside mRNA counts) — https://github.com/satijalab/seurat

## Examples

```
gene_activities <- Signac::GeneActivity(seurat_obj)
```

## Evaluation signals

- Spot count after preprocessing matches expectation: 1949 of the initial 2500 MISAR-seq spots retained
- Peak filtering behaves as documented: FindTopFeatures retains 47,287 of 105,350 peaks when requiring detection in at least 200 cells
- Gene activity matrix gene identifiers align with (or are mappable to) the mRNA matrix features, enabling shared normalization (library size, log transform, zero mean/unit variance scaling)
- Downstream SMOPCA on the mapped gene activity plus SPARK-filtered (FDR 0.05, Benjamini-Yekutieli) spatially variable genes yields competitive AMI/NMI/ARI clustering metrics
- McFadden-adjusted pseudo R2 of the resulting latent representation is computable and comparable to competing methods (chromVAR, cisTopic, LSA, PeakVI, SCALE, scMDC, Seurat, SpatialPCA)

## Limitations

- Collapsing peaks to gene activity loses peak-level and motif-level resolution that peak-based ATAC methods (e.g., chromVAR, cisTopic, SCALE) exploit
- The number of retained spots after preprocessing (1949/2500) shows that spot quality filtering precedes or accompanies the mapping and can substantially shrink the dataset
- Aggregation choices (gene region definitions, peak filters) from the Satija lab protocol must be kept consistent to remain comparable with published benchmarks

## Evidence

- [methods] we mapped ATAC reads to gene regions and collapsed the peak matrix into a gene activity matrix, adhering to the established protocol from the Satija lab: "we mapped ATAC reads to gene regions and collapsed the peak matrix into a gene activity matrix, adhering to the established protocol from the Satija lab"
- [methods] We then use the “FindTopFeatures” function in Signac [100] (v1.4.0) to select the peaks that are at least detected in 200 cells: "We then use the “FindTopFeatures” function in Signac [100] (v1.4.0) to select the peaks that are at least detected in 200 cells"
- [discussion] After preprocessing, 1949 out of the initial 2500 spots were retained.: "After preprocessing, 1949 out of the initial 2500 spots were retained."
