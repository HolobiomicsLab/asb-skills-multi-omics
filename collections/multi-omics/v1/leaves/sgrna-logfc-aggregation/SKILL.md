---
name: sgrna-logfc-aggregation
description: Use when multiple CRISPR/Cas9 screening cohorts (e.g. Freeman, Kearney,
  Manguso, Pan, Patel, Vredevoogd, Lawson) must be combined into a consensus gene
  ranking of immune-evasion or immunotherapy-response modifiers.
when_to_use_negative:
- The task is fitting the iMLGAM prediction model itself (ABESS feature selection,
  base learners, genetic algorithm, risk scores) rather than ranking CRISPR screen
  hits
- Only a single CRISPR cohort is available with no possibility of cross-cohort z-score
  aggregation
- Data are bulk transcriptomic expression profiles without sgRNA-level logFCs between
  immune-competent and immune-deficient conditions
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: iMLGAM
  role: R package environment in which the CRISPR cohorts are loaded, aggregated by
    z-scores/mean ranks, and linked to iMLGAM-score Cox regression results
  repo: https://github.com/Yelab1994/iMLGAM
- name: R
  role: Analysis language for loading cohorts, computing sgRNA logFC-based gene prioritization,
    z-score aggregation, and univariate Cox regression
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:4d307ed2b1b46825b07e54af29ffaa632e3db04568ce542b0ba5de68dba4b0a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - iMLGAM
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# sgrna-logfc-aggregation

## Summary

Aggregates multiple published CRISPR/Cas9 screening cohorts by prioritizing genes according to sgRNA read logFCs between immune-competent and immune-deficient conditions, then computes cross-cohort normalized z-scores and mean gene rankings to identify immune-evasion targets such as CEP55.

## When to use

Use when multiple CRISPR/Cas9 screening cohorts (e.g. Freeman, Kearney, Manguso, Pan, Patel, Vredevoogd, Lawson) must be combined into a consensus gene ranking of immune-evasion or immunotherapy-response modifiers.

## When NOT to use

- The task is fitting the iMLGAM prediction model itself (ABESS feature selection, base learners, genetic algorithm, risk scores) rather than ranking CRISPR screen hits
- Only a single CRISPR cohort is available with no possibility of cross-cohort z-score aggregation
- Data are bulk transcriptomic expression profiles without sgRNA-level logFCs between immune-competent and immune-deficient conditions

## Inputs

- Seven published CRISPR/Cas9 screening cohorts (Freeman, Kearney, Manguso, Pan, Patel, Vredevoogd, Lawson) with sgRNA read counts/logFCs for immune-competent vs immune-deficient conditions
- Meta cohort with survival/clinical annotations for Cox regression
- iMLGAM scores per sample to stratify prognostic genes

## Outputs

- crispr_gene_ranking.csv with gene symbols, aggregated z-scores, mean ranks, and univariate Cox regression statistics
- Consensus cross-cohort gene ranking with CEP55 at rank 1 among prognostic genes associated with high iMLGAM scores

## How to apply

Load each CRISPR/Cas9 screening cohort into R within the iMLGAM package environment. For each cohort, prioritize genes according to their sgRNA read logFCs between immune-competent and immune-deficient conditions. Aggregate gene-level statistics across cohorts (17 datasets in the source study) by computing normalized z-scores and mean gene rankings to obtain a cross-cohort consensus. Separately, run univariate Cox regression on the meta cohort to flag which ranked genes are prognostic markers associated with elevated iMLGAM scores. Export the ranked table (gene symbols, aggregated z-scores, mean ranks, Cox statistics) and verify that the expected top hit — CEP55 at rank 1 among high-iMLGAM prognostic genes (Figure 4B) — reproduces.

## Related tools

- **iMLGAM** (R package environment in which the CRISPR cohorts are loaded, aggregated by z-scores/mean ranks, and linked to iMLGAM-score Cox regression results) — https://github.com/Yelab1994/iMLGAM
- **R** (Analysis language for loading cohorts, computing sgRNA logFC-based gene prioritization, z-score aggregation, and univariate Cox regression)

## Examples

```
devtools::install_github("Yelab1994/iMLGAM"); library(iMLGAM)
```

## Evaluation signals

- Output CSV is well-formed and parses without errors
- CEP55 appears at rank 1 among prognostic genes associated with high iMLGAM scores, matching Figure 4B
- The ranked table contains the required columns: gene symbols, aggregated z-scores, mean ranks, and Cox regression statistics
- All seven cohorts (Freeman, Kearney, Manguso, Pan, Patel, Vredevoogd, Lawson) contribute to the aggregation across 17 datasets
- Univariate Cox regression results identify CEP55 as prognostic for elevated iMLGAM scores

## Limitations

- CEP55 function was validated primarily in lung cancer, lacking extensive verification across other cancer types
- The precise molecular mechanism by which CEP55 regulates PD-L1 expression is not fully elucidated; signal transduction and transcriptional regulatory mechanisms remain incompletely understood
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses

## Evidence

- [discussion] CRISPR datasets were analyzed, with genes prioritized according to their sgRNA read logFCs between immune‐competent and immune‐deficient conditions.: "CRISPR datasets were analyzed, with genes prioritized according to their sgRNA read logFCs between immune‐competent and immune‐deficient conditions."
- [discussion] univariate Cox regression analysis was performed on the meta cohort to identify prognostic markers associated with elevated iMLGAM scores, leading to the identification of CEP55: "univariate Cox regression analysis was performed on the meta cohort to identify prognostic markers associated with elevated iMLGAM scores, leading to the identification of CEP55"
- [abstract] through clustered regularly interspaced short palindromic repeats screening, we identified Centrosomal Protein 55 (CEP55) as a key molecule modulating tumor immune evasion: "through clustered regularly interspaced short palindromic repeats screening, we identified Centrosomal Protein 55 (CEP55) as a key molecule modulating tumor immune evasion"
- [readme] devtools::install_github("Yelab1994/iMLGAM"): "devtools::install_github("Yelab1994/iMLGAM")"
