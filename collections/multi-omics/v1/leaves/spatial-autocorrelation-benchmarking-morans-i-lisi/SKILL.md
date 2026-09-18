---
name: spatial-autocorrelation-benchmarking-morans-i-lisi
description: Use when comparing spatial-domain detection methods on spatially-resolved
  multi-omics data (e.g., spatial-CITE-seq) and you need quantitative spatial-smoothness
  scores — Moran's I and LISI — computed on predicted spot-level cluster labels.
when_to_use_negative:
- The data are non-spatial single-cell multi-omics without tissue coordinates — Moran's
  I/LISI spatial smoothness is undefined unless you deliberately adopt SMOPCA's UMAP
  pseudo-spatial-coordinate workaround.
- You need absolute biological accuracy against ground-truth domain annotations rather
  than a relative spatial-smoothness ranking of competing methods.
- You have only one method's cluster labels and no baselines — a single Moran's I/LISI
  value without comparison methods provides no benchmark.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2269
tools:
- name: SMOPCA
  role: 'Method under test: its integrated posterior-mean latent factor Z, clustered
    with KMeans (k=7), is scored with Moran''s I and LISI and expected to top the
    ranking'
- name: sklearn.cluster.KMeans
  role: Produces the final k=7 spatial-domain labels from each low-dimensional representation
    prior to Moran's I / LISI scoring
- name: SPARK
  role: Selects up to 3000 spatially variable genes (Benjamini-Yekutieli FDR 0.05)
    that fix the shared gene feature space across all methods
- name: SpatialPCA
  role: Baseline spatially aware dimension-reduction method included in the Moran's
    I / LISI comparison
  repo: https://github.com/shangll123/SpatialPCA
- name: MEFISTO
  role: Baseline multi-omics factor method (handles 3+ modalities) included in the
    comparison
  repo: https://biofam.github.io/MOFA2/MEFISTO
- name: SpatialGlue
  role: Baseline graph-neural-network integration method with dual-attention mechanism
    included in the comparison
  repo: https://github.com/JinmiaoChenLab/SpatialGlue/tree/main
- name: SpaVAE
  role: Baseline spatial variational-autoencoder method included in the comparison
  repo: https://github.com/ttgump/spaVAE
- name: scMDC
  role: Baseline single-cell multi-omics deep clustering method included in the comparison
  repo: https://github.com/xianglin226/scMDC
- name: Seurat V4
  role: Baseline single-cell multimodal integration method included in the comparison
  repo: https://github.com/satijalab/seurat
- name: TotalVI
  role: Baseline probabilistic CITE-seq integration method included in the comparison
  repo: https://scvi-tools.org
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:de5ff3d8a8f2dbb5d0a9852410117ac7574a238adda191f2c6de1f6d328b41b1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/hemberg-lab/SC3
  repo_url: https://github.com/shangll123/SpatialPCA
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# spatial-autocorrelation-benchmarking-morans-i-lisi

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill that scores spatial-domain clusters from spatial multi-omics dimension-reduction methods (SMOPCA vs SpatialPCA, MEFISTO, SpatialGlue, SpaVAE, scMDC, Seurat, TotalVI, PCA+K-means) using Moran's I for spatial autocorrelation and LISI for neighborhood cell-type mixing, demonstrated on the human tonsil spatial-CITE-seq dataset GSE213264.

## When to use

Use when comparing spatial-domain detection methods on spatially-resolved multi-omics data (e.g., spatial-CITE-seq) and you need quantitative spatial-smoothness scores — Moran's I and LISI — computed on predicted spot-level cluster labels.

## When NOT to use

- The data are non-spatial single-cell multi-omics without tissue coordinates — Moran's I/LISI spatial smoothness is undefined unless you deliberately adopt SMOPCA's UMAP pseudo-spatial-coordinate workaround.
- You need absolute biological accuracy against ground-truth domain annotations rather than a relative spatial-smoothness ranking of competing methods.
- You have only one method's cluster labels and no baselines — a single Moran's I/LISI value without comparison methods provides no benchmark.

## Inputs

- Human tonsil spatial-CITE-seq count data (GSE213264): 2491 spots x 28,417 genes and 283 proteins
- Spot 2D spatial coordinates for the neighborhood graph / spatial kernel used by Moran's I, LISI, and SMOPCA
- Low-dimensional embeddings and predicted spatial-domain labels per method (SMOPCA posterior-mean Z + KMeans k=7; SpatialPCA, MEFISTO, SpatialGlue, SpaVAE, scMDC, Seurat, TotalVI, PCA+K-means outputs)
- Marker panel for annotation checks: CD3, CD4, CD49a, CD21, CD23, IgM

## Outputs

- Per-method Moran's I and LISI scores on the tonsil spatial-CITE-seq benchmark
- Benchmark table written to smopca_tonsil_benchmark.csv including metrics and marker-based annotation checks
- Spatial domain labels (k=7 clusters) per method available for downstream marker verification

