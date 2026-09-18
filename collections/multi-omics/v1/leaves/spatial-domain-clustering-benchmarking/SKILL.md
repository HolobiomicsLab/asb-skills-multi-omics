---
name: spatial-domain-clustering-benchmarking
description: Use when comparing spatial-domain/clustering performance of multi-omics
  integration methods on CITE-seq, SMAGE-seq, spatial-CITE-seq, MISAR-seq, or SRTsim-simulated
  data with ground-truth spatial-domain labels.
when_to_use_negative:
- The data are raw counts that must be modeled directly — SMOPCA uses normalized data
  rather than raw expression measurements, so count-aware deep generative alternatives
  are the wrong comparison target for raw-count inputs
- Only a single feature/embedding table is already available with no spatial coordinates
  or ground-truth domain labels — there is nothing to benchmark spatial smoothness
  or clustering accuracy against
- 'More than two modalities are required from SpaVAE or SpatialGlue: these ''could
  only process at most two modalities'' and would need redesign (use SpatialGlue_3M
  or MEFISTO instead)'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SMOPCA
  role: Spatial multi-omics PCA method under evaluation; eigen decomposition of the
    spatial kernel matrix, iterative MLE of (sigma_k, W_k), posterior mean of Z as
    integrated representation
- name: SpatialPCA
  role: Baseline; spatially aware dimension reduction applied to concatenated modalities
    or gene expression with a kernel matrix modeling spatial correlation
  repo: https://github.com/shangll123/SpatialPCA
- name: SpatialGlue_3M
  role: Baseline extended from SpatialGlue to accommodate three modalities in the
    12-section SRTsim simulation benchmark
  repo: https://github.com/JinmiaoChenLab/SpatialGlue_3M
- name: scMDC
  role: Single-cell multi-omics deep clustering baseline run on the same preprocessed
    features
  repo: https://github.com/xianglin226/scMDC
- name: BREM-SC
  role: Baseline joint clustering method for CITE-seq comparison
  repo: https://github.com/tarot0410/BREMSC
- name: Seurat V4
  role: Preprocessing (LogNormalize, SCTransform, VST) and multimodal baseline clustering
  repo: https://github.com/satijalab/seurat
- name: TotalVI
  role: Probabilistic transcriptome-proteome integration baseline
  repo: https://scvi-tools.org
- name: MEFISTO
  role: Baseline supporting 3+ modalities natively in the tonsil spatial-CITE-seq
    benchmark
  repo: https://biofam.github.io/MOFA2/MEFISTO
- name: scikit-learn KMeans
  role: Final clustering on low-dimensional vectors to predict spatial-domain labels
    for SMOPCA and the PCA baseline
merged_aliases:
- spatial-domain-clustering-evaluation
- spatial-domain-clustering-kmeans
- spatial-domain-clustering-metric-computation
merged_alias_records:
- alias: spatial-domain-clustering-evaluation
  slug: spatial-domain-clustering-evaluation
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
- alias: spatial-domain-clustering-kmeans
  slug: spatial-domain-clustering-kmeans
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
- alias: spatial-domain-clustering-metric-computation
  slug: spatial-domain-clustering-metric-computation
  cosine_score: 0.8442
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_008
  - task_006
  - task_007
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:2bfc3cac51cccf70bc62e09a2486766db91701f1dd12af34d03d0dd6a80325cc
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
  - BREM-SC
  - MEFISTO
  - SMOPCA
  - SPARK
  - SRTsim
  - Seurat V4
  - SpaVAE
  - SpatialGlue
  - SpatialGlue_3M
  - SpatialPCA
  - TotalVI
  - scMDC
  - scikit-learn
  - scikit-learn KMeans
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# spatial-domain-clustering-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark spatial-domain detection methods (SMOPCA, SpatialPCA, SpatialGlue, SpaVAE, MEFISTO, scMDC, Seurat, TotalVI, PCA+KMeans) across simulated and real spatial multi-omics datasets, scoring clusterings with AMI, NMI, ARI, Moran's I, and LISI.

## When to use

Use when comparing spatial-domain/clustering performance of multi-omics integration methods on CITE-seq, SMAGE-seq, spatial-CITE-seq, MISAR-seq, or SRTsim-simulated data with ground-truth spatial-domain labels.

## When NOT to use

- The data are raw counts that must be modeled directly — SMOPCA uses normalized data rather than raw expression measurements, so count-aware deep generative alternatives are the wrong comparison target for raw-count inputs
- Only a single feature/embedding table is already available with no spatial coordinates or ground-truth domain labels — there is nothing to benchmark spatial smoothness or clustering accuracy against
- More than two modalities are required from SpaVAE or SpatialGlue: these 'could only process at most two modalities' and would need redesign (use SpatialGlue_3M or MEFISTO instead)

## Inputs

- CITE-seq count matrices (PBMC: 3762 cells, 33,538 genes, 49 proteins; SLN111D1/D2, SLN208D1/D2)
- SMAGE-seq count matrices (PBMC3K, PBMC10K)
- LIBD human DLPFC 12 sections (151507-151510, 151669-151676) with 5000 HVGs per section
- Human tonsil spatial-CITE-seq data (GSE213264; 2491 spots, 28,417 genes, 283 proteins)
- MISAR-seq E15.5 mouse brain spatial ATAC-RNA data (1949 of 2500 spots retained; peak matrix collapsed to gene activity matrix per Satija lab protocol)
- Ground-truth cell-type / spatial-domain labels

