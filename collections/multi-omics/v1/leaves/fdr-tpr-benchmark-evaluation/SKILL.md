---
name: fdr-tpr-benchmark-evaluation
description: Use when a conditional independence or feature-selection method claims
  FDR control and you need to quantify its empirical FDR and TPR against a ground
  truth (e.g. t-tests of perturbed vs unperturbed cells) and compare to OLS and marginal-test
  baselines.
when_to_use_negative:
- When no independent ground truth for feature relevance exists — FDR/TPR cannot be
  estimated without it, and VI-VS discoveries 'have no guarantees to be functional
  in the general case'
- When the goal is calibration of a single method's p-values rather than comparative
  benchmarking — use the VI-VS CRT calibration procedure itself instead
- When the response data are normalized rather than raw counts, since VIVS requires
  AnnData containing raw counts, invalidating comparability with the published setup
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: VI-VS
  role: 'Method under test: supplies conditional randomization test p-values for feature
    selection with FDR control'
  repo: https://github.com/YosefLab/VIVS
- name: VIVS
  role: Python package (Jax backend) used to run the VI-VS conditional randomization
    test with K Monte Carlo replicates and fixed-posterior sampling
  repo: https://github.com/YosefLab/VIVS
- name: scanpy
  role: Loads the perturbational screen data as an AnnData object with raw counts
    and guide annotations
- name: scVI
  role: Deep generative model trained on expression counts to enable the fixed-posterior
    sampling scheme of the CRT
- name: Jax
  role: Parallel computing and just-in-time compilation backend speeding up VI-VS
    inference
- name: PyTorch
  role: Alternative backend for the VIVS implementation, noted as roughly half the
    speed of the Jax backend
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:e48edf3e8fb41abde380c172a81d07bda14f4e16137bbf31bff81d5e1e15df97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VIVS
  tools:
  - Jax
  - PyTorch
  - Python
  - VI-VS
  - VIVS
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# fdr-tpr-benchmark-evaluation

## Summary

Benchmark protocol for comparing calibrated variable-selection procedures (e.g. VI-VS) against baselines such as OLS regression and marginal independence tests by estimating false discovery rate and true positive rate against an externally defined ground truth, as done on a Perturb-seq dataset at target FDR alpha=0.1.

## When to use

Use when a conditional independence or feature-selection method claims FDR control and you need to quantify its empirical FDR and TPR against a ground truth (e.g. t-tests of perturbed vs unperturbed cells) and compare to OLS and marginal-test baselines.

## When NOT to use

- When no independent ground truth for feature relevance exists — FDR/TPR cannot be estimated without it, and VI-VS discoveries 'have no guarantees to be functional in the general case'
- When the goal is calibration of a single method's p-values rather than comparative benchmarking — use the VI-VS CRT calibration procedure itself instead
- When the response data are normalized rather than raw counts, since VIVS requires AnnData containing raw counts, invalidating comparability with the published setup

## Inputs

- Perturb-seq / perturbational screen dataset as a scanpy AnnData object with raw gene expression counts and per-cell guide/perturbation annotations
- Ground-truth positive feature list from two-sample t-tests (perturbed vs unperturbed cells) for the ten most-guided target genes
- Per-method p-values or ranked feature selections (VI-VS CRT p-values, OLS, marginal test)

## Outputs

- Per-method table of estimated FDR and TPR (perturbational_screen_fdr_tpr.csv)
- Benjamini-Hochberg-adjusted discoveries at target FDR alpha=0.1 for each method
- Comparison statement of precision-vs-power trade-offs between methods

## How to apply

Define ground-truth positives independently of the methods under test — in the paper's Perturb-seq benchmark, two-sample t-tests comparing perturbed versus unperturbed cells for the ten target genes with the largest number of guides. Run each method (VI-VS conditional randomization test p-values, OLS, and a marginal independence test) on the same response and apply the Benjamini-Hochberg procedure at the target FDR level (alpha=0.1) to each method's ranked p-values to call discoveries. Compute estimated FDR and TPR per method against the t-test ground truth and record the per-method comparison in a table (e.g. perturbational_screen_fdr_tpr.csv). Interpret results by both precision and power: in the paper, VI-VS achieved ~70% FDR vs ~95% for OLS while matching OLS's true positive count, showing improved precision without reduced power. The key validation invariant is that a calibrated procedure's estimated FDR should stay at or below the nominal threshold.

## Related tools

- **VI-VS** (Method under test: supplies conditional randomization test p-values for feature selection with FDR control) — https://github.com/YosefLab/VIVS
- **VIVS** (Python package (Jax backend) used to run the VI-VS conditional randomization test with K Monte Carlo replicates and fixed-posterior sampling) — https://github.com/YosefLab/VIVS
- **scanpy** (Loads the perturbational screen data as an AnnData object with raw counts and guide annotations)
- **scVI** (Deep generative model trained on expression counts to enable the fixed-posterior sampling scheme of the CRT)
- **Jax** (Parallel computing and just-in-time compilation backend speeding up VI-VS inference)
- **PyTorch** (Alternative backend for the VIVS implementation, noted as roughly half the speed of the Jax backend)

## Examples

```
model = VIVS(adata, feature_obsm_key='my_obsm_key', xy_linear=False, xy_model_kwargs={'n_hidden': 8}); model.train_all(); res = model.get_hier_importance(n_clusters_list=[100, 200])
```

## Evaluation signals

- Calibrated method's estimated FDR at or below the nominal target (alpha=0.1) after Benjamini-Hochberg correction
- Ground truth reproduced independently: two-sample t-tests of perturbed vs unperturbed cells for the ten target genes with the most guides yield the positive set
- TPR of VI-VS comparable to OLS's TPR despite a smaller pool of detected associations
- VI-VS FDR substantially lower than OLS (~70% vs ~95% in the published run) on the same response and discovery budget
- Output CSV (perturbational_screen_fdr_tpr.csv) contains one FDR and TPR row per method

## Limitations

- VI-VS discoveries have no guarantees to be functional in the general case; feedback loops, cell communication, or unobserved molecular species can lead to spurious discoveries
- Marginal association methods inflate correlations via technical factors (batch effects, sequencing depth) and nuisance biology (e.g. cell cycle), which distorts baseline comparisons if not accounted for
- VIVS may not scale to many thousands of genes; genes should be filtered first (e.g. select_genes with n_top_genes=2000)

## Evidence

- [results] Calibrated method beats OLS on precision without power loss: [evidence span withheld — anchor status: not_found]
- [methods] Benchmark target level and correction procedure: [evidence span withheld — anchor status: not_found]
- [methods] Ground truth definition via perturbation t-tests: [evidence span withheld — anchor status: not_found]
- [intro] Theoretical FDR calibration claim: "We demonstrate and theoretically prove that our procedure provides a calibrated estimation of the false discovery rate."
- [discussion] Spurious discovery risk in general case: "Feedback loops prevalent in molecular interactions, cell communication, or unobserved molecular species are just a few examples of phenomena that lead to spurious discoveries."
- [readme] Raw counts requirement in AnnData: "In particular, make sure than the anndata contains raw counts and not normalized data."
