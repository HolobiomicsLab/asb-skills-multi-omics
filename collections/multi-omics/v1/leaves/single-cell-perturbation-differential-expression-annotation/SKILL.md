---
name: single-cell-perturbation-differential-expression-annotation
description: Use when a Perturb-Seq AnnData with raw counts and per-cell guide annotations
  must be screened for causal gene-gene interactions with calibrated FDR control,
  and VI-VS must be benchmarked against OLS and marginal tests.
when_to_use_negative:
- The AnnData X matrix contains normalized rather than raw counts, since VIVS only
  supports assays where X corresponds to gene expression counts
- The goal is a marginal co-expression network rather than conditionally dependent
  (causal) features; VI-VS is designed for conditional associations
- The feature set has many thousands of genes without pre-filtering, as VIVS may not
  scale to many thousands of genes
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: VI-VS
  role: Conditional randomization test framework with calibrated FDR control used
    to identify candidate gene-gene interactions and generate p-values
  repo: github.com/YosefLab/VIVS
- name: VIVS
  role: Open-source Python implementation of VI-VS, fit with xy_linear=False and n_hidden=8
    on unperturbed cells
  repo: github.com/YosefLab/VIVS
- name: scVI
  role: Deep generative model trained on expression counts whose decoder supplies
    the fixed-posterior null samples p_theta(x_g|z,s)
- name: scanpy
  role: 'Data container: the perturbational screen is imported as a scanpy AnnData
    object with raw counts'
- name: Jax
  role: Backend providing parallel computing and just-in-time compilation to speed
    up VI-VS inference
- name: PyTorch
  role: Alternative backend mentioned as slower than the default implementation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:3871024013fe917ae1522f9fb8d8aa1ebe9404db2128b70146f0de0b82834352
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# single-cell-perturbation-differential-expression-annotation

## Summary

Benchmark VI-VS conditional randomization testing against OLS and marginal independence tests for identifying causal gene-gene interactions in a Perturb-Seq screen, using two-sample t-tests of perturbed vs unperturbed cells as ground truth and evaluating estimated FDR (Benjamini-Hochberg at alpha=0.1) and true positive rate.

## When to use

Use when a Perturb-Seq AnnData with raw counts and per-cell guide annotations must be screened for causal gene-gene interactions with calibrated FDR control, and VI-VS must be benchmarked against OLS and marginal tests.

## When NOT to use

- The AnnData X matrix contains normalized rather than raw counts, since VIVS only supports assays where X corresponds to gene expression counts
- The goal is a marginal co-expression network rather than conditionally dependent (causal) features; VI-VS is designed for conditional associations
- The feature set has many thousands of genes without pre-filtering, as VIVS may not scale to many thousands of genes

## Inputs

- Perturb-Seq dataset as a scanpy AnnData object with raw gene expression counts (not normalized data)
- Per-cell guide/perturbation annotations
- Target gene list restricted to the ten target genes with the most guides

## Outputs

- Per-method (VI-VS, OLS, marginal test) table of estimated FDR and TPR written to perturbational_screen_fdr_tpr.csv
- Benjamini-Hochberg-adjusted VI-VS p-values at alpha=0.1

## How to apply

Load the perturbational screen as a scanpy AnnData object with raw gene expression counts and per-cell guide/perturbation annotations, then define ground-truth positives by two-sample t-tests comparing perturbed versus unperturbed cells for the ten target genes with the largest number of guides. Train a scVI generative model and fit the VI-VS importance model (nonlinear MLP with xy_linear=False, n_hidden=8) on unperturbed cells only, with the target gene expression as response. Run the conditional randomization test per candidate feature with K Monte Carlo replicates using fixed-posterior sampling: draw one latent z per cell from the encoder q(z|x), then sample synthetic counts iid from the decoder p_theta(x_g|z,s) independent of y. Apply Benjamini-Hochberg at target FDR alpha=0.1 to VI-VS p-values, run OLS and the marginal test on the same response, and compute estimated FDR and TPR per method against the t-test positives, writing perturbational_screen_fdr_tpr.csv.

## Related tools

- **VI-VS** (Conditional randomization test framework with calibrated FDR control used to identify candidate gene-gene interactions and generate p-values) — github.com/YosefLab/VIVS
- **VIVS** (Open-source Python implementation of VI-VS, fit with xy_linear=False and n_hidden=8 on unperturbed cells) — github.com/YosefLab/VIVS
- **scVI** (Deep generative model trained on expression counts whose decoder supplies the fixed-posterior null samples p_theta(x_g|z,s))
- **scanpy** (Data container: the perturbational screen is imported as a scanpy AnnData object with raw counts)
- **Jax** (Backend providing parallel computing and just-in-time compilation to speed up VI-VS inference)
- **PyTorch** (Alternative backend mentioned as slower than the default implementation)

## Examples

```
from vivs import VIVS; model = VIVS(adata, feature_obsm_key="my_obsm_key", xy_linear=False, xy_model_kwargs={"n_hidden": 8}); model.train_all()
```

## Evaluation signals

- Calibrated VI-VS procedure achieves an estimated FDR at or below the nominal alpha=0.1 threshold after Benjamini-Hochberg
- VI-VS reaches a lower estimated FDR than OLS and the marginal test (paper reports ~70% for VI-VS vs ~95% for OLS)
- VI-VS identifies a similar number of true positives as OLS despite a smaller pool of detected associations, i.e., improved precision without reduced power
- Ground-truth positives match two-sample t-test results comparing perturbed vs unperturbed cells for the ten target genes with the most guides
- Per-method FDR/TPR table is written and inspectable in perturbational_screen_fdr_tpr.csv

## Limitations

- VI-VS discoveries have no guarantees to be functional in the general case; feedback loops, cell communication, or unobserved molecular species can lead to spurious discoveries
- VIVS may not scale to many thousands of genes, so genes should be filtered beforehand (e.g., select_genes with n_top_genes=2000)
- No software version information is provided for the generative model or VI-VS implementation, and applicability to chromatin accessibility data is not yet demonstrated

## Evidence

- [abstract] It uses deep generative models to identify conditionally dependent features, with false discovery rate control.: "It uses deep generative models to identify conditionally dependent features, with false discovery rate control."
- [methods] We implemented VI-VS in a fast and scalable way that is available as an open-source Python package.: "We implemented VI-VS in a fast and scalable way that is available as an open-source Python package."
- [readme] In particular, make sure than the anndata contains raw counts and not normalized data.: "In particular, make sure than the anndata contains raw counts and not normalized data."
- [readme] In such a case, it is recommended to filter the genes before running VIVS: "VIVS may not scale to many thousands of genes.
In such a case, it is recommended to filter the genes before running VIVS"
- [discussion] In the general case, VI-VS discoveries have no guarantees to be functional.: "In the general case, VI-VS discoveries have no guarantees to be functional."
