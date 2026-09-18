---
name: spatial-transcriptomics-simulation
description: Use when a benchmark study needs controlled spatial multi-omics data
  with ground-truth spatial domain labels, e.g., simulating 3-omics DLPFC sections
  to compare SMOPCA against SpatialPCA, SpaVAE, or SpatialGlue_3M.
when_to_use_negative:
- You already possess real spatial multi-omics measurements with spatial coordinates
  and ground-truth or manually annotated domains — run the spatial domain detection
  method directly on the real data instead of simulating.
- The analysis target is single-cell (non-spatial) multi-omics data without pseudo-spatial
  coordinates, where the simulation-based spatial benchmark is irrelevant.
- The research question concerns real-data performance such as runtime or model limitations
  (e.g., O(n^3) time complexity) rather than controlled comparison of clustering accuracy
  against known spatial domains.
tools:
- name: SRTsim
  role: 'Core simulator: generates simulated SRT/spatial multi-omics data that maintains
    the expression characteristics of the reference data while preserving spatial
    patterns'
- name: Seurat
  role: 'Preprocessing of simulated count matrices: LogNormalize (library-size normalization,
    log transformation, scaling to zero mean and unit variance), SCTransform, and
    VST, plus HVG selection'
  repo: https://github.com/satijalab/seurat
- name: SPARK
  role: Calls spatially variable genes at FDR 0.05 with the Benjamini-Yekutieli procedure
    as an alternative simulated gene set to HVGs
- name: SpatialGlue_3M
  role: Downstream benchmark method run on the simulated three-omics sections (extended
    SpatialGlue accommodating three modalities)
  repo: github.com/JinmiaoChenLab/SpatialGlue_3M
- name: scikit-learn KMeans
  role: Clusters low-dimensional vectors from SMOPCA on simulated sections to produce
    spatial domain labels for AMI/ARI scoring
merged_aliases:
- spatial-transcriptomics-data-simulation
merged_alias_records:
- alias: spatial-transcriptomics-data-simulation
  slug: spatial-transcriptomics-data-simulation
  jaccard_score: 0.75
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1186/s13059-025-03576-9
    title: 'SMOPCA: spatially aware dimension reduction integrating multi-omics improves
      the efficiency of spatial domain detection'
  source_content_hash: sha256:26add55e7f222c245b5b8b754cc6af143b3a3808ace3464955c47a8fea2db375
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
  - SMOPCA
  - SPARK
  - SRTsim
  - Seurat
  - SpaVAE
  - SpatialGlue_3M
  - SpatialPCA
  - scikit-learn
  - scikit-learn KMeans
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# spatial-transcriptomics-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates synthetic spatial transcriptomics / spatial multi-omics datasets with SRTsim from a real SRT reference section (e.g., LIBD human DLPFC), producing simulated count matrices that preserve the reference's expression characteristics and spatial patterns for benchmarking spatial domain detection methods like SMOPCA.

## When to use

Use when a benchmark study needs controlled spatial multi-omics data with ground-truth spatial domain labels, e.g., simulating 3-omics DLPFC sections to compare SMOPCA against SpatialPCA, SpaVAE, or SpatialGlue_3M.

## When NOT to use

- You already possess real spatial multi-omics measurements with spatial coordinates and ground-truth or manually annotated domains — run the spatial domain detection method directly on the real data instead of simulating.
- The analysis target is single-cell (non-spatial) multi-omics data without pseudo-spatial coordinates, where the simulation-based spatial benchmark is irrelevant.
- The research question concerns real-data performance such as runtime or model limitations (e.g., O(n^3) time complexity) rather than controlled comparison of clustering accuracy against known spatial domains.

## Inputs

- LIBD human DLPFC reference sections (151507-151510, 151669-151676) with count matrices and spatial coordinates
- Filtered gene expression matrix restricted to the top 5000 HVGs per DLPFC section
- Optionally, SPARK spatially variable genes called at FDR 0.05 (Benjamini-Yekutieli procedure) as an alternative gene set

