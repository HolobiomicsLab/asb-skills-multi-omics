---
name: crispr-screen-gene-prioritization
description: Use when multiple published CRISPR/Cas9 screening cohorts with immune-competent
  vs immune-deficient conditions are available and the goal is a consensus gene ranking
  tied to iMLGAM-score prognostic markers.
when_to_use_negative:
- The input is the iMLGAM model-building RNA-seq data (trainset_7/trainset_3/testset
  .qs files) — use the ABESS/basic_learner/GA_train modeling workflow instead
- Only a single CRISPR cohort is available, so no cross-cohort z-score/mean-rank consensus
  can be computed
- No survival or clinical annotations exist for the meta cohort, so univariate Cox
  regression of iMLGAM-score-associated genes cannot be run
edam_operation: http://edamontology.org/operation_2495
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: iMLGAM
  role: R package environment used to load the CRISPR cohorts and compute the iMLGAM
    scores that genes are prioritized against
  repo: https://github.com/Yelab1994/iMLGAM
- name: R
  role: Implementation language for the cohort loading, sgRNA logFC prioritization,
    z-score aggregation, and univariate Cox regression
- name: qs
  role: Library for reading the .qs serialized cohort/example files used in the iMLGAM
    environment (qread)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:94a8913bf81a47c6d6cb2e83989aadcef27479b5ef24b5acf4d9627f3e55ac3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - iMLGAM
  - qs
  edam_operation: http://edamontology.org/operation_2495
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# crispr-screen-gene-prioritization

## Summary

Aggregates seven published CRISPR/Cas9 immune-evasion screening cohorts (Freeman, Kearney, Manguso, Pan, Patel, Vredevoogd, Lawson) inside the iMLGAM R environment, prioritizes genes by sgRNA read logFCs between immune-competent and immune-deficient conditions, and links the cross-cohort consensus ranking to univariate Cox regression on the meta cohort to surface immune-evasion targets such as CEP55.

## When to use

Use when multiple published CRISPR/Cas9 screening cohorts with immune-competent vs immune-deficient conditions are available and the goal is a consensus gene ranking tied to iMLGAM-score prognostic markers.

## When NOT to use

- The input is the iMLGAM model-building RNA-seq data (trainset_7/trainset_3/testset .qs files) — use the ABESS/basic_learner/GA_train modeling workflow instead
- Only a single CRISPR cohort is available, so no cross-cohort z-score/mean-rank consensus can be computed
- No survival or clinical annotations exist for the meta cohort, so univariate Cox regression of iMLGAM-score-associated genes cannot be run

## Inputs

- Seven published CRISPR/Cas9 screening cohorts (Freeman, Kearney, Manguso, Pan, Patel, Vredevoogd, Lawson)
- sgRNA read count/logFC data per gene per cohort under immune-competent and immune-deficient conditions
- Meta cohort survival data for univariate Cox regression of iMLGAM-score-associated genes

## Outputs

- crispr_gene_ranking.csv with gene symbols, aggregated z-scores, mean ranks, and Cox regression statistics
- Cross-cohort consensus ranking of candidate immune-evasion genes (CEP55 at rank 1; Figure 4B)

## How to apply

Load the seven published CRISPR/Cas9 screening cohorts into R using the iMLGAM package environment. For each cohort, prioritize genes according to their sgRNA read logFCs between immune-competent and immune-deficient conditions. Aggregate gene-level statistics across the 17 datasets by computing normalized z-scores and mean gene rankings to obtain a cross-cohort consensus ranking. Then perform univariate Cox regression analysis on the meta cohort to identify prognostic markers associated with elevated iMLGAM scores, and export the final ranked gene table (gene symbols, aggregated z-scores, mean ranks, Cox regression statistics) as crispr_gene_ranking.csv.

## Related tools

- **iMLGAM** (R package environment used to load the CRISPR cohorts and compute the iMLGAM scores that genes are prioritized against) — https://github.com/Yelab1994/iMLGAM
- **R** (Implementation language for the cohort loading, sgRNA logFC prioritization, z-score aggregation, and univariate Cox regression)
- **qs** (Library for reading the .qs serialized cohort/example files used in the iMLGAM environment (qread))

## Examples

```
devtools::install_github("Yelab1994/iMLGAM"); library(iMLGAM)
```

## Evaluation signals

- CEP55 appears at rank 1 among prognostic genes associated with high iMLGAM scores (Figure 4B)
- crispr_gene_ranking.csv is a well-formed CSV containing gene symbol, aggregated z-score, mean rank, and Cox regression statistic columns
- The ranking is reproducible from the seven named cohorts (Freeman, Kearney, Manguso, Pan, Patel, Vredevoogd, Lawson) aggregated across 17 datasets via normalized z-scores and mean gene rankings
- Genes are ordered by sgRNA read logFCs between immune-competent and immune-deficient conditions within each cohort before aggregation
- Cox regression statistics correspond to univariate models run on the meta cohort, identifying markers associated with elevated iMLGAM scores

## Limitations

- CEP55 function was validated primarily in lung cancer, lacking extensive verification across other cancer types
- The precise molecular mechanism by which CEP55 regulates PD-L1 expression is not fully elucidated; signal transduction pathways and transcriptional regulatory mechanisms remain incompletely understood
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses

## Evidence

- [discussion] CRISPR datasets were analyzed, with genes prioritized according to their sgRNA read logFCs between immune‐competent and immune‐deficient conditions.: "CRISPR datasets were analyzed, with genes prioritized according to their sgRNA read logFCs between immune‐competent and immune‐deficient conditions."
- [discussion] univariate Cox regression analysis was performed on the meta cohort to identify prognostic markers associated with elevated iMLGAM scores, leading to the identification of CEP55: "univariate Cox regression analysis was performed on the meta cohort to identify prognostic markers associated with elevated iMLGAM scores, leading to the identification of CEP55"
- [abstract] through clustered regularly interspaced short palindromic repeats screening, we identified Centrosomal Protein 55 (CEP55) as a key molecule modulating tumor immune evasion: "through clustered regularly interspaced short palindromic repeats screening, we identified Centrosomal Protein 55 (CEP55) as a key molecule modulating tumor immune evasion"
- [readme] devtools::install_github("Yelab1994/iMLGAM"): "devtools::install_github("Yelab1994/iMLGAM")"
