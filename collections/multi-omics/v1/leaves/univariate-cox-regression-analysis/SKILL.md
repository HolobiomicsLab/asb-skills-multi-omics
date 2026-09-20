---
name: univariate-cox-regression-analysis
description: Use when you have survival-annotated cohort data plus a computed iMLGAM
  score (or similar risk score) and need to identify prognostic markers, or when ranking
  CRISPR-screening hits by prognostic relevance in immune-evasion contexts.
when_to_use_negative:
- Input is already a feature/signature table without survival (time-to-event) annotations
  — Cox regression requires survival outcomes.
- The goal is ICB response prediction modeling (base-learner construction, ABESS feature
  selection, GA optimization) rather than prognostic marker identification — use the
  iMLGAM scoring workflow instead.
- Cohorts lack clinical annotations needed for survival analysis, as noted in the
  article's limitation that 'some datasets lack critical clinical annotations, hindering
  comprehensive biomarker analyses'.
edam_operation: http://edamontology.org/operation_3561
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: iMLGAM
  role: R package providing the environment and scoring system; iMLGAM scores stratify
    the meta cohort so univariate Cox regression can identify prognostic markers associated
    with elevated scores
  repo: https://github.com/Yelab1994/iMLGAM
- name: R
  role: Runtime language in which the Cox regression, cohort loading, and gene-ranking
    aggregation are executed
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:0404e51fd5968fdd0fb0e1605e96df498fdad336a2590e0a79692a948803ad70
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - iMLGAM
  edam_operation: http://edamontology.org/operation_3561
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# univariate-cox-regression-analysis

## Summary

Perform univariate Cox regression on a meta cohort within the iMLGAM R environment to identify genes (e.g., CEP55) whose status is prognostic for patients with elevated iMLGAM scores, and combine the results with aggregated CRISPR-screening z-scores to rank immune-evasion targets.

## When to use

Use when you have survival-annotated cohort data plus a computed iMLGAM score (or similar risk score) and need to identify prognostic markers, or when ranking CRISPR-screening hits by prognostic relevance in immune-evasion contexts.

## When NOT to use

- Input is already a feature/signature table without survival (time-to-event) annotations — Cox regression requires survival outcomes.
- The goal is ICB response prediction modeling (base-learner construction, ABESS feature selection, GA optimization) rather than prognostic marker identification — use the iMLGAM scoring workflow instead.
- Cohorts lack clinical annotations needed for survival analysis, as noted in the article's limitation that 'some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses'.

## Inputs

- Seven published CRISPR/Cas9 screening cohorts (Freeman, Kearney, Manguso, Pan, Patel, Vredevoogd, Lawson) with sgRNA read logFC values
- Meta-cohort transcriptomic and survival data (e.g., pan-cancer TCGA datasets) with computed iMLGAM scores per sample

## Outputs

- crispr_gene_ranking.csv: ranked gene table with gene symbols, aggregated z-scores, mean ranks, and Cox regression statistics
- Ranked list of prognostic genes associated with high iMLGAM scores (CEP55 expected at rank 1)

## How to apply

First prioritize candidate genes from CRISPR/Cas9 screening cohorts (Freeman, Kearney, Manguso, Pan, Patel, Vredevoogd, Lawson) according to sgRNA read logFCs between immune-competent and immune-deficient conditions, aggregating gene-level statistics via normalized z-scores and mean gene rankings across the 17 datasets. Then, in the iMLGAM package environment in R, run univariate Cox regression on the meta cohort (e.g., TCGA-derived pan-cancer transcriptomic data) to test each candidate gene's association with survival among patients with elevated iMLGAM scores. Export a ranked gene table (crispr_gene_ranking.csv) with gene symbols, aggregated z-scores, mean ranks, and Cox regression statistics. Validation: confirm the CSV is well-formed and that CEP55 appears at rank 1 among prognostic genes associated with high iMLGAM scores, as reported in Figure 4B of the source study.

## Related tools

- **iMLGAM** (R package providing the environment and scoring system; iMLGAM scores stratify the meta cohort so univariate Cox regression can identify prognostic markers associated with elevated scores) — https://github.com/Yelab1994/iMLGAM
- **R** (Runtime language in which the Cox regression, cohort loading, and gene-ranking aggregation are executed)

## Examples

```
library(iMLGAM); cox_fit <- univariate_cox(meta_cohort, surv_time, surv_event, genes = ranked_genes)
```

## Evaluation signals

- The output crispr_gene_ranking.csv parses as a well-formed CSV containing gene symbol, aggregated z-score, mean rank, and Cox regression statistics columns.
- CEP55 appears at rank 1 among prognostic genes associated with high iMLGAM scores, reproducing Figure 4B of the article.
- Gene prioritization step yields sgRNA read logFC-based rankings for all seven named CRISPR cohorts (Freeman, Kearney, Manguso, Pan, Patel, Vredevoogd, Lawson) covering the 17 datasets.
- Aggregated z-scores and mean ranks are consistent across the cross-cohort consensus (no missing values for genes present in all cohorts).

## Limitations

- CEP55 function was validated primarily in lung cancer, lacking extensive verification across other cancer types.
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses.
- The iMLGAM score's pan-cancer predictive efficacy requires further validation in prospective ICB trials; analyzed RNA-seq cohorts cover only six cancer types.

## Evidence

- [discussion] genes prioritized according to their sgRNA read logFCs between immune-competent and immune-deficient conditions: "CRISPR datasets were analyzed, with genes prioritized according to their sgRNA read logFCs between immune‐competent and immune‐deficient conditions."
- [discussion] univariate Cox regression identifies CEP55 as prognostic marker: "univariate Cox regression analysis was performed on the meta cohort to identify prognostic markers associated with elevated iMLGAM scores, leading to the identification of CEP55"
- [abstract] CEP55 identified as key immune-evasion molecule: "through clustered regularly interspaced short palindromic repeats screening, we identified Centrosomal Protein 55 (CEP55) as a key molecule modulating tumor immune evasion"
- [readme] install and load iMLGAM in R: "devtools::install_github("Yelab1994/iMLGAM")"