## Outputs

- SRTsim-simulated count matrices preserving the reference section's spatial patterns (three simulated omics matrices per section; 36 total across 12 sections)
- Simulated ground-truth spatial domain labels used for AMI/ARI scoring of clustering methods
- Per-method AMI comparison table across simulated sections with paired t-test statistic and p-value (e.g., ami_comparison_dlpfc.csv)

## How to apply

Select a reference SRT section with annotated spatial domains — the paper uses the 12 LIBD human DLPFC sections (151507–151510, 151669–151676) — and filter the count matrix, selecting the top 5000 highly variable genes (HVGs) per section as SRTsim input. Run SRTsim to generate simulated data that maintains the expression characteristics of the SRT data while preserving spatial patterns; for a 3-omics benchmark, generate three simulated omics count matrices per section (36 matrices total across 12 sections). Preprocess the simulated counts as needed for the benchmark (library-size normalization with log transformation and scaling to zero mean and unit variance, or SCTransform/VST), optionally deriving SPARK spatially variable genes at FDR 0.05 with the Benjamini-Yekutieli procedure as an alternative to HVGs. Run the methods under comparison (SMOPCA with a Matern kernel at nu=0.5/1.5/2.5 plus eigen decomposition, SpatialPCA, SpaVAE, PCA + sklearn K-means, or SpatialGlue_3M for three modalities) on each simulated section and score clusterings against the simulated ground-truth domains using AMI or ARI. The simulation design also enables sensitivity analysis across kernel parameters, normalization methods, and gene-selection strategies (18 combinations), which the paper reports leaves SMOPCA's clustering performance robust.

## Related tools

- **SRTsim** (Core simulator: generates simulated SRT/spatial multi-omics data that maintains the expression characteristics of the reference data while preserving spatial patterns)
- **Seurat** (Preprocessing of simulated count matrices: LogNormalize (library-size normalization, log transformation, scaling to zero mean and unit variance), SCTransform, and VST, plus HVG selection) — https://github.com/satijalab/seurat
- **SPARK** (Calls spatially variable genes at FDR 0.05 with the Benjamini-Yekutieli procedure as an alternative simulated gene set to HVGs)
- **SpatialGlue_3M** (Downstream benchmark method run on the simulated three-omics sections (extended SpatialGlue accommodating three modalities)) — github.com/JinmiaoChenLab/SpatialGlue_3M
- **scikit-learn KMeans** (Clusters low-dimensional vectors from SMOPCA on simulated sections to produce spatial domain labels for AMI/ARI scoring)

## Evaluation signals

- Three simulated omics count matrices are produced per reference section (36 total across the 12 DLPFC sections), matching the simulation design
- Simulated data reproduces the reference section's expression characteristics and spatial patterns (as asserted by SRTsim's design and confirmed by summary statistics/visual inspection)
- Downstream clustering on simulated sections yields AMI/ARI scores against the simulated ground-truth spatial domains; SMOPCA's AMI advantage over baselines should be statistically significant under a paired t-test across the 12 sections
- Robustness analysis across 18 (normalization x gene-selection x kernel) combinations should show negligible variation in clustering performance, consistent with Additional file 1: Fig. S22-S31
- Aggregated results are written to a machine-readable table (e.g., ami_comparison_dlpfc.csv or robustness_results.csv)

## Limitations

- Simulation fidelity depends on the chosen reference section; only 5000 HVGs per section are carried into SRTsim, so lowly expressed or non-variable genes are absent from the synthetic data
- Simulated benchmarks preserve the reference's expression characteristics and spatial patterns but may not capture artifacts of real multi-omics assays (e.g., modality-specific noise or dropouts)
- Sensitivity conclusions are drawn from simulated data mirroring Additional file 1: Fig. S22-S31; robustness on real tissues with complex spatial structure is not established by this simulation alone

## Evidence
