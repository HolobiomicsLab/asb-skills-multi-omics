---
name: marker-based-cluster-annotation-verification
description: Use when clustering labels (K-means, Specter, Seurat, etc.) have been
  assigned to spots/cells and the biological identity of each cluster must be validated
  against known marker genes or proteins.
when_to_use_negative:
- The cluster labels have not yet been computed or the low-dimensional embedding is
  still being tuned — verification is a downstream check, not a substitute for the
  clustering step
- No prior biological knowledge of expected marker expression exists for the tissue
  (e.g., novel tissue with no established marker panel) — then use unbiased DE markers
  instead
- Data are raw counts that have not been library-size normalized, log-transformed,
  and scaled — marker comparisons on unnormalized counts are unreliable
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Seurat V4
  role: R toolkit used for preprocessing and per-cluster differential expression (FindAllMarkers)
    to assign cell-type identity to clusters
  repo: https://github.com/satijalab/seurat
- name: sklearn.cluster.KMeans
  role: Produces the cluster labels whose annotations are verified against markers
    (k=7 for the tonsil spatial domains)
- name: SMOPCA
  role: Supplies the integrated low-dimensional representation (posterior mean of
    the latent factor) that the verified clusters are derived from
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:783bffc5c58a85b6e3dd4094b2d32a9071662a8a51f428cead87fa637dbb9f09
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
  - MEFISTO
  - SMOPCA
  - SPARK
  - Seurat V4
  - SpaVAE
  - SpatialGlue
  - SpatialPCA
  - TotalVI
  - scMDC
  - sklearn.cluster.KMeans
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# Marker-based cluster annotation verification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Sanity-checks data-driven spatial-domain or cell-type cluster labels by confirming that known marker genes/proteins are upregulated in the expected clusters (e.g., CD3/CD4/CD49a in T-cell zones, CD21/CD23/IgM in germinal centers) before trusting benchmark outputs.

## When to use

Use when clustering labels (K-means, Specter, Seurat, etc.) have been assigned to spots/cells and the biological identity of each cluster must be validated against known marker genes or proteins.

## When NOT to use

- The cluster labels have not yet been computed or the low-dimensional embedding is still being tuned — verification is a downstream check, not a substitute for the clustering step
- No prior biological knowledge of expected marker expression exists for the tissue (e.g., novel tissue with no established marker panel) — then use unbiased DE markers instead
- Data are raw counts that have not been library-size normalized, log-transformed, and scaled — marker comparisons on unnormalized counts are unreliable

## Inputs

- Cluster label vector per spot/cell (e.g., from KMeans on SMOPCA latent factors, k=7)
- Normalized, log-transformed gene expression matrix (e.g., 28,417-gene tonsil spatial-CITE-seq matrix)
- Normalized protein/ADT matrix (e.g., 283-protein panel from GSE213264)
- Known marker list per expected cell type (e.g., CD3/CD4/CD49a for T cells; CD21/CD23/IgM for germinal centers)

## Outputs

- Per-cluster marker upregulation table (cluster-to-annotation assignments)
- Annotation-verification entries appended to the benchmark results file (e.g., smopca_tonsil_benchmark.csv)
- Optionally, per-cluster DE marker lists (e.g., FindAllMarkers output)

## How to apply

After obtaining cluster labels from the low-dimensional representation (e.g., sklearn.cluster.KMeans with k=7 on SMOPCA posterior-mean latent factors of the tonsil spatial-CITE-seq data), compute per-cluster expression of expected markers for each modality (RNA and protein). In the SMOPCA tonsil application, cluster 6 is verified as the T-cell zone by upregulated CD3, CD4, and CD49a, while germinal-center clusters are verified by upregulated CD21, CD23, and IgM (protein markers). Use per-cluster differential-expression tests such as Seurat's FindAllMarkers (max.cells.per.ident=100, min.diff.pct=0.3, only.pos=TRUE) when running in R, or equivalent per-cluster mean comparisons. Record which clusters match expected marker patterns alongside the quantitative benchmark metrics (Moran's I, LISI) in the results file (e.g., smopca_tonsil_benchmark.csv) so annotation quality is documented next to clustering performance.

## Related tools

- **Seurat V4** (R toolkit used for preprocessing and per-cluster differential expression (FindAllMarkers) to assign cell-type identity to clusters) — https://github.com/satijalab/seurat
- **sklearn.cluster.KMeans** (Produces the cluster labels whose annotations are verified against markers (k=7 for the tonsil spatial domains))
- **SMOPCA** (Supplies the integrated low-dimensional representation (posterior mean of the latent factor) that the verified clusters are derived from)

## Examples

```
specter.rna.markers <- FindAllMarkers(pbmc, max.cells.per.ident = 100, min.diff.pct = 0.3, only.pos = TRUE)
```

## Evaluation signals

- Expected markers show upregulation in the annotated cluster: CD3/CD4/CD49a high in cluster 6 (T-cell zone); CD21/CD23/IgM high in germinal-center clusters
- Each of the k clusters (k=7 for the tonsil data) receives a consistent marker-based identity, or is flagged as unannotated
- The verified annotations are spatially coherent with the Moran's I / LISI claims (annotated domains should correspond to the spatially smooth clusters)
- Annotation checks are written to the benchmark output (smopca_tonsil_benchmark.csv) alongside metrics, so label provenance is auditable
- Positive-only DE markers (FindAllMarkers, only.pos=TRUE) are enriched for known markers rather than ambient/technology artifacts

## Limitations

- Verification depends on a curated marker panel; clusters lacking canonical markers (e.g., intermediate zones in the tonsil) cannot be confidently annotated this way
- Normalization choices (library-size normalization, log transform, scaling to zero mean and unit variance) affect marker fold-change comparisons, and the underlying clustering does not model dropout or mean–variance relationships of raw counts
- Marker agreement does not guarantee clustering optimality — it only validates biological plausibility, so it must be paired with quantitative metrics like Moran's I and LISI

## Evidence

- [results] Verify cluster annotations against marker expression (CD3/CD4/CD49a upregulated in the T-cell zone cluster 6; CD21/CD23/IgM upregulated in germinal-center clusters): [evidence span withheld — anchor status: not_found]
- [readme] Specter README: DE test to assign cell type identity to clusters with FindAllMarkers: [evidence span withheld — anchor status: not_found]
- [readme] Specter README: Seurat used for preprocessing and visualization of CITE-seq clustering results: [evidence span withheld — anchor status: not_found]