## Outputs

- Per-method AMI/NMI/ARI metric tables (e.g., simulated_spatial_clustering_metrics.csv, benchmark_results.csv)
- AMI comparison table with paired t-test statistic and p-value (ami_comparison_dlpfc.csv)
- Moran's I and LISI scores with cluster-annotation checks (smopca_tonsil_benchmark.csv)
- Robustness aggregation over kernel/normalization/gene-selection settings (robustness_results.csv)

## How to apply

Preprocess each modality by library-size normalization, log transformation, and scaling to zero mean and unit variance; select spatially variable genes with SPARK at a Benjamini-Yekutieli FDR threshold of 0.05 (up to 3000 SVGs, or 5000 HVGs per DLPFC section for SRTsim simulation input). Fit SMOPCA by performing eigen decomposition on the kernel matrix (Matern nu = 0.5, 1.5, 2.5), iteratively estimating modality-specific parameters (sigma_k, W_k) with gamma updated across modalities, and taking the posterior mean of latent Z as the integrated representation. Predict final labels with sklearn.cluster.KMeans on the low-dimensional vectors (e.g., k=7 for the human tonsil spatial-CITE-seq data with 2491 spots, 28,417 genes, 283 proteins). Run baselines — SpatialPCA on concatenated modalities, scMDC, SpatialGlue/SpatialGlue_3M, SpaVAE, MEFISTO, chromVAR, cisTopic, LSA, PeakVI, SCALE, Seurat, TotalVI, PCA+KMeans — on identical inputs, then compute AMI, NMI, ARI against reference labels; for the tonsil data additionally compute Moran's I and LISI to quantify spatial smoothness, and verify cluster annotations against markers such as CD3/CD4/CD49a (T-cell zone) and CD21/CD23/IgM (germinal center). Repeat simulations (e.g., 10 replicates per dataset; 12 SRTsim-simulated DLPFC sections) and assess significance with a paired t-test.

## Related tools

- **SMOPCA** (Spatial multi-omics PCA method under evaluation; eigen decomposition of the spatial kernel matrix, iterative MLE of (sigma_k, W_k), posterior mean of Z as integrated representation)
- **SpatialPCA** (Baseline; spatially aware dimension reduction applied to concatenated modalities or gene expression with a kernel matrix modeling spatial correlation) — https://github.com/shangll123/SpatialPCA
- **SpatialGlue_3M** (Baseline extended from SpatialGlue to accommodate three modalities in the 12-section SRTsim simulation benchmark) — https://github.com/JinmiaoChenLab/SpatialGlue_3M
- **scMDC** (Single-cell multi-omics deep clustering baseline run on the same preprocessed features) — https://github.com/xianglin226/scMDC
- **BREM-SC** (Baseline joint clustering method for CITE-seq comparison) — https://github.com/tarot0410/BREMSC
- **Seurat V4** (Preprocessing (LogNormalize, SCTransform, VST) and multimodal baseline clustering) — https://github.com/satijalab/seurat
- **TotalVI** (Probabilistic transcriptome-proteome integration baseline) — https://scvi-tools.org
- **MEFISTO** (Baseline supporting 3+ modalities natively in the tonsil spatial-CITE-seq benchmark) — https://biofam.github.io/MOFA2/MEFISTO
- **scikit-learn KMeans** (Final clustering on low-dimensional vectors to predict spatial-domain labels for SMOPCA and the PCA baseline)

## Examples

```
from sklearn.cluster import KMeans; labels = KMeans(n_clusters=7).fit_predict(smopca_posterior_mean_Z)
```

## Evaluation signals

- SMOPCA achieves higher AMI, NMI, and ARI than SpatialPCA (on concatenated modalities), scMDC, PCA+KMeans, and single-modality baselines on simulated single-cell datasets
- Paired t-test across the 12 SRTsim-simulated DLPFC sections shows SMOPCA's AMI improvement is statistically significant
- On tonsil spatial-CITE-seq, SMOPCA yields the highest Moran's I and lowest LISI among SpatialPCA, MEFISTO, SpatialGlue, and SpaVAE
- Robustness check: clustering performance varies negligibly across Matern nu = 0.5/1.5/2.5, LogNormalize/SCTransform/VST, and SVG-vs-HVG gene selection (18 combinations)
- Cluster annotation consistency: CD3/CD4/CD49a upregulated in the T-cell zone cluster (cluster 6) and CD21/CD23/IgM upregulated in germinal-center clusters

## Limitations

- SMOPCA's implementation has O(n^3) time and O(n^2) space complexity; reducing both is future work
- The method does not model dropout events or the mean-variance relationship of raw counts and uses normalized data as input, which may reduce analytical power
- Learning the kernel lengthscale may be difficult when data are sparse/noisy or spatial structure is complicated; manual tuning may be required
- SpaVAE and SpatialGlue (without the _3M extension) process at most two modalities; MEFISTO handles 3+ but trains significantly longer and is less competitive

## Evidence

- [readme] SpatialPCA baseline description: [evidence span withheld — anchor status: external_doc]
