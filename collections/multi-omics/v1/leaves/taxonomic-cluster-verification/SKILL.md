---
name: taxonomic-cluster-verification
description: Use when 2D UMAP (or similar) embeddings of metabolite abundance matrices
  from strains carrying different added bacterial taxa must be checked for taxonomy-driven
  clustering before downstream mGWAS association.
when_to_use_negative:
- The goal is gene-trait association itself (linking pan-genome orthogenes to metabolite
  traits) rather than validating taxonomy-driven clustering — use Scoary2's mGWAS
  workflow instead
- The input is already a low-dimensional embedding or clustering result; re-embedding
  would discard the existing structure rather than verify it
- Strains lack added starter-culture taxonomy labels, since the cluster-dominance
  check (Propionibacteriaceae vs Lactobacillales) cannot be evaluated
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: umap-learn
  role: Fits the 2-component UMAP embeddings of the preprocessed LC-MS and GC-MS metabolite
    matrices
- name: pandas
  role: Loads the metabolite abundance matrices and writes the combined umap_projections.csv
- name: matplotlib
  role: Renders the scatter plots of both projections colored by starter-culture taxonomy
- name: Scoary2
  role: Downstream mGWAS tool whose Fig. 4 taxonomy grouping of yogurt metabolomes
    this verification reproduces
  repo: https://github.com/MrTomRod/scoary-2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:f070620f3dbc62ee8d7c7dbd4c6fcfaf4fb3204043f04d61d3d92519eba5d5ff
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MrTomRod/scoary-2
  tools:
  - Scoary2
  - matplotlib
  - pandas
  - umap-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# Taxonomic-cluster verification of yogurt metabolome embeddings

## Summary

Validates that dimensionality-reduced yogurt metabolome profiles (LC-MS and GC-MS volatiles, 182 isolates) separate into clusters dominated by the added starter-culture taxonomy — one Propionibacteriaceae-dominated and one Lactobacillales-dominated — reproducing the grouping reported in Fig. 4 of the Scoary2 paper.

## When to use

Use when 2D UMAP (or similar) embeddings of metabolite abundance matrices from strains carrying different added bacterial taxa must be checked for taxonomy-driven clustering before downstream mGWAS association.

## When NOT to use

- The goal is gene-trait association itself (linking pan-genome orthogenes to metabolite traits) rather than validating taxonomy-driven clustering — use Scoary2's mGWAS workflow instead
- The input is already a low-dimensional embedding or clustering result; re-embedding would discard the existing structure rather than verify it
- Strains lack added starter-culture taxonomy labels, since the cluster-dominance check (Propionibacteriaceae vs Lactobacillales) cannot be evaluated

## Inputs

- LC-MS metabolite abundance matrix (2348 metabolites x 182 yogurt isolates)
- GC-MS volatile abundance matrix (1541 metabolites x 182 yogurt isolates)
- Per-isolate annotation of the added starter-culture taxonomy (e.g., Propionibacteriaceae vs Lactobacillales)

## Outputs

- umap_projections.csv with 2D coordinates, dataset labels (LC-MS/GC-MS), and isolate annotations for 182 isolates
- Scatter plots of the LC-MS and GC-MS UMAP embeddings colored by starter-culture taxonomy

## How to apply

Load the LC-MS abundance matrix (2348 metabolites x 182 isolates) and the GC-MS volatile matrix (1541 metabolites x 182 isolates) with pandas, and preprocess each independently — filter near-constant features and scale/normalize intensities so all metabolites contribute comparably. Fit a separate 2-component UMAP (umap-learn) to each matrix, yielding 2D coordinates for each of the 182 isolates, and write the combined coordinates, dataset labels (LC-MS vs GC-MS), and isolate annotations to umap_projections.csv. Render scatter plots with matplotlib, coloring each isolate by its added starter-culture taxonomy. Consider the skill correctly applied only if each embedding resolves into two clusters dominated by Propionibacteriaceae versus Lactobacillales, consistent with the taxonomy grouping reported in Fig. 4 of the Scoary2 paper.

## Related tools

- **umap-learn** (Fits the 2-component UMAP embeddings of the preprocessed LC-MS and GC-MS metabolite matrices)
- **pandas** (Loads the metabolite abundance matrices and writes the combined umap_projections.csv)
- **matplotlib** (Renders the scatter plots of both projections colored by starter-culture taxonomy)
- **Scoary2** (Downstream mGWAS tool whose Fig. 4 taxonomy grouping of yogurt metabolomes this verification reproduces) — https://github.com/MrTomRod/scoary-2

## Examples

```
import pandas as pd, umap; X = pd.read_csv('lcms_matrix.csv'); emb = umap.UMAP(n_components=2).fit_transform(X)
```

## Evaluation signals

- Each 2D embedding separates the 182 isolates into two clusters, one dominated by Propionibacteriaceae and one by Lactobacillales
- Both the LC-MS (2348 x 182) and GC-MS (1541 x 182) embeddings show the same taxonomy-driven grouping, consistent with Fig. 4 of the Scoary2 paper
- umap_projections.csv contains 2D coordinates for all 182 isolates in both datasets (LC-MS and GC-MS) with dataset labels and isolate annotations
- Near-constant features were removed and intensities scaled/normalized independently for each matrix before embedding, so no single metabolite dominates the coordinates

## Limitations

- Scoary2's performance focus is on tens of thousands of traits in mGWAS; the validation here is a 2D qualitative cluster check and is sensitive to UMAP stochasticity and preprocessing choices
- The extracted article text lacks a discussion section, so no discussion-level caveats or failure modes for this clustering step could be traced

## Evidence

- [abstract] Scoary2 enables exploration of the yogurt metabolome via taxonomy-structured omics data: "we explore the metabolome of yogurts, each produced with a different Propionibacterium reichii strain and discover two genes affecting carnitine metabolism"
- [abstract] Scoary2 supports large phenotypic datasets, of which the 182-isolate metabolome matrices are an instance: "With its data exploration app and improved performance, Scoary2 is the first tool to enable the study of large phenotypic datasets using mGWAS"
- [readme] Scoary2's purpose is associating pan-genome orthogenes with traits such as metabolome measurements: "Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder]
or [Roary][roary] to traits. It reports a list of genes sorted by strength of"
