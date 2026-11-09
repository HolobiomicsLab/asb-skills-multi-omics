---
name: synthetic-response-simulation-single-cell
description: Use when benchmarking a feature-selection or conditional-independence
  method (e.g., VI-VS, GENIE3 wrappers) on single-cell expression data where empirical
  FDR calibration must be verified against known ground-truth dependencies.
when_to_use_negative:
- The goal is biological discovery on real responses — synthetic responses validate
  calibration/power but VI-VS discoveries on real data carry no functional guarantees
  in the general case
- The data are already normalized rather than raw counts — VIVS requires raw counts
  in the AnnData object
- The dataset has too many genes for VIVS to scale (VIVS may not scale to many thousands
  of genes without prior filtering via select_genes)
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: VIVS
  role: Conditional randomization test run on the semi-synthetic data to obtain per-gene
    p-values whose FDR calibration is evaluated against the known ground truth
  repo: github.com/YosefLab/VIVS
- name: scanpy
  role: Provides the AnnData object format for loading the single-cell expression
    dataset with raw counts
- name: Jax
  role: Backend for the VIVS implementation providing parallel computing and just-in-time
    compilation
- name: GENIE3
  role: Regression-tree ensemble whose prediction errors serve as importance scores
    in the VI-VS wrapper for a second semi-synthetic benchmark
- name: scVI
  role: Generative model whose normalized gene expression can serve as features in
    the VI-VS framework
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:ca6f7cdfdf981e9ce1fa6caaf1b65f36172b9bd09c0e67e8dcb52cf069c8d461
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GENIE3
  - Jax
  - Python
  - VI-VS
  - VIVS
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7368421052631579
n_claims: 19
---

# synthetic-response-simulation-single-cell

## Summary

Constructs semi-synthetic single-cell benchmark responses with known ground-truth conditional dependencies by generating synthetic response variables from sampled genes plus added noise, enabling empirical validation of false discovery rate (FDR) control and true positive rate (TPR) for conditional independence testing methods such as VI-VS.

## When to use

Use when benchmarking a feature-selection or conditional-independence method (e.g., VI-VS, GENIE3 wrappers) on single-cell expression data where empirical FDR calibration must be verified against known ground-truth dependencies.

## When NOT to use

- The goal is biological discovery on real responses — synthetic responses validate calibration/power but VI-VS discoveries on real data carry no functional guarantees in the general case
- The data are already normalized rather than raw counts — VIVS requires raw counts in the AnnData object
- The dataset has too many genes for VIVS to scale (VIVS may not scale to many thousands of genes without prior filtering via select_genes)

## Inputs

- Single-cell gene expression dataset with raw counts as a scanpy AnnData object (e.g., PBMC 6,855-cell dataset)
- Number of genes to sample as predictors (e.g., 150 random genes)
- Noise model specification for synthetic responses (e.g., Gaussian noise)
- Number of synthetic response replicates (e.g., five responses y)

## Outputs

- Synthetic response variables y with known ground-truth conditional dependencies
- Empirical FDR and TPR summary table per method and per target FDR level (e.g., fdr_tpr_table.csv, genie3_fdr_tpr.csv)
- Per-gene p-values after conditional randomization testing and Benjamini-Hochberg correction

## How to apply

Load a single-cell dataset (e.g., the PBMC 6,855-cell dataset) as a scanpy AnnData object containing raw gene expression counts. Sample a subset of genes (e.g., 150 random genes) and compute transformed features such as squared log-CPM values to serve as predictor variables. Generate synthetic response variables y from the selected genes by adding Gaussian noise, so the ground-truth conditional dependencies are known by construction — genes used to construct y are true positives, all others are true nulls. Run the method under evaluation (e.g., VI-VS with an MLP importance score via xy_linear=False, xy_model_kwargs={'n_hidden': 8}, or a GENIE3 prediction-error importance score) followed by Benjamini-Hochberg correction at several target FDR levels, and repeat across multiple replicates (e.g., five responses). Compute empirical FDR (false discoveries / total calls) and TPR (true discoveries / total positives) per method per target level, and verify the method under test is calibrated — empirical FDR does not exceed the nominal target — while baselines (OLS t-test, marginal independence test) may be anti-conservative or conservative due to violated linearity/Gaussian assumptions.

## Related tools

- **VIVS** (Conditional randomization test run on the semi-synthetic data to obtain per-gene p-values whose FDR calibration is evaluated against the known ground truth) — github.com/YosefLab/VIVS
- **scanpy** (Provides the AnnData object format for loading the single-cell expression dataset with raw counts)
- **Jax** (Backend for the VIVS implementation providing parallel computing and just-in-time compilation)
- **GENIE3** (Regression-tree ensemble whose prediction errors serve as importance scores in the VI-VS wrapper for a second semi-synthetic benchmark)
- **scVI** (Generative model whose normalized gene expression can serve as features in the VI-VS framework)

## Examples

```
from vivs import VIVS; model = VIVS(adata, feature_obsm_key='my_obsm_key', xy_linear=False, xy_model_kwargs={'n_hidden': 8}); model.train_all()
```

## Evaluation signals

- Output table (e.g., fdr_tpr_table.csv) exists and contains rows for all methods under test and all target FDR levels with FDR and TPR columns
- Empirical FDR of the method under test does not exceed the nominal target FDR level (calibration), consistent with p-values stochastically dominating the uniform distribution under the null
- TPR is reported alongside FDR to confirm the method retains power (true discoveries / total positives)
- Baseline behavior is reproduced: marginal independence test exceeds target FDR levels and OLS t-test overestimates FDR, consistent with violated linearity/Gaussian assumptions
- Results are consistent across the five synthetic response replicates

## Limitations

- VI-VS discoveries have no guarantees to be functional in the general case; feedback loops, cell communication, or unobserved molecular species can lead to spurious discoveries even when FDR is calibrated
- VIVS may not scale to many thousands of genes; genes should be filtered (e.g., select_genes with n_top_genes=2000) before running
- Gaussian-noise synthetic responses validate statistical calibration, not biological relevance, and assume the semi-synthetic ground truth mirrors real dependency structure

## Evidence

- [abstract] It uses deep generative models to identify conditionally dependent features, with false discovery rate control.: "It uses deep generative models to identify conditionally dependent features, with false discovery rate control."
- [intro] We demonstrate and theoretically prove that our procedure provides a calibrated estimation of the false discovery rate.: "We demonstrate and theoretically prove that our procedure provides a calibrated estimation of the false discovery rate."
- [readme] To use VIVS in your project, import the data of intest in a scanpy AnnData object.: "To use VIVS in your project, import the data of intest in a scanpy AnnData object."
- [discussion] In the general case, VI-VS discoveries have no guarantees to be functional.: "In the general case, VI-VS discoveries have no guarantees to be functional."
- [readme] VIVS may not scale to many thousands of genes.: "VIVS may not scale to many thousands of genes."
