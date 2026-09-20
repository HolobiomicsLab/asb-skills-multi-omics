---
name: cite-seq-protein-gene-integration
description: Use when paired CITE-seq or spatial-CITE-seq measurements (mRNA counts
  plus ADT protein counts) must be fused into a single latent space for joint clustering
  or spatial domain detection, e.g. GSE213264 human tonsil (2491 spots, 28,417 genes,
  283 proteins).
when_to_use_negative:
- The analysis requires count-aware modeling of raw UMI counts (dropout events, mean-variance
  relationship) — SMOPCA runs on normalized data and does not model these.
- The data are very sparse and noisy or the spatial structure is very complicated,
  so the lengthscale parameter cannot be learned reliably without manual user tuning.
- Only one modality (RNA or protein alone) is available — there is no second modality
  to fuse; use a single-modality spatial method such as SpatialPCA instead.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0091
tools:
- name: SMOPCA
  role: 'Primary integrator: spatial multi-omics PCA that estimates modality-specific
    parameters (σ_k, W_k) by iterative maximum likelihood with eigen decomposition
    of the kernel matrix, returning the posterior mean of the latent factor as the
    fused gene+protein representation'
- name: SPARK
  role: Identifies spatially variable genes for the RNA modality (Benjamini-Yekutieli
    FDR 0.05, up to 3000 SVGs) prior to fusion with all 283 proteins
- name: sklearn.cluster.KMeans
  role: Clusters the integrated low-dimensional vectors (k=7) to produce the final
    spatial-domain labels
- name: SpatialPCA
  role: Spatially aware dimension-reduction baseline (single RNA modality) benchmarked
    with Moran's I / LISI on the tonsil data
  repo: github.com/shangll123/SpatialPCA
- name: MEFISTO
  role: Multi-omics factor-analysis baseline that natively handles 3+ modalities;
    benchmarked but slower to train and less competitive than SpatialGlue
  repo: https://biofam.github.io/MOFA2/MEFISTO
- name: SpatialGlue
  role: Graph neural network baseline with dual-attention for spatial multi-omics
    integration (at most two modalities)
  repo: https://github.com/JinmiaoChenLab/SpatialGlue/tree/main
- name: SpaVAE
  role: Spatial dependency-aware variational autoencoder baseline (negative binomial
    model with Gaussian process prior) for spatial multi-omics
  repo: github.com/ttgump/spaVAE
- name: scMDC
  role: Deep multimodal clustering baseline for single-cell multi-omics integration
  repo: https://github.com/xianglin226/scMDC
- name: Seurat V4
  role: Single-cell multimodal integration baseline benchmarked on the tonsil spatial-CITE-seq
    data
  repo: https://github.com/satijalab/seurat
- name: TotalVI
  role: Joint probabilistic RNA+protein (scvi-tools) baseline for CITE-seq integration
  repo: https://scvi-tools.org
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:f570aaa3f3d05d463e4bc15f56d097f59c3bf872772a3b78d7ac199ea1b64f97
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/hemberg-lab/SC3
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
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# cite-seq-protein-gene-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Integrates paired CITE-seq gene-expression and antibody-derived protein (ADT) modalities into one shared low-dimensional latent representation — via SMOPCA's spatially aware multi-omics PCA — enabling joint clustering and spatial domain detection that outperforms single-modality and existing multimodal baselines on spatial-CITE-seq data.

## When to use

Use when paired CITE-seq or spatial-CITE-seq measurements (mRNA counts plus ADT protein counts) must be fused into a single latent space for joint clustering or spatial domain detection, e.g. GSE213264 human tonsil (2491 spots, 28,417 genes, 283 proteins).

## When NOT to use

- The analysis requires count-aware modeling of raw UMI counts (dropout events, mean-variance relationship) — SMOPCA runs on normalized data and does not model these.
- The data are very sparse and noisy or the spatial structure is very complicated, so the lengthscale parameter cannot be learned reliably without manual user tuning.
- Only one modality (RNA or protein alone) is available — there is no second modality to fuse; use a single-modality spatial method such as SpatialPCA instead.

## Inputs

- Human tonsil spatial-CITE-seq count matrices (GSE213264): 2491 spots x 28,417 genes and x 283 proteins
- Spot spatial coordinates used to build the spatial kernel
- Optionally, paired single-cell CITE-seq counts (e.g., PBMC: 3762 cells, 33,538 genes, 49 proteins) with pseudo-spatial coordinates from UMAP

## Outputs

- Integrated low-dimensional representation per spot (posterior mean of the latent factor Z)
- Spatial-domain cluster labels from sklearn.cluster.KMeans (k=7)
- Moran's I and LISI benchmark metrics per method written to smopca_tonsil_benchmark.csv
- Marker-based cluster annotation check (CD3/CD4/CD49a for T-cell zone; CD21/CD23/IgM for germinal centers)

## How to apply

