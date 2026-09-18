---
name: umap-embedding-of-omics-matrices
description: Use when a high-dimensional omics abundance matrix (thousands of metabolite
  features x tens-to-hundreds of isolates) needs a low-dimensional embedding to test
  whether samples cluster by a known grouping such as bacterial taxonomy.
when_to_use_negative:
- The goal is gene-trait association (mGWAS) rather than exploratory dimensionality
  reduction — use Scoary2 itself, which associates orthogenes to traits
- The matrix is small or low-dimensional, where filtering near-constant features and
  nonlinear embedding adds noise rather than insight
- A supervised model predicting taxonomy labels is required instead of an unsupervised
  embedding
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3172
- http://edamontology.org/topic_2269
tools:
- name: umap-learn
  role: Fits the 2-component UMAP embeddings for each omics matrix
- name: pandas
  role: Loads and preprocesses the LC-MS and GC-MS abundance matrices and writes umap_projections.csv
- name: matplotlib
  role: Renders scatter plots of both projections colored by starter-culture taxonomy
- name: Scoary2
  role: Source study and paper whose yogurt metabolome data and Fig. 4 taxonomy clustering
    this embedding reproduces; also the mGWAS tool for downstream gene-trait association
  repo: https://github.com/MrTomRod/scoary-2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:8524e0dd20bdc2c873458cfebbd7faa5f8da1176af2701c3693867b0037cfeaf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MrTomRod/scoary-2
  tools:
  - Scoary2
  - matplotlib
  - pandas
  - umap-learn
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3172
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# umap-embedding-of-omics-matrices

## Summary

Projects large metabolomics abundance matrices (e.g. the Scoary2 yogurt study's LC-MS 2348 metabolites x 182 isolates and GC-MS 1541 volatiles x 182 isolates) into 2D UMAP space to reveal taxonomy-driven clustering of starter-culture strains.

## When to use

Use when a high-dimensional omics abundance matrix (thousands of metabolite features x tens-to-hundreds of isolates) needs a low-dimensional embedding to test whether samples cluster by a known grouping such as bacterial taxonomy.

## When NOT to use

- The goal is gene-trait association (mGWAS) rather than exploratory dimensionality reduction — use Scoary2 itself, which associates orthogenes to traits
- The matrix is small or low-dimensional, where filtering near-constant features and nonlinear embedding adds noise rather than insight
- A supervised model predicting taxonomy labels is required instead of an unsupervised embedding

## Inputs

- LC-MS metabolite abundance matrix (2348 metabolites x 182 isolates)
- GC-MS volatile abundance matrix (1541 metabolites x 182 isolates)
- Isolate annotations mapping each of the 182 yogurt isolates to its added starter-culture taxonomy

## Outputs

- umap_projections.csv with combined 2D UMAP coordinates, dataset labels (LC-MS/GC-MS), and isolate annotations
- Matplotlib scatter plots of both 2D embeddings colored by starter-culture taxonomy

## How to apply

Load each omics matrix independently with pandas (LC-MS: 2348 metabolites x 182 isolates; GC-MS: 1541 volatiles x 182 isolates). Preprocess each matrix separately by filtering near-constant features and scaling/normalizing metabolite intensities so all traits contribute comparably to the embedding. Fit a 2-component UMAP embedding (umap-learn) per matrix, yielding 2D coordinates for each of the 182 isolates, then write the combined coordinates, dataset labels, and isolate annotations to umap_projections.csv. Render scatter plots with matplotlib, coloring each isolate by its added starter-culture taxonomy, and validate that the embeddings separate isolates into two clusters dominated by Propionibacteriaceae vs Lactobacillales, matching Fig. 4 of the Scoary2 paper.

## Related tools

- **umap-learn** (Fits the 2-component UMAP embeddings for each omics matrix)
- **pandas** (Loads and preprocesses the LC-MS and GC-MS abundance matrices and writes umap_projections.csv)
- **matplotlib** (Renders scatter plots of both projections colored by starter-culture taxonomy)
- **Scoary2** (Source study and paper whose yogurt metabolome data and Fig. 4 taxonomy clustering this embedding reproduces; also the mGWAS tool for downstream gene-trait association) — https://github.com/MrTomRod/scoary-2

## Examples

```
import pandas as pd, umap; X = pd.read_csv('lcms_matrix.csv', index_col=0).values; emb = umap.UMAP(n_components=2).fit_transform(X)
```

## Evaluation signals

- umap_projections.csv contains 2D coordinates for all 182 isolates in both the LC-MS and GC-MS datasets, with dataset labels and isolate annotations attached
- Each embedding separates the isolates into two clusters dominated by Propionibacteriaceae vs Lactobacillales
- Near-constant features were filtered and intensities scaled/normalized independently per matrix before fitting UMAP
- Scatter plots color each isolate by its added starter-culture taxonomy and are consistent with the grouping reported in Fig. 4 of the Scoary2 paper

## Limitations

- UMAP is stochastic and hyperparameter-sensitive, so the exact embedding geometry may vary between runs even when the taxonomy-driven cluster structure is preserved
- The two matrices must be preprocessed independently (separate filtering and scaling) because LC-MS and GC-MS feature intensities are not directly comparable
- Clustering by added starter-culture taxonomy reflects the yogurt isolate set (strains from the Agroscope strain collection) and may not generalize to other omics matrices or strain panels

## Evidence

- [other] 2D UMAP embeddings of the full LC-MS and GC-MS volatiles datasets (182 strains) show taxonomy-based clustering: [evidence span withheld — anchor status: not_found]
- [other] Fit a 2-component UMAP embedding (umap-learn) to the LC-MS matrix and a second 2-component UMAP embedding to the GC-MS matrix, yielding 2D coordinates for each of the 182 isolates: [evidence span withheld — anchor status: not_found]
- [readme] Scoary2 associates orthogenes (e.g. generated using OrthoFinder or Roary) to traits, providing the mGWAS context in which these yogurt metabolome embeddings were produced: "Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder]
or [Roary][roary] to traits."
