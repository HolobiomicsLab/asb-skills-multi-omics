---
name: negative-control-protein-calibration-assessment
description: Use when testing a feature-selection or association method (e.g., VI-VS
  vs OLS) on CITE-seq gene-protein data and you need a negative-control (isotype)
  benchmark plus per-protein association counts to judge calibration at a fixed target
  FDR.
when_to_use_negative:
- The data are normalized expression rather than raw counts — VIVS requires raw counts
  in the AnnData X matrix
- The response variables are chromatin accessibility or other features without a validated
  generative model — the paper notes ATAC-seq sparsity may require a specialized generative
  model to keep p-values valid
- There are no negative-control features or nuisance annotations (e.g., batch) available,
  so the calibration comparison cannot be grounded
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: VI-VS / VIVS
  role: Conditional randomization test framework whose calibrated FDR is benchmarked
    against OLS using isotype negative-control proteins
  repo: https://github.com/YosefLab/VIVS
- name: scVI
  role: Latent-variable generative model of gene counts used for the conditional distribution,
    conditioned on the 13 batch nuisance factors
  repo: https://github.com/YosefLab/VIVS
- name: scanpy
  role: Provides the AnnData container holding raw gene counts and the protein responses
    in obsm
- name: Jax
  role: Backend providing parallel computing and JIT compilation to speed up VI-VS
    inference
- name: Python
  role: Implementation language; VI-VS is an open-source Python package installed
    via pip
  repo: https://github.com/YosefLab/VIVS
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:f9e5fd97cff80922f343898ffd9e458e1cd374a3c41a36f043a3e2ee6600a6b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VIVS
  tools:
  - Jax
  - Python
  - VI-VS
  - VI-VS / VIVS
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# negative-control-protein-calibration-assessment

## Summary

A calibration-assessment practice from the VI-VS CITE-seq PBMC analysis in which isotype negative-control proteins (Rat-IgG) are used alongside real surface proteins to compare how VI-VS conditional randomization testing versus an OLS baseline controls the number of gene-protein associations reported at a 10% target FDR.

## When to use

Use when testing a feature-selection or association method (e.g., VI-VS vs OLS) on CITE-seq gene-protein data and you need a negative-control (isotype) benchmark plus per-protein association counts to judge calibration at a fixed target FDR.

## When NOT to use

- The data are normalized expression rather than raw counts — VIVS requires raw counts in the AnnData X matrix
- The response variables are chromatin accessibility or other features without a validated generative model — the paper notes ATAC-seq sparsity may require a specialized generative model to keep p-values valid
- There are no negative-control features or nuisance annotations (e.g., batch) available, so the calibration comparison cannot be grounded

## Inputs

- CITE-seq PBMC AnnData object (scanpy) with raw gene counts (~50,000 cells, ~2000 top genes) and 224 surface protein responses stored in obsm
- Batch assignment annotations (13 batches) used as nuisance factors
- Identification of the four Rat-IgG isotype negative-control proteins among the 224 protein responses

## Outputs

- Per-protein association-count table comparing VI-VS and OLS at 10% target FDR (cite_seq_pbmc_associations.csv)
- Negative-control detection counts on the four Rat-IgG isotype proteins per method
- StringDB combined-score distribution figure per method for predicted gene-protein pairs

## How to apply

Load the CITE-seq PBMC AnnData object (50,000 cells, raw gene counts, 224 surface protein responses in obsm), filter with select_genes(adata, n_top_genes=2000), and train the VIVS model (xy_linear=False, xy_model_kwargs={'n_hidden': 8}, model.train_all()) using a scVI generative model conditioned on the 13 batch assignments as nuisance factors, never exposing protein responses to the generative model. Run the conditional randomization test per protein with K Monte Carlo synthetic gene vectors, computing p-values as pg = (1 + sum of indicator terms)/(K+1). Apply Benjamini-Hochberg at 10% target FDR to VI-VS p-values and, in parallel, fit an OLS baseline with the same nuisance correction and threshold. Tabulate per-protein association counts for both methods, explicitly including detections on the four Rat-IgG isotype negative-control proteins: a calibrated method should make few or no calls on isotype controls, whereas in the reference analysis OLS returned a very large number of associations for all 224 proteins while VI-VS called associations for only 51 proteins (52 interactions per protein on average, vs 140 for OLS). Optionally map predicted gene-protein pairs to StringDB combined scores and plot the per-method score distribution as a further sanity check.

## Related tools

- **VI-VS / VIVS** (Conditional randomization test framework whose calibrated FDR is benchmarked against OLS using isotype negative-control proteins) — https://github.com/YosefLab/VIVS
- **scVI** (Latent-variable generative model of gene counts used for the conditional distribution, conditioned on the 13 batch nuisance factors) — https://github.com/YosefLab/VIVS
- **scanpy** (Provides the AnnData container holding raw gene counts and the protein responses in obsm)
- **Jax** (Backend providing parallel computing and JIT compilation to speed up VI-VS inference)
- **Python** (Implementation language; VI-VS is an open-source Python package installed via pip) — https://github.com/YosefLab/VIVS

## Examples

```
from vivs import VIVS, select_genes; adata = select_genes(adata, n_top_genes=2000); model = VIVS(adata, feature_obsm_key='protein', xy_linear=False, xy_model_kwargs={'n_hidden': 8}); model.train_all()
```

## Evaluation signals

- At the same 10% target FDR, a calibrated method (VI-VS) makes few or no calls on the four Rat-IgG isotype negative-control proteins
- VI-VS reports associations for a subset of proteins (51 of 224 in the reference run) rather than for all 224 proteins as the uncalibrated OLS baseline does
- Mean interactions per protein is smaller for VI-VS (~52) than for OLS (~140), consistent with stringency rather than uniform inflation
- Benjamini-Hochberg thresholding is applied at exactly the 10% target FDR to both methods identically, with the same nuisance (13-batch) correction
- Predicted gene-protein pairs, when mapped to StringDB, show a combined-score distribution per method that can be plotted and compared

## Limitations

- VI-VS discoveries have no guarantees to be functional in the general case; feedback loops, cell communication, or unobserved molecular species can lead to spurious discoveries
- No software version information is provided for the generative model or the VI-VS implementation
- VIVS may not scale to many thousands of genes, so gene filtering (e.g., n_top_genes=2000) is required before running
- Cross-environment reproducibility of discoveries is only a prospective mitigation, not yet validated

## Evidence

- [abstract] VI-VS p-values remain calibrated and OLS returns associations for all 224 proteins: "It uses deep generative models to identify conditionally dependent features, with false discovery rate control."
- [readme] Methods: model initialization and training parameters: "model = VIVS(
    adata,
    feature_obsm_key="my_obsm_key",
    xy_linear=False,
    xy_model_kwargs={"n_hidden": 8}
)
model.train_all()"
- [readme] Gene filtering before running VIVS: "adata = select_genes(adata, n_top_genes=2000)"
- [readme] Data format requirements: raw counts and responses in obsm: "In particular, make sure than the anndata contains raw counts and not normalized data.

The response(s) $Y$ of interest are expected to be stored in the `obsm` attribute of the anndata object"
- [intro] Calibration guarantee of the FDR estimate: "We demonstrate and theoretically prove that our procedure provides a calibrated estimation of the false discovery rate."
- [discussion] Nuisance-factor conditioning as a rationale for the comparison: "VI-VS effectively addresses this issue by conditioning on these nuisance factors."
