---
name: mean-rank-gene-meta-analysis
description: Use when several independent CRISPR/Cas9 screening cohorts must be aggregated
  into one consensus gene ranking to prioritize immune-evasion or ICB-response targets.
when_to_use_negative:
- You have a single CRISPR cohort only — z-score/mean-rank aggregation across 17 datasets
  requires multiple cohorts to be meaningful
- Your goal is training the iMLGAM prediction model itself (base learners, ABESS feature
  selection, GA_train, risk scoring) rather than prioritizing immune-evasion target
  genes
- Your screens lack paired immune-competent vs immune-deficient conditions, so sgRNA
  read logFCs between those conditions cannot be computed
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: iMLGAM
  role: R package providing the environment for CRISPR cohort analysis, gene prioritization,
    z-score/mean-rank aggregation, and Cox regression in the meta cohort
  repo: https://github.com/Yelab1994/iMLGAM
- name: R
  role: Runtime environment in which the iMLGAM package and the meta-analysis workflow
    are executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:768720aeafcd8ad81a59d3a4bf1b3afc3baf3d2edc7532b286abe96852dc187f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - iMLGAM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# mean-rank-gene-meta-analysis

## Summary

Aggregates multiple CRISPR/Cas9 screening cohorts into a cross-cohort consensus gene ranking by computing normalized gene-level z-scores and mean gene ranks, then links ranked genes to prognosis via univariate Cox regression on a meta cohort. In the iMLGAM study this workflow ranked CEP55 first among prognostic genes associated with a high iMLGAM score (Figure 4B).

## When to use

Use when several independent CRISPR/Cas9 screening cohorts must be aggregated into one consensus gene ranking to prioritize immune-evasion or ICB-response targets.

## When NOT to use

- You have a single CRISPR cohort only — z-score/mean-rank aggregation across 17 datasets requires multiple cohorts to be meaningful
- Your goal is training the iMLGAM prediction model itself (base learners, ABESS feature selection, GA_train, risk scoring) rather than prioritizing immune-evasion target genes
- Your screens lack paired immune-competent vs immune-deficient conditions, so sgRNA read logFCs between those conditions cannot be computed

## Inputs

- sgRNA read logFC tables from seven CRISPR/Cas9 screening cohorts comparing immune-competent vs immune-deficient conditions (Freeman, Kearney, Manguso, Pan, Patel, Vredevoogd, Lawson)
- Meta-cohort gene expression and survival data for univariate Cox regression against iMLGAM score strata

## Outputs

- crispr_gene_ranking.csv: gene symbols, aggregated normalized z-scores, mean ranks, and Cox regression statistics
- A consensus ranked gene list visualizable as Figure 4B (CEP55 at rank 1 among prognostic genes associated with high iMLGAM scores)

## How to apply

Load each published CRISPR/Cas9 screening cohort (here: Freeman, Kearney, Manguso, Pan, Patel, Vredevoogd, Lawson) into the R/iMLGAM package environment. Within each cohort, prioritize genes according to their sgRNA read logFCs between immune-competent and immune-deficient conditions. Aggregate gene-level statistics across the datasets by computing normalized z-scores and mean gene rankings to obtain a cross-cohort consensus ranking. On the meta cohort, perform univariate Cox regression to identify which ranked genes are prognostic for elevated iMLGAM scores. Export the ranked table (gene symbols, aggregated z-scores, mean ranks, Cox regression statistics) and validate that the expected top hit, CEP55, appears at rank 1 among the prognostic genes.

## Related tools

- **iMLGAM** (R package providing the environment for CRISPR cohort analysis, gene prioritization, z-score/mean-rank aggregation, and Cox regression in the meta cohort) — https://github.com/Yelab1994/iMLGAM
- **R** (Runtime environment in which the iMLGAM package and the meta-analysis workflow are executed)

## Examples

```
devtools::install_github("Yelab1994/iMLGAM"); library(iMLGAM); library(qs)
```

## Evaluation signals

- Output file is a well-formed CSV containing gene symbol, aggregated z-score, mean rank, and Cox regression columns
- CEP55 appears at rank 1 among prognostic genes associated with high iMLGAM scores, reproducing Figure 4B
- Z-scores are normalized across datasets before averaging, so per-cohort contributions are comparable
- Every ranked gene has Cox regression statistics computed on the meta cohort

## Limitations

- CEP55 function was validated primarily in lung cancer, lacking extensive verification across other cancer types
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses
- The precise molecular mechanism by which CEP55 regulates PD-L1 expression is not fully elucidated

## Evidence

- [discussion] CRISPR datasets were analyzed, with genes prioritized according to their sgRNA read logFCs between immune-competent and immune-deficient conditions.: [evidence span withheld — anchor status: not_found]
- [discussion] univariate Cox regression analysis was performed on the meta cohort to identify prognostic markers associated with elevated iMLGAM scores, leading to the identification of CEP55: "univariate Cox regression analysis was performed on the meta cohort to identify prognostic markers associated with elevated iMLGAM scores, leading to the identification of CEP55"
- [abstract] through clustered regularly interspaced short palindromic repeats screening, we identified Centrosomal Protein 55 (CEP55) as a key molecule modulating tumor immune evasion: "through clustered regularly interspaced short palindromic repeats screening, we identified Centrosomal Protein 55 (CEP55) as a key molecule modulating tumor immune evasion"
- [readme] devtools::install_github("Yelab1994/iMLGAM"): "devtools::install_github("Yelab1994/iMLGAM")"
