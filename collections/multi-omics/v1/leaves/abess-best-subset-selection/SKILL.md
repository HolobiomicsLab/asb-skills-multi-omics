---
name: abess-best-subset-selection
description: Use when a training-set RNA-seq feature matrix (e.g. gene-pair columns
  with a 'res' response) is too large for direct modeling and a small, information-maximizing
  feature subset must be selected before fitting base learners.
when_to_use_negative:
- The feature matrix already contains only a small curated feature set, so best-subset
  selection would add no reduction and may discard informative pairs
- Rows have missing 'res' labels and cannot be cleaned first — ABESS fitting on the
  response requires complete training labels
- The goal is selecting from non-feature variables such as predicted base-learner
  matrices — that stage is handled by GA_train(), not ABESS
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: abess
  role: R library providing the abess() adaptive best-subset selection fit and extract()
    function used to pull the top 5 ranked gene pairs
- name: qs
  role: Loads the .qs cohort files ('Sample File.qs') whose trainset_7 split supplies
    the ABESS training matrix
- name: iMLGAM
  role: R package whose pipeline embeds the ABESS feature-selection step before basic_learner()
    construction
  repo: https://github.com/Yelab1994/iMLGAM
- name: devtools
  role: Installs the iMLGAM package from GitHub via devtools::install_github()
  repo: https://github.com/Yelab1994/iMLGAM
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1002/imt2.70011
    title: 'iMLGAM: Integrated Machine Learning and Genetic Algorithm‐driven Multiomics
      analysis for pan‐cancer immunotherapy response prediction'
  source_content_hash: sha256:d0d54c5fcedc2c380bbf10d61931cf201900a9869eb9415359ea1e2dfaabdfcc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Yelab1994/iMLGAM
  tools:
  - R
  - abess
  - devtools
  - iMLGAM
  - qs
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# abess-best-subset-selection

## Summary

Applies adaptive best-subset selection (the R abess package) to a high-dimensional gene-pair feature matrix from an ICB cohort to identify a small, rank-ordered set of discriminative features (top gene pairs) before ensemble base-learner construction in the iMLGAM pipeline.

## When to use

Use when a training-set RNA-seq feature matrix (e.g. gene-pair columns with a 'res' response) is too large for direct modeling and a small, information-maximizing feature subset must be selected before fitting base learners.

## When NOT to use

- The feature matrix already contains only a small curated feature set, so best-subset selection would add no reduction and may discard informative pairs
- Rows have missing 'res' labels and cannot be cleaned first — ABESS fitting on the response requires complete training labels
- The goal is selecting from non-feature variables such as predicted base-learner matrices — that stage is handled by GA_train(), not ABESS

## Inputs

- A .qs-serialized cohort object (e.g. 'Sample File.qs') loaded with qs::qread, containing trainset_7, trainset_3, and testset data frames whose first column is the outcome 'res' and remaining columns are gene-pair features
- A feature matrix with gene-pair columns and a binary/ordinal ICB response column 'res'

## Outputs

- An abess fitted object (abess_fit) over support sizes 0:100
- A character vector of the top 5 selected gene pairs (key_gene) from extract(abess_fit, 5)[4][1]$support.vars
- Feature-subsetted trainset, validationset, and testset data frames restricted to c('res', key_gene)

## How to apply

First remove training rows with missing response labels ('res') and split the cohort into trainset (70%), validationset (30%), and testset. Fit the ABESS model with abess(trainset[,-1], trainset$res, support.size=0:100), letting the algorithm scan support sizes from 0 to 100 gene pairs. Extract the top 5 ranked gene pairs via extract(abess_fit, 5)[4][1]$support.vars, then subset the trainset, validationset, and testset identically to the columns c('res', key_gene) so all three cohorts carry the same reduced feature space. The rationale is to reduce the gene-pair space to a manageable, batch-effect-robust signature prior to the basic_learner() ensemble and GA-optimized stacking steps of iMLGAM.

## Related tools

- **abess** (R library providing the abess() adaptive best-subset selection fit and extract() function used to pull the top 5 ranked gene pairs)
- **qs** (Loads the .qs cohort files ('Sample File.qs') whose trainset_7 split supplies the ABESS training matrix)
- **iMLGAM** (R package whose pipeline embeds the ABESS feature-selection step before basic_learner() construction) — https://github.com/Yelab1994/iMLGAM
- **devtools** (Installs the iMLGAM package from GitHub via devtools::install_github()) — https://github.com/Yelab1994/iMLGAM

## Examples

```
abess_fit <- abess(trainset[,-1], trainset$res, support.size=0:100); key_gene <- extract(abess_fit, 5)[4][1]$support.vars
```

## Evaluation signals

- abess_fit must have been fitted with support.size=0:100 over all feature columns of trainset (excluding the 'res' column)
- extract(abess_fit, 5) must return a support.vars object containing exactly 5 gene-pair features
- trainset, validationset, and testset must all be subsetted to identical columns c('res', key_gene) after selection
- Training rows with NA 'res' must be removed before abess() is called (trainset <- trainset[!is.na(trainset$res),])
- The downstream basic_learner() call must run on the ABESS-reduced matrix without dimension-mismatch errors across the three cohorts

## Limitations

- The default extraction takes the top 5 gene pairs, which constrains the signature size; other support sizes within 0:100 are explored but only rank 5 is retained in the reference workflow
- ABESS is applied only to the training split; selected pairs assume the 70/30 train–validation partitioning already present in the sample file
- Selection is on RNA-seq-derived gene pairs from cohorts spanning only six cancer types, so pan-cancer generalizability of the selected feature set requires further prospective ICB-trial validation

## Evidence

- [readme] abess support-size scan and top-5 extraction in the reference workflow: "abess_fit <- abess(trainset[,-1], trainset$res,support.size=0:100)
key_gene <- extract(abess_fit, 5)[4][1]$support.vars"
- [methods] ABESS feature-selection step with NA-label removal in the article methods: "Feature Selection using ABESS Algorithm
trainset <- trainset[!is.na(trainset$res),]"
- [intro] Article intro situating ABESS within the iMLGAM objective: [evidence span withheld — anchor status: not_found]
- [readme] Identical subsetting of validation and test cohorts to the selected pairs: "validationset  <- validationset [,c("res",key_gene )]
testset <- testset [,c("res",key_gene )]"
