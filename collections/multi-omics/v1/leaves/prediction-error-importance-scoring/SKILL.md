---
name: prediction-error-importance-scoring
description: Use when you have an existing feature-importance or network-inference
  method (e.g., GENIE3) whose raw discoveries lack calibration, and you need theoretically
  valid, FDR-controlled decisions on single-cell expression data.
when_to_use_negative:
- When the response variable or prediction model of interest is not expressible as
  per-cell negative log prediction errors from a fitted predictor
- When you need functional/causal guarantees rather than conditional-dependence calibration
  — VI-VS discoveries have no guarantees to be functional in the general case (feedback
  loops, unobserved species can yield spurious discoveries)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GENIE3
  role: Regression-tree ensemble whose held-out prediction errors serve as the importance
    scores plugged into VI-VS
- name: VIVS
  role: Open-source Python package implementing the VI-VS conditional randomization
    test wrapper with calibrated FDR estimation
  repo: https://github.com/YosefLab/VIVS
- name: Jax
  role: Backend providing parallel computing and just-in-time compilation to speed
    up VI-VS inference
- name: scanpy
  role: Used to import the expression data as an AnnData object before running VIVS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:578b0bb9cfc87013d96100d1a4ec7197a43f15281bfd8e4cab195a1ebcefce4c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VIVS
  tools:
  - GENIE3
  - Jax
  - Python
  - VI-VS
  - VIVS
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# prediction-error-importance-scoring

## Summary

Wraps an existing pairwise-relationship algorithm (e.g., the GENIE3 regression-tree ensemble) inside the VI-VS conditional randomization test, using its held-out prediction errors as importance scores so its heuristic discoveries receive calibrated FDR control on single-cell genomics data.

## When to use

Use when you have an existing feature-importance or network-inference method (e.g., GENIE3) whose raw discoveries lack calibration, and you need theoretically valid, FDR-controlled decisions on single-cell expression data.

## When NOT to use

- When the response variable or prediction model of interest is not expressible as per-cell negative log prediction errors from a fitted predictor
- When you need functional/causal guarantees rather than conditional-dependence calibration — VI-VS discoveries have no guarantees to be functional in the general case (feedback loops, unobserved species can yield spurious discoveries)

## Inputs

- Simulated or real single-cell expression dataset (scanpy AnnData object with raw counts)
- Response variable Y of interest (stored in the AnnData obsm attribute)
- Development/validation cell split for held-out prediction-error computation
- Target FDR levels (several, e.g., for BH correction)

## Outputs

- Per-gene p-values from the VI-VS conditional randomization test
- BH-corrected discovery calls at each target FDR level
- Empirical FDR and TPR summary table (e.g., genie3_fdr_tpr.csv)

## How to apply

Load the single-cell expression data (with a synthetic or observed response) as a scanpy AnnData object and split cells into development and validation sets. Train the GENIE3 regression-tree ensemble on the development split to predict the response from gene expression, then evaluate on the held-out validation split, computing per-cell negative log prediction errors T_phi(X, Y, S) = (1/N) * sum -log p_phi(y^n | x^n, s^n) as the importance score. Run the VI-VS conditional randomization test with K Monte Carlo randomized replicates (VIVS package, Jax backend) to obtain a p-value per gene, then apply Benjamini-Hochberg correction and call discoveries at several target FDR levels. The rationale is that VI-VS provides a calibrated (theoretically proven) FDR estimate regardless of the response distribution or interaction structure, so plugging in GENIE3's prediction errors converts heuristic importance into calibrated decisions.

## Related tools

- **GENIE3** (Regression-tree ensemble whose held-out prediction errors serve as the importance scores plugged into VI-VS)
- **VIVS** (Open-source Python package implementing the VI-VS conditional randomization test wrapper with calibrated FDR estimation) — https://github.com/YosefLab/VIVS
- **Jax** (Backend providing parallel computing and just-in-time compilation to speed up VI-VS inference)
- **scanpy** (Used to import the expression data as an AnnData object before running VIVS)

## Examples

```
from vivs import VIVS; model = VIVS(adata, feature_obsm_key="my_obsm_key", xy_linear=False, xy_model_kwargs={"n_hidden": 8}); model.train_all()
```

## Evaluation signals

- Empirical FDR at each target level should not exceed the nominal threshold after BH correction
- VI-VS p-values under the null should stochastically dominate the uniform distribution (calibration check)
- True positive rate should remain large at the controlled FDR levels (as reported for the simulated 100-gene dataset)
- Held-out prediction errors must be computed on a validation split distinct from the GENIE3 training split

## Limitations

- VI-VS discoveries have no guarantees to be functional in the general case; feedback loops, cell communication, or unobserved molecular species can lead to spurious discoveries
- VI-VS may not scale to many thousands of genes — gene filtering (e.g., select_genes with n_top_genes=2000) is recommended before running
- Spurious discoveries from the wrapped method's prediction errors are corrected for calibration but not eliminated when conditional-dependence structure is mis-specified

## Evidence

- [intro] wrapper framework: "our procedure provides a theoretically grounded “wrapper” framework that can take existing algorithms for detecting pairwise relationships"
- [discussion] meta-algorithm: "VI-VS can be employed as a meta-algorithm to make the discoveries of existing methods more interpretable by constructing importance scores"
- [methods] Jax backend: "Our implementation first relies on parallel computing and just-in-time compilation components of Jax to speed up the inference"
- [abstract] calibrated FDR: "It uses deep generative models to identify conditionally dependent features, with false discovery rate control."
- [readme] AnnData input: "To use VIVS in your project, import the data of intest in a scanpy AnnData object."
