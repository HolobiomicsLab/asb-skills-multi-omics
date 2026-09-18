---
name: lc-ms-gc-ms-data-preprocessing
description: Use when raw LC-MS or GC-MS metabolite abundance matrices (metabolites
  x isolates, e.g. yogurt fermentations with 182 added starter strains) need normalization
  and feature filtering before dimensionality reduction or association testing.
when_to_use_negative:
- The data are pan-genome gene presence/absence tables or orthogroup outputs from
  OrthoFinder/Roary — these are Scoary2 association inputs, not metabolite matrices
  needing intensity normalization.
- The matrix is already feature-filtered and normalized to comparable scales — re-applying
  filtering/scaling could distort validated embeddings.
- The goal is gene-trait association reporting (p-value sorted gene lists), which
  is Scoary2's core function rather than this preprocessing step.
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: pandas
  role: Load the LC-MS and GC-MS abundance matrices as dataframes before filtering
    and scaling
- name: umap-learn
  role: Fit 2-component UMAP embeddings to each preprocessed matrix to yield 2D coordinates
    for the 182 isolates
- name: matplotlib
  role: Render scatter plots of both projections colored by added starter-culture
    taxonomy
- name: Scoary2
  role: Downstream mGWAS consumer of the preprocessed numeric trait matrices; its
    data exploration app reviews associations
  repo: github.com/MrTomRod/scoary-2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:f64b5bfddd4288247da21bca78c1a603edb1f69c6daec0330790bc4915354fe3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MrTomRod/scoary-2
  tools:
  - Scoary2
  - matplotlib
  - pandas
  - umap-learn
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# lc-ms-gc-ms-data-preprocessing

## Summary

Preprocess LC-MS metabolite abundance (2348 metabolites x 182 isolates) and GC-MS volatiles (1541 metabolites x 182 isolates) matrices from the yogurt metabolome study by filtering near-constant features and scaling/normalizing intensities so that downstream UMAP embedding and mGWAS analyses are comparable across traits.

## When to use

Use when raw LC-MS or GC-MS metabolite abundance matrices (metabolites x isolates, e.g. yogurt fermentations with 182 added starter strains) need normalization and feature filtering before dimensionality reduction or association testing.

## When NOT to use

- The data are pan-genome gene presence/absence tables or orthogroup outputs from OrthoFinder/Roary — these are Scoary2 association inputs, not metabolite matrices needing intensity normalization.
- The matrix is already feature-filtered and normalized to comparable scales — re-applying filtering/scaling could distort validated embeddings.
- The goal is gene-trait association reporting (p-value sorted gene lists), which is Scoary2's core function rather than this preprocessing step.

## Inputs

- LC-MS metabolite abundance matrix (2348 metabolites x 182 yogurt isolates)
- GC-MS volatiles matrix (1541 metabolites x 182 yogurt isolates)
- Isolate/starter-culture taxonomy annotations (e.g. Propionibacteriaceae vs Lactobacillales)

## Outputs

- Filtered and scaled/normalized LC-MS feature matrix
- Filtered and scaled/normalized GC-MS feature matrix
- umap_projections.csv with 2D UMAP coordinates, dataset labels (LC-MS / GC-MS), and isolate annotations
- Scatter plots (matplotlib) of both projections colored by starter-culture taxonomy

## How to apply

Load each matrix independently with pandas — the LC-MS metabolite abundance matrix (2348 metabolites x 182 isolates) and the GC-MS volatile matrix (1541 metabolites x 182 isolates). Preprocess each dataset separately: remove near-constant features whose variance carries no discriminative signal, then scale/normalize metabolite intensities so all traits contribute comparably to downstream analyses. The preprocessed matrices then feed 2-component UMAP embeddings (umap-learn) per dataset, producing 2D coordinates for each of the 182 isolates that are written to umap_projections.csv along with dataset labels and isolate annotations. The rationale, per the Scoary2 paper (Genome Biology, 10.1186/s13059-024-03233-7), is that existing tools could not pre-process numeric traits, so explicit filtering/scaling is required before taxonomy-driven structure (Propionibacteriaceae vs Lactobacillales clusters) can be recovered and before traits are associated to pan-genomes.

## Related tools

- **pandas** (Load the LC-MS and GC-MS abundance matrices as dataframes before filtering and scaling)
- **umap-learn** (Fit 2-component UMAP embeddings to each preprocessed matrix to yield 2D coordinates for the 182 isolates)
- **matplotlib** (Render scatter plots of both projections colored by added starter-culture taxonomy)
- **Scoary2** (Downstream mGWAS consumer of the preprocessed numeric trait matrices; its data exploration app reviews associations) — github.com/MrTomRod/scoary-2

## Examples

```
import pandas as pd, umap; X = pd.read_csv('lcms_matrix.csv', index_col=0); X = X.loc[X.var(axis=1) > 0]; emb = umap.UMAP(n_components=2).fit_transform(X.T)
```

## Evaluation signals

- Output umap_projections.csv contains 2D coordinates for all 182 isolates from both the LC-MS and GC-MS datasets, with dataset labels and isolate annotations present.
- Each 2D embedding separates isolates into two clusters dominated by Propionibacteriaceae vs Lactobacillales, consistent with the grouping reported in Fig. 4 of the Scoary2 paper.
- Near-constant features (no informative variance across the 182 isolates) have been removed from each matrix before embedding.
- Scaled/normalized intensities show all traits contributing comparably — no single metabolite dominates the UMAP geometry.
- The two datasets (2348 LC-MS and 1541 GC-MS features) were preprocessed independently, not pooled.

## Limitations

- UMAP embeddings are stochastic; cluster separation should be validated against the taxonomy-dominated grouping from Fig. 4 rather than assumed from raw coordinates.
- The source document lacks a discussion section, so no discussion-level limitations on preprocessing choices (e.g. normalization scheme details) are documented.
- This workflow addresses numeric trait preprocessing; it does not resolve the broader challenge that 40-60% of bacterial genes are undescribed, limiting interpretation of any associated genes.

## Evidence

- [other] Load the LC-MS metabolite abundance matrix (2348 metabolites x 182 isolates) and the GC-MS volatile matrix (1541 metabolites x 182 isolates) with pandas.: [evidence span withheld — anchor status: not_found]
- [other] 2D UMAP embeddings show taxonomy-based clustering: [evidence span withheld — anchor status: not_found]
- [abstract] Scoary2 is the first tool to enable the study of large phenotypic datasets using mGWAS: "With its data exploration app and improved performance, Scoary2 is the first tool to enable the study of large phenotypic datasets using mGWAS"
- [readme] Scoary2 associates orthogenes to traits: "Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder]
or [Roary][roary] to traits."
