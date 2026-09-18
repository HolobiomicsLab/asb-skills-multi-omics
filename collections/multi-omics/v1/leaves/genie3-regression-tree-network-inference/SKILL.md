---
name: genie3-regression-tree-network-inference
description: Use when you have a single-cell expression dataset (scanpy AnnData with
  raw counts) plus a response variable, and you want to score gene-response dependencies
  via GENIE3 regression trees and feed those scores into calibrated conditional independence
  testing.
when_to_use_negative:
- Your AnnData contains normalized rather than raw counts — VIVS 'only supports assays
  where X corresponds to gene expression counts' with raw counts
- Your dataset has many thousands of genes — the README warns 'VIVS may not scale
  to many thousands of genes'; pre-filter with select_genes first or use a different
  approach
- You need biologically functional interpretation of the discoveries — the paper states
  'In the general case, VI-VS discoveries have no guarantees to be functional' and
  feedback loops or unobserved species can cause spurious discoveries
tools:
- name: GENIE3
  role: Regression-tree-based network inference engine whose prediction errors are
    used as importance scores inside the VI-VS wrapper
- name: VIVS
  role: Python package running the conditional randomization test that calibrates
    GENIE3-derived importance scores into p-values
  repo: github.com/YosefLab/VIVS
- name: Jax
  role: Backend providing parallel computing and just-in-time compilation to speed
    up VIVS inference
- name: scanpy
  role: 'Data container: imports the gene-by-cell data of interest as a scanpy AnnData
    object with raw counts'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:38a0370054504b67873f03fc17021a0a90fe3e739a8b4278f197f544cc44752a
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# genie3-regression-tree-network-inference

## Summary

Uses the GENIE3 regression-tree ensemble to predict a response variable from single-cell gene expression, yielding per-cell prediction errors that serve as importance scores for VI-VS conditional randomization testing. In the Boyeau et al. (2024) framework, GENIE3 is the pairwise-detection engine wrapped by VI-VS to produce calibrated, FDR-controlled decisions.

## When to use

Use when you have a single-cell expression dataset (scanpy AnnData with raw counts) plus a response variable, and you want to score gene-response dependencies via GENIE3 regression trees and feed those scores into calibrated conditional independence testing.

## When NOT to use

- Your AnnData contains normalized rather than raw counts — VIVS 'only supports assays where X corresponds to gene expression counts' with raw counts
- Your dataset has many thousands of genes — the README warns 'VIVS may not scale to many thousands of genes'; pre-filter with select_genes first or use a different approach
- You need biologically functional interpretation of the discoveries — the paper states 'In the general case, VI-VS discoveries have no guarantees to be functional' and feedback loops or unobserved species can cause spurious discoveries

## Inputs

- Single-cell expression dataset as a scanpy AnnData object with raw (unnormalized) counts
- Synthetic or measured response variable Y stored in the AnnData obsm attribute (array or dataframe)
- Cell-level covariates/conditioning set S (e.g. technical nuisance factors)
- Development/validation cell split

## Outputs

- Per-cell negative log prediction errors from the fitted GENIE3 model (importance scores)
- Per-gene p-values from the VI-VS conditional randomization test
- BH-adjusted discoveries at multiple target FDR levels
- Empirical FDR and TPR summary table (e.g. genie3_fdr_tpr.csv)

## How to apply

Load the gene-by-cell expression data as a scanpy AnnData object and split cells into a development set (for fitting) and a validation set (for error evaluation). Train the GENIE3 regression-tree ensemble on the development split to predict the response from gene expression, then evaluate the fitted model on the held-out validation split and compute the per-cell negative log prediction error T_phi(X, Y, S) = (1/N) * sum -log p_phi(y^n | x^n, s^n) as the importance score. Pass these prediction-error scores into the VI-VS conditional randomization test (K Monte Carlo randomized replicates, e.g. Jax backend) so that GENIE3's heuristic relevance ranking is converted into calibrated p-values. Correct the resulting per-gene p-values with Benjamini-Hochberg and call discoveries at several target FDR levels. The rationale is that VI-VS acts as a theoretically grounded wrapper: raw GENIE3 rankings carry no statistical calibration, but under the null the VI-VS p-values stochastically dominate the uniform distribution, so empirical FDR should not exceed nominal targets.

## Related tools

- **GENIE3** (Regression-tree-based network inference engine whose prediction errors are used as importance scores inside the VI-VS wrapper)
- **VIVS** (Python package running the conditional randomization test that calibrates GENIE3-derived importance scores into p-values) — github.com/YosefLab/VIVS
- **Jax** (Backend providing parallel computing and just-in-time compilation to speed up VIVS inference)
- **scanpy** (Data container: imports the gene-by-cell data of interest as a scanpy AnnData object with raw counts)

## Evaluation signals

- Empirical FDR at each target level must not exceed the nominal threshold
- VI-VS p-values under the null should stochastically dominate the uniform distribution (calibrated FDR estimation)
- True positive rates should remain large at the target FDR levels (as reported on the simulated 100-gene dataset)
- Result table (e.g. genie3_fdr_tpr.csv) contains empirical FDR and TPR values for each tested target level

## Limitations

- VI-VS discoveries have no guarantees to be functional in the general case; feedback loops, cell communication, or unobserved molecular species can lead to spurious discoveries
- VIVS may not scale to many thousands of genes, so genes should be filtered (e.g. select_genes with n_top_genes=2000) before running
- No software version information is provided for the generative model or VI-VS implementation, hindering exact reproducibility

## Evidence

- [intro] GENIE3 as demonstrated wrapper: "We demonstrate this using the popular GENIE-3 algorithm for inference of regulatory networks."
- [intro] wrapper framework rationale: "our procedure provides a theoretically grounded “wrapper” framework that can take existing algorithms for detecting pairwise relationships and use them to output calibrated decisions"
- [discussion] meta-algorithm interpretation: "VI-VS can be employed as a meta-algorithm to make the discoveries of existing methods more interpretable by constructing importance scores from their predictions."
- [discussion] GENIE3 calibration via VI-VS: "In this work, we calibrated GENIE3 discoveries via VI-VS but other models could be used instead."