## How to apply

Preprocess a shared input for every method: human tonsil spatial-CITE-seq data (GSE213264; 2491 spots, 28,417 genes, 283 proteins) with zero-count genes/proteins removed, up to 3000 SPARK-selected spatially variable genes at a Benjamini-Yekutieli FDR of 0.05 combined with all 283 proteins, then library-size normalization, log transformation, and scaling to zero mean/unit variance. Obtain spot-level spatial domain labels from each method — for SMOPCA take the posterior mean of the latent factor Z and run sklearn.cluster.KMeans with k=7 — keeping the feature space and cluster count consistent so scores are comparable. For each method compute Moran's I, where higher values indicate more spatially autocorrelated, homogeneous clusters, and LISI, where lower values indicate less mixing of different cell types among spatial neighbors. Rank SMOPCA against the baselines SpatialPCA, MEFISTO, SpatialGlue, SpaVAE, scMDC, Seurat, TotalVI, and PCA+K-means; the expected outcome on this dataset is SMOPCA achieving the highest Moran's I and the lowest LISI. Sanity-check that spatial smoothness corresponds to biology via marker expression (CD3/CD4/CD49a upregulated in the T-cell-zone cluster 6; CD21/CD23/IgM upregulated in germinal-center clusters) and write the metrics and annotation checks to smopca_tonsil_benchmark.csv.

## Related tools

- **SMOPCA** (Method under test: its integrated posterior-mean latent factor Z, clustered with KMeans (k=7), is scored with Moran's I and LISI and expected to top the ranking)
- **sklearn.cluster.KMeans** (Produces the final k=7 spatial-domain labels from each low-dimensional representation prior to Moran's I / LISI scoring)
- **SPARK** (Selects up to 3000 spatially variable genes (Benjamini-Yekutieli FDR 0.05) that fix the shared gene feature space across all methods)
- **SpatialPCA** (Baseline spatially aware dimension-reduction method included in the Moran's I / LISI comparison) — https://github.com/shangll123/SpatialPCA
- **MEFISTO** (Baseline multi-omics factor method (handles 3+ modalities) included in the comparison) — https://biofam.github.io/MOFA2/MEFISTO
- **SpatialGlue** (Baseline graph-neural-network integration method with dual-attention mechanism included in the comparison) — https://github.com/JinmiaoChenLab/SpatialGlue/tree/main
- **SpaVAE** (Baseline spatial variational-autoencoder method included in the comparison) — https://github.com/ttgump/spaVAE
- **scMDC** (Baseline single-cell multi-omics deep clustering method included in the comparison) — https://github.com/xianglin226/scMDC
- **Seurat V4** (Baseline single-cell multimodal integration method included in the comparison) — https://github.com/satijalab/seurat
- **TotalVI** (Baseline probabilistic CITE-seq integration method included in the comparison) — https://scvi-tools.org

## Examples

```
from sklearn.cluster import KMeans; labels = KMeans(n_clusters=7).fit_predict(Z_posterior_mean)
```

## Evaluation signals

- Ranking invariant on GSE213264 tonsil data: SMOPCA attains the highest Moran's I and the lowest LISI among SpatialPCA, MEFISTO, SpatialGlue, and SpaVAE.
- Comparability check: every method is run on the same feature matrix (SPARK SVGs at BY FDR 0.05 plus all 283 proteins) and clustered with the same k=7.
- Marker congruence: CD3/CD4/CD49a upregulated in the T-cell-zone cluster 6; CD21/CD23/IgM upregulated in germinal-center clusters.
- Artifact check: smopca_tonsil_benchmark.csv contains one Moran's I / LISI row per method plus annotation-check results.
- Preprocessing invariant: zero-count genes/proteins are removed and the normalized matrix has zero mean and unit variance.

## Limitations

- SMOPCA's current implementation has O(n^3) time and O(n^2) space complexity, so the benchmark does not scale to very large spot counts; the authors aim to reduce both in future work.
- SMOPCA uses normalized data rather than raw counts and does not model dropout events or the mean-variance relationship of raw counts, which the authors note may cause a loss of inference accuracy and reduced analytical power.
- The lengthscale parameter may be hard to learn when data are very sparse and noisy or the spatial structure is very complicated; manual user tuning may be required.
- A shared covariance assumption (Sigma_1=...=Sigma_d) across modalities is imposed for computational convenience; no closed-form solution exists when it is relaxed.
- Moran's I and LISI quantify spatial smoothness and neighbor mixing, not biological correctness of domain identity — marker-based annotation checks (CD3/CD4/CD49a; CD21/CD23/IgM) are still required.

## Evidence

- [discussion] Tonsil dataset dimensions: [evidence span withheld — anchor status: not_found]
- [readme] SpatialPCA baseline description: [evidence span withheld — anchor status: external_doc]