Load the paired count matrices (human tonsil spatial-CITE-seq GSE213264: 2491 spots with 28,417 genes and 283 proteins) and first filter out genes and proteins with zero counts. Restrict the RNA modality to spatially variable genes selected by SPARK at a Benjamini-Yekutieli FDR threshold of 0.05 (up to 3000 SVGs) while keeping all 283 protein features, then normalize each modality by library size (total read count), log-transform, and scale to zero mean and unit variance. Fit SMOPCA by iteratively estimating modality-specific parameters (σ_k, W_k) via maximum likelihood with eigen decomposition of the kernel matrix for scalable computation, and take the posterior mean of the latent factor Z as the integrated low-dimensional representation. Cluster these vectors with sklearn.cluster.KMeans (k=7 for the tonsil tissue) to predict spatial domain labels, and judge integration quality by Moran's I (higher is better) and LISI (lower is better) against the baselines SpatialPCA, MEFISTO, SpatialGlue, SpaVAE, scMDC, Seurat, TotalVI, and PCA+K-means. Finally verify biological plausibility via marker checks — CD3/CD4/CD49a upregulated in the T-cell zone cluster (cluster 6) and CD21/CD23/IgM upregulated in germinal-center clusters — and write the metrics and annotation checks to smopca_tonsil_benchmark.csv.

## Related tools

- **SMOPCA** (Primary integrator: spatial multi-omics PCA that estimates modality-specific parameters (σ_k, W_k) by iterative maximum likelihood with eigen decomposition of the kernel matrix, returning the posterior mean of the latent factor as the fused gene+protein representation)
- **SPARK** (Identifies spatially variable genes for the RNA modality (Benjamini-Yekutieli FDR 0.05, up to 3000 SVGs) prior to fusion with all 283 proteins)
- **sklearn.cluster.KMeans** (Clusters the integrated low-dimensional vectors (k=7) to produce the final spatial-domain labels)
- **SpatialPCA** (Spatially aware dimension-reduction baseline (single RNA modality) benchmarked with Moran's I / LISI on the tonsil data) — github.com/shangll123/SpatialPCA
- **MEFISTO** (Multi-omics factor-analysis baseline that natively handles 3+ modalities; benchmarked but slower to train and less competitive than SpatialGlue) — https://biofam.github.io/MOFA2/MEFISTO
- **SpatialGlue** (Graph neural network baseline with dual-attention for spatial multi-omics integration (at most two modalities)) — https://github.com/JinmiaoChenLab/SpatialGlue/tree/main
- **SpaVAE** (Spatial dependency-aware variational autoencoder baseline (negative binomial model with Gaussian process prior) for spatial multi-omics) — github.com/ttgump/spaVAE
- **scMDC** (Deep multimodal clustering baseline for single-cell multi-omics integration) — https://github.com/xianglin226/scMDC
- **Seurat V4** (Single-cell multimodal integration baseline benchmarked on the tonsil spatial-CITE-seq data) — https://github.com/satijalab/seurat
- **TotalVI** (Joint probabilistic RNA+protein (scvi-tools) baseline for CITE-seq integration) — https://scvi-tools.org

## Examples

```
from sklearn.cluster import KMeans; labels = KMeans(n_clusters=7).fit_predict(Z)
```

## Evaluation signals

- SMOPCA should achieve the highest Moran's I score and the lowest LISI score among SpatialPCA, MEFISTO, SpatialGlue, and SpaVAE on the tonsil data, indicating reduced mixing of cell types and more spatially homogeneous clusters.
- Marker invariant: CD3/CD4/CD49a upregulated in the T-cell-zone cluster (cluster 6); CD21/CD23/IgM upregulated in the germinal-center clusters.
- Feature-set invariant: zero-count genes/proteins removed; at most 3000 SPARK SVGs (Benjamini-Yekutieli FDR 0.05) retained plus all 283 proteins; all features scaled to zero mean and unit variance.
- Labels must come from sklearn.cluster.KMeans with k=7 applied to the posterior mean of the latent factor, with metrics recorded in smopca_tonsil_benchmark.csv.

## Limitations

- SMOPCA utilizes normalized data rather than raw expression measurements and does not model dropout events or the inherent mean-variance relationship of raw counts, which may reduce analytical power.
- Current implementation has O(n^3) time and O(n^2) space complexity; the authors aim to reduce both in future work.
- When the data are very sparse and noisy, or the spatial structure is very complicated, learning the appropriate lengthscale parameter may be challenging and require manual tuning.
- A shared covariance assumption (Σ1=...=Σd) is imposed for computational convenience; relaxing it requires orthogonal-constrained (Stiefel manifold) numerical optimization deferred to future work.

## Evidence

- [discussion] Tonsil spatial-CITE-seq dataset dimensions: [evidence span withheld — anchor status: not_found]
- [discussion] Zero-count feature filter applied before integration: "Initially, genes and proteins with zero counts were filtered out."
- [discussion] SPARK SVG selection threshold for the RNA modality: "Gene significance is assessed using a False Discovery Rate (FDR) threshold of 0.05, applying the Benjamini-Yekutieli (BY) procedure"
- [discussion] Normalization protocol per modality: "the count matrix was normalized by library size (the total read count), followed by log transformation and scaling to achieve zero mean and unit variance"
- [discussion] Final clustering step on the integrated vectors: "K-means clustering (“sklearn.cluster.KMeans”) is performed on the low-dimensional vectors to predict the final clustering labels for SMOPCA."
- [discussion] Scalable kernel computation in model fitting: "we perform eigen decomposition on the kernel matrix"
- [abstract] Headline performance claim of the integration: "SMOPCA outperforms existing single-modal and multimodal dimension reduction and clustering methods"
- [discussion] Normalized-input limitation: "our method utilizes normalized data, rather than raw expression measurements, as inputs for dimension reduction analysis"
- [discussion] Lengthscale-learning limitation: "when the data are very sparse and noisy, or the spatial structure is very complicated, it may be challenging to learn the appropriate lengthscale parameter"
- [readme] SpatialPCA baseline definition from its repository: [evidence span withheld — anchor status: external_doc]
