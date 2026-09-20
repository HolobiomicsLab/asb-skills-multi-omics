---
name: metabolomics-dimensionality-reduction
description: Use when you have a metabolite-by-isolate abundance matrix (hundreds
  to thousands of features) and need to test whether samples cluster by a known categorical
  label, such as starter-culture taxonomy.
when_to_use_negative:
- The goal is association of the metabolomics traits with genes (use Scoary2 mGWAS
  instead), not unsupervised visualization of sample clustering.
- The input matrix has very few samples or features, where a 2D UMAP embedding of
  182 isolates x 1000+ metabolites is unnecessary or unstable.
- A supervised or classification model is required rather than exploratory 2D embedding
  of metabolome profiles.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3364
tools:
- name: umap-learn
  role: fit the 2-component UMAP embeddings of the LC-MS and GC-MS metabolite matrices
- name: pandas
  role: load the LC-MS and GC-MS abundance matrices and write the combined 2D coordinates
    to umap_projections.csv
- name: matplotlib
  role: render scatter plots of both projections colored by starter-culture taxonomy
- name: Scoary2
  role: the downstream mGWAS context whose Fig. 4 the UMAP cluster separation must
    reproduce
  repo: https://github.com/MrTomRod/scoary-2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:b8806663dcbab0f1eaaa36057029b405609390e6cacac6427eb5f0fcdd3eb716
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MrTomRod/scoary-2
  tools:
  - Scoary2
  - matplotlib
  - pandas
  - umap-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3364
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# metabolomics-dimensionality-reduction

## Summary

Project high-dimensional metabolomics matrices (e.g., LC-MS and GC-MS profiles of 182 yogurt isolates with 2348 and 1541 metabolites, respectively) into 2D UMAP embeddings to reveal taxonomy-driven clustering of strains.

## When to use

Use when you have a metabolite-by-isolate abundance matrix (hundreds to thousands of features) and need to test whether samples cluster by a known categorical label, such as starter-culture taxonomy.

## When NOT to use

- The goal is association of the metabolomics traits with genes (use Scoary2 mGWAS instead), not unsupervised visualization of sample clustering.
- The input matrix has very few samples or features, where a 2D UMAP embedding of 182 isolates x 1000+ metabolites is unnecessary or unstable.
- A supervised or classification model is required rather than exploratory 2D embedding of metabolome profiles.

## Inputs

- LC-MS metabolite abundance matrix (2348 metabolites x 182 isolates, CSV/TSV)
- GC-MS volatile matrix (1541 metabolites x 182 isolates, CSV/TSV)
- isolate-to-taxonomy annotation table (added starter-culture taxonomy per strain)

## Outputs

- 2D UMAP coordinates per isolate (e.g., umap_projections.csv with coordinates, dataset labels, and isolate annotations)
- matplotlib scatter plots of both projections colored by starter-culture taxonomy

## How to apply

Load each metabolomics matrix independently (LC-MS: 2348 metabolites x 182 isolates; GC-MS: 1541 metabolites x 182 isolates) with pandas. Preprocess each dataset separately: filter near-constant features and scale/normalize metabolite intensities so all traits contribute comparably before embedding. Fit a 2-component UMAP (umap-learn) independently to the LC-MS and GC-MS matrices to obtain 2D coordinates per isolate, then combine coordinates, dataset labels, and isolate annotations into a single CSV (e.g., umap_projections.csv). Render scatter plots with matplotlib, coloring each isolate by its added starter-culture taxonomy. Correctness is judged by whether each embedding separates the 182 isolates into two clusters dominated by Propionibacteriaceae versus Lactobacillales, consistent with Fig. 4 of the Scoary2 paper.

## Related tools

- **umap-learn** (fit the 2-component UMAP embeddings of the LC-MS and GC-MS metabolite matrices)
- **pandas** (load the LC-MS and GC-MS abundance matrices and write the combined 2D coordinates to umap_projections.csv)
- **matplotlib** (render scatter plots of both projections colored by starter-culture taxonomy)
- **Scoary2** (the downstream mGWAS context whose Fig. 4 the UMAP cluster separation must reproduce) — https://github.com/MrTomRod/scoary-2

## Examples

```
import umap; coords = umap.UMAP(n_components=2).fit_transform(scaled_lcms_matrix)
```

## Evaluation signals

- Each 2D embedding separates the 182 isolates into two clusters, one dominated by Propionibacteriaceae and one by Lactobacillales, matching Fig. 4 of the Scoary2 paper.
- The LC-MS and GC-MS matrices are preprocessed independently (near-constant features filtered, intensities scaled) before embedding.
- umap_projections.csv contains 2D coordinates, dataset labels, and isolate annotations for all 182 isolates from both datasets.
- Each isolate receives exactly one 2D coordinate pair per dataset (2 components per UMAP fit).

## Limitations

- Preprocessing is applied to each matrix independently (LC-MS vs GC-MS), so coordinates are not comparable across datasets within a single embedding.
- Cluster separation is qualitative (visual, taxonomy-dominated clusters), not a quantitative clustering metric; the discussion-section context of the paper was unavailable to extract further caveats.

## Evidence

- [other] workflow: [evidence span withheld — anchor status: not_found]
- [readme] readme: "Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder] or [Roary][roary] to traits."
