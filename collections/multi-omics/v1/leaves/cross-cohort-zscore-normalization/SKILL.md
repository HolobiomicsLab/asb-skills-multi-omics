---
name: cross-cohort-zscore-normalization
description: Use when sgRNA read logFCs from several independent CRISPR screening
  cohorts (e.g. Freeman, Kearney, Manguso, Pan, Patel, Vredevoogd, Lawson) must be
  combined into a single consensus gene ranking.
when_to_use_negative:
- Only a single CRISPR screening dataset is available — there is nothing to aggregate
  across cohorts
- The input data lack sgRNA logFCs between immune-competent and immune-deficient conditions,
  so z-score aggregation of screening statistics is not applicable
- The goal is computing iMLGAM risk scores from transcriptomic data (Steps 1–4 of
  the iMLGAM package workflow), not prioritizing immune-evasion genes from screens
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Analysis environment in which the seven CRISPR cohorts are loaded, z-scores
    and mean ranks are computed, and the ranked CSV is exported
  repo: https://github.com/Yelab1994/iMLGAM
- name: iMLGAM
  role: R package environment used to load the cohorts and support gene prioritization
    and Cox regression on the meta cohort
  repo: https://github.com/Yelab1994/iMLGAM
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:9dd44a3575be4b090cf96eec62de27881ca5f73387ba7041b80cf9e3a39bb98a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - iMLGAM
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# cross-cohort-zscore-normalization

## Summary

Aggregates gene-level statistics across multiple CRISPR/Cas9 screening cohorts by computing normalized z-scores and mean gene rankings, yielding a cross-cohort consensus ranking of immune-evasion genes. In the iMLGAM study this procedure ranked CEP55 first among prognostic genes associated with a high iMLGAM score (Figure 4B).

## When to use

Use when sgRNA read logFCs from several independent CRISPR screening cohorts (e.g. Freeman, Kearney, Manguso, Pan, Patel, Vredevoogd, Lawson) must be combined into a single consensus gene ranking.

## When NOT to use

- Only a single CRISPR screening dataset is available — there is nothing to aggregate across cohorts
- The input data lack sgRNA logFCs between immune-competent and immune-deficient conditions, so z-score aggregation of screening statistics is not applicable
- The goal is computing iMLGAM risk scores from transcriptomic data (Steps 1–4 of the iMLGAM package workflow), not prioritizing immune-evasion genes from screens

## Inputs

- Seven published CRISPR/Cas9 screening cohorts: Freeman, Kearney, Manguso, Pan, Patel, Vredevoogd, Lawson
- Per-gene sgRNA read logFC values comparing immune-competent vs immune-deficient conditions
- Meta-cohort survival data for univariate Cox regression against iMLGAM scores

## Outputs

- Cross-cohort consensus gene ranking based on normalized z-scores and mean gene ranks
- crispr_gene_ranking.csv with gene symbols, aggregated z-scores, mean ranks, and Cox regression statistics

## How to apply

Load the seven published CRISPR/Cas9 screening cohorts (Freeman, Kearney, Manguso, Pan, Patel, Vredevoogd, Lawson) into an R environment with the iMLGAM package. For each cohort, prioritize genes according to their sgRNA read logFCs between immune-competent and immune-deficient conditions. Aggregate gene-level statistics across the 17 underlying datasets by computing normalized z-scores and mean gene rankings to obtain a cross-cohort consensus ranking. Downstream, perform univariate Cox regression on the meta cohort to flag which consensus-ranked genes are prognostic markers associated with elevated iMLGAM scores, and export the ranked table (gene symbols, aggregated z-scores, mean ranks, Cox statistics) as crispr_gene_ranking.csv.

## Related tools

- **R** (Analysis environment in which the seven CRISPR cohorts are loaded, z-scores and mean ranks are computed, and the ranked CSV is exported) — https://github.com/Yelab1994/iMLGAM
- **iMLGAM** (R package environment used to load the cohorts and support gene prioritization and Cox regression on the meta cohort) — https://github.com/Yelab1994/iMLGAM

## Examples

```
devtools::install_github("Yelab1994/iMLGAM"); library(iMLGAM)
```

## Evaluation signals

- CEP55 appears at rank 1 among prognostic genes associated with high iMLGAM scores in the consensus ranking (expected to reproduce Figure 4B)
- Output is a well-formed CSV (crispr_gene_ranking.csv) containing gene symbols, aggregated z-scores, mean ranks, and Cox regression statistics
- Each of the seven cohorts (Freeman, Kearney, Manguso, Pan, Patel, Vredevoogd, Lawson) contributed gene-level statistics prior to aggregation across the 17 datasets
- Genes were prioritized by sgRNA read logFCs between immune-competent and immune-deficient conditions before z-score aggregation

## Limitations

- CEP55 function was validated primarily in lung cancer, lacking extensive verification across other cancer types
- Some datasets lack critical clinical annotations, hindering comprehensive biomarker analyses
- No large-scale, multi-center pan-cancer cohort validation, limiting the generalizability of the derived rankings

## Evidence

- [discussion] CRISPR datasets were analyzed, with genes prioritized according to their sgRNA read logFCs between immune‐competent and immune‐deficient conditions.: "CRISPR datasets were analyzed, with genes prioritized according to their sgRNA read logFCs between immune‐competent and immune‐deficient conditions."
- [discussion] univariate Cox regression on meta cohort identifying CEP55: "univariate Cox regression analysis was performed on the meta cohort to identify prognostic markers associated with elevated iMLGAM scores, leading to the identification of CEP55"
- [abstract] CEP55 identified as key immune-evasion molecule via CRISPR screening: "through clustered regularly interspaced short palindromic repeats screening, we identified Centrosomal Protein 55 (CEP55) as a key molecule modulating tumor immune evasion"
- [readme] iMLGAM is an R package: "Make sure you have R and the following packages installed: iMLAGA. Install "iMLAGA" via this [GitHub page](https://github.com/Yelab1994/iMLAGA) or by running the code."
