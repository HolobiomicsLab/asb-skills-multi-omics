---
name: multi-omics-scatter-visualization
description: Use when you have high-dimensional abundance matrices (e.g., 2348 LC-MS
  metabolites x 182 isolates, 1541 GC-MS volatiles x 182 isolates) and need to visually
  assess whether samples cluster by a categorical annotation such as bacterial taxonomy.
when_to_use_negative:
- The goal is statistical association of traits with genes (mGWAS) rather than exploratory
  visualization — use Scoary2's association workflow instead.
- The data are already 2D or low-dimensional embeddings, so dimensionality reduction
  via UMAP adds no information.
- Only a single omics modality with pre-computed sample coordinates is available and
  only plotting (not embedding) is required.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3520
tools:
- name: umap-learn
  role: Fits the 2-component UMAP embedding of each preprocessed LC-MS/GC-MS abundance
    matrix to yield 2D coordinates per isolate.
- name: pandas
  role: Loads the metabolite abundance matrices and writes the combined 2D coordinates,
    dataset labels, and isolate annotations to umap_projections.csv.
- name: matplotlib
  role: Renders the scatter plots of both UMAP projections, coloring each isolate
    by its added starter-culture taxonomy.
- name: Scoary2
  role: Source study tool whose yogurt metabolome dataset (182 strains) and Fig. 4
    taxonomy-driven clustering provide the reference result this visualization reproduces.
  repo: github.com/MrTomRod/scoary-2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:d0a54e15b389f7612edf263e5106480006e5f94e8d5459d2e084bc923d6a0844
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Scoary2
  - matplotlib
  - pandas
  - umap-learn
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# multi-omics-scatter-visualization

## Summary

Embed high-dimensional multi-omics feature matrices (here LC-MS metabolite abundances and GC-MS volatiles) into 2D scatter plots with UMAP, coloring points by metadata (e.g., starter-culture taxonomy) to visually test whether profiles cluster by biological grouping, as in Fig. 4 of the Scoary2 paper.

## When to use

Use when you have high-dimensional abundance matrices (e.g., 2348 LC-MS metabolites x 182 isolates, 1541 GC-MS volatiles x 182 isolates) and need to visually assess whether samples cluster by a categorical annotation such as bacterial taxonomy.

## When NOT to use

- The goal is statistical association of traits with genes (mGWAS) rather than exploratory visualization — use Scoary2's association workflow instead.
- The data are already 2D or low-dimensional embeddings, so dimensionality reduction via UMAP adds no information.
- Only a single omics modality with pre-computed sample coordinates is available and only plotting (not embedding) is required.

## Inputs

- LC-MS metabolite abundance matrix (2348 metabolites x 182 isolates)
- GC-MS volatile abundance matrix (1541 metabolites x 182 isolates)
- Isolate annotation table mapping each yogurt isolate to its added starter-culture taxonomy (e.g., Propionibacteriaceae vs Lactobacillales)

## Outputs

- umap_projections.csv containing 2D UMAP coordinates, dataset labels (LC-MS/GC-MS), and isolate annotations for all 182 isolates
- 2D scatter plots (matplotlib) of the LC-MS and GC-MS embeddings colored by starter-culture taxonomy

## How to apply

Load each omics matrix independently with pandas (LC-MS: 2348 metabolites x 182 isolates; GC-MS: 1541 metabolites x 182 isolates from the Agroscope yogurt strain collection). Preprocess each matrix separately — filter near-constant features and scale/normalize metabolite intensities so all traits contribute comparably to the embedding. Fit a 2-component UMAP (umap-learn) to each preprocessed matrix to obtain 2D coordinates for each of the 182 isolates, then write the combined coordinates, dataset labels (LC-MS vs GC-MS), and isolate annotations to umap_projections.csv. Render matplotlib scatter plots of both projections, coloring each isolate by its added starter-culture taxonomy, and validate that each embedding separates isolates into two clusters dominated by Propionibacteriaceae vs Lactobacillales, consistent with Fig. 4 of the Scoary2 paper.

## Related tools

- **umap-learn** (Fits the 2-component UMAP embedding of each preprocessed LC-MS/GC-MS abundance matrix to yield 2D coordinates per isolate.)
- **pandas** (Loads the metabolite abundance matrices and writes the combined 2D coordinates, dataset labels, and isolate annotations to umap_projections.csv.)
- **matplotlib** (Renders the scatter plots of both UMAP projections, coloring each isolate by its added starter-culture taxonomy.)
- **Scoary2** (Source study tool whose yogurt metabolome dataset (182 strains) and Fig. 4 taxonomy-driven clustering provide the reference result this visualization reproduces.) — github.com/MrTomRod/scoary-2

## Examples

```
import umap; emb = umap.UMAP(n_components=2).fit_transform(X_scaled)
```

## Evaluation signals

- umap_projections.csv contains 2D coordinates for exactly 182 isolates for each dataset (LC-MS and GC-MS), with dataset labels and taxonomy annotations present.
- Each embedding separates the isolates into two clusters, one dominated by Propionibacteriaceae and one by Lactobacillales.
- The clustering pattern is consistent with the grouping reported in Fig. 4 of the Scoary2 paper.
- Preprocessing was applied independently per matrix (near-constant features filtered, intensities scaled/normalized) so no matrix dominates the embedding.
- Scatter plots render for both projections with points colored by added starter-culture taxonomy.

## Limitations

- UMAP embeddings are stochastic; exact cluster positions vary between runs unless random_state is fixed, so validation should rely on taxonomy-driven cluster separation rather than point-level identity.
- Taxonomy-based clustering is a qualitative visual finding, not a quantitative association test — it does not by itself identify genes affecting metabolites.
- The workflow's validation signal (two taxonomy-dominated clusters) is specific to the 182-strain yogurt dataset; other datasets may not show this separation.

## Evidence

- [other] 2D UMAP embeddings of the full LC-MS and GC-MS volatiles datasets (182 strains) show taxonomy-based clustering: [evidence span withheld — anchor status: not_found]
- [abstract] carnitine metabolism in yogurt metabolomes: "we explore the metabolome of yogurts, each produced with a different Propionibacterium reichii strain and discover two genes affecting carnitine metabolism"
- [readme] Scoary2 interactive exploration app: "The results can be explored interactively with a simple, static HTML/JS app."
