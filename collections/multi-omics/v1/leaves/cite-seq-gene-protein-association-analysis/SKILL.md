---
name: cite-seq-gene-protein-association-analysis
description: Use when analyzing CITE-seq or other single-cell multiomic AnnData with
  raw gene counts in X and protein responses in obsm, to find conditionally dependent
  gene–protein pairs with calibrated FDR control rather than marginal correlations.
when_to_use_negative:
- The AnnData X contains normalized expression rather than raw counts — VIVS explicitly
  requires raw counts
- The feature matrix has many thousands of genes and cannot be filtered to ~2000 top
  genes, since VIVS may not scale to many thousands of genes
- The goal is marginal co-expression network inference without FDR calibration; VI-VS
  targets conditional dependencies and may miss associations OLS would flag
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: VIVS
  role: 'Core model: conditional randomization test with scVI generative model for
    gene counts, variable selection with FDR control'
  repo: github.com/YosefLab/VIVS
- name: scVI
  role: Generative model of normalized gene expression used to condition synthetic
    gene vectors on nuisance factors
  repo: github.com/YosefLab/VIVS
- name: Jax
  role: Parallel computing and just-in-time compilation backend for fast, scalable
    VI-VS inference
  repo: https://jax.readthedocs.io/en/latest/installation.html
- name: scanpy
  role: AnnData container format for importing raw counts and obsm protein responses
- name: Python
  role: Runtime environment (>= 3.9) for installing vivs via pip and running the pipeline
  repo: github.com/YosefLab/VIVS
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13059-024-03419-z
    title: 'VI-VS: calibrated identification of feature dependencies in single-cell
      multiomics'
  source_content_hash: sha256:9833685fb73c764d47a689ba0c3cc301fbaa841bd39b0327474e4ccfe1fbd257
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jax
  - Python
  - VI-VS
  - VIVS
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# cite-seq-gene-protein-association-analysis

## Summary

Reproduces the VI-VS conditional randomization test workflow for identifying gene–protein associations in a CITE-seq PBMC dataset (50,000 cells, 2000 genes, 224 surface proteins), comparing VI-VS against an OLS baseline at 10% target FDR with Benjamini-Hochberg correction and Rat-IgG isotype negative controls.

## When to use

Use when analyzing CITE-seq or other single-cell multiomic AnnData with raw gene counts in X and protein responses in obsm, to find conditionally dependent gene–protein pairs with calibrated FDR control rather than marginal correlations.

## When NOT to use

- The AnnData X contains normalized expression rather than raw counts — VIVS explicitly requires raw counts
- The feature matrix has many thousands of genes and cannot be filtered to ~2000 top genes, since VIVS may not scale to many thousands of genes
- The goal is marginal co-expression network inference without FDR calibration; VI-VS targets conditional dependencies and may miss associations OLS would flag

## Inputs

- scanpy AnnData object with raw CITE-seq gene counts in X (50,000 PBMCs, 2000 top genes after filtering)
- Protein responses (224 surface proteins, including 4 Rat-IgG isotype controls) stored in the AnnData obsm attribute as an array or dataframe
- 13 batch assignments used as nuisance factors for the scVI generative model

## Outputs

- cite_seq_pbmc_associations.csv: full table of predicted gene–protein associations with per-protein counts for VI-VS and OLS
- Benjamini-Hochberg-adjusted VI-VS p-values per gene–protein pair at 10% target FDR
- Per-method StringDB combined-score distribution figure for predicted gene–protein pairs

## How to apply

Load the CITE-seq PBMC AnnData with raw counts and the 224 protein responses in obsm, then filter to the top 2000 genes via select_genes(adata, n_top_genes=2000) since VIVS may not scale beyond a few thousand genes. Initialize VIVS with xy_linear=False and xy_model_kwargs={'n_hidden': 8} to capture nonlinear gene–protein interactions, and train with model.train_all(); the underlying scVI generative model conditions on the 13 batch assignments as nuisance factors and never exposes the protein response to the generative model. Run the conditional randomization test per protein using K Monte Carlo synthetic gene vectors, computing p-values as pg = (1 + sum of indicator terms)/(K+1). Apply Benjamini-Hochberg at 10% target FDR to VI-VS p-values and, in parallel, fit OLS with the same nuisance correction and threshold. Tabulate per-protein association counts for both methods (VI-VS found associations for only 51 proteins with ~52 interactions per protein vs. OLS returning associations for all 224 proteins with ~140 interactions), including detections on the four Rat-IgG isotype negative-control proteins, and map predicted pairs to StringDB combined scores to compare calibration.

## Related tools

- **VIVS** (Core model: conditional randomization test with scVI generative model for gene counts, variable selection with FDR control) — github.com/YosefLab/VIVS
- **scVI** (Generative model of normalized gene expression used to condition synthetic gene vectors on nuisance factors) — github.com/YosefLab/VIVS
- **Jax** (Parallel computing and just-in-time compilation backend for fast, scalable VI-VS inference) — https://jax.readthedocs.io/en/latest/installation.html
- **scanpy** (AnnData container format for importing raw counts and obsm protein responses)
- **Python** (Runtime environment (>= 3.9) for installing vivs via pip and running the pipeline) — github.com/YosefLab/VIVS

## Examples

```
from vivs import select_genes, VIVS; adata = select_genes(adata, n_top_genes=2000); model = VIVS(adata, feature_obsm_key='protein', xy_linear=False, xy_model_kwargs={'n_hidden': 8}); model.train_all()
```

## Evaluation signals

- VI-VS should flag associations for far fewer proteins than OLS at the same 10% target FDR (~51 of 224 proteins vs. all 224), with ~52 interactions per protein vs. ~140 for OLS
- The four Rat-IgG isotype negative-control proteins should show few or no VI-VS detections, serving as a false-positive calibration check
- p-values must follow the CRT formula pg = (1 + sum of indicator terms)/(K+1), guaranteeing valid super-uniformity under the null
- Gene filtering step must report exactly n_top_genes=2000 retained features before model training
- StringDB combined-score distribution for VI-VS calls should be shifted toward higher scores relative to OLS, reflecting fewer but more specific associations

## Limitations

- VI-VS discoveries have no guarantees to be functional in the general case; feedback loops, cell communication, or unobserved molecular species can lead to spurious discoveries
- VIVS may not scale to many thousands of genes, so gene filtering is required before running
- No software version information is provided for the generative model or VI-VS implementation, and applicability to chromatin accessibility data is not yet demonstrated

## Evidence

- [readme] README requires raw counts and obsm responses: "In particular, make sure than the anndata contains raw counts and not normalized data. The response(s) $Y$ of interest are expected to be stored in the `obsm` attribute of the anndata object"
- [readme] README gene filtering: "VIVS may not scale to many thousands of genes. In such a case, it is recommended to filter the genes before running VIVS"
- [methods] Model initialization parameters: "model = VIVS(
    adata,
    feature_obsm_key="my_obsm_key",
    xy_linear=False,
    xy_model_kwargs={"n_hidden": 8}
)
model.train_all()"
- [intro] FDR calibration claim: "We demonstrate and theoretically prove that our procedure provides a calibrated estimation of the false discovery rate."
- [discussion] Batch nuisance conditioning: "VI-VS effectively addresses this issue by conditioning on these nuisance factors."
- [methods] Jax backend: "Our implementation first relies on parallel computing and just-in-time compilation components of Jax to speed up the inference"
