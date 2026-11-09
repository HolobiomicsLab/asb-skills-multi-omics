---
name: microbial-phenotype-data-preprocessing
description: Use when a phenotypic trait table contains continuous (numeric) columns
  that must be binarized before Scoary2 genome-wide association, and uncertain isolates
  (posterior probability < 0.85) should be excluded rather than forced into a binary
  label.
when_to_use_negative:
- Trait columns are already binary (0/1) labels — binarization via k-means/GMM is
  unnecessary and could distort defined phenotypes
- Using the original Scoary, which lacks the ability to pre-process numeric traits
  and cannot handle this automatic binarization
- When a trait's distribution is unimodal or the two-component GMM is a poor fit,
  since forced two-cluster separation of uncertain isolates would be misleading
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: Scoary2
  role: Provides the automatic binarization of continuous phenotypes and the 85% posterior-probability
    retention rule; used via its Python library interface to produce binary trait
    tables for mGWAS
  repo: https://github.com/MrTomRod/scoary-2
- name: OrthoFinder
  role: Generates the orthogenes that the binarized traits are subsequently associated
    against in Scoary2
  repo: https://github.com/davidemms/OrthoFinder
- name: Roary
  role: Alternative tool to OrthoFinder for generating orthogenes used in the association
    step after trait binarization
  repo: https://sanger-pathogens.github.io/Roary/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:811bdf9cb6b19d0f39cf3e8ca8416158fcdf17111b54fbeaf17fbae05c562afd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MrTomRod/scoary-2
  tools:
  - OrthoFinder
  - Roary
  - Scoary2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# microbial-phenotype-data-preprocessing

## Summary

Preprocesses continuous (numeric) microbial phenotype/omics measurements — e.g. yogurt metabolome traits — into binary trait labels for Scoary2 mGWAS, using k-means (k=2) for hard labels and a two-component Gaussian mixture model to discard isolates classified with less than 85% posterior probability.

## When to use

Use when a phenotypic trait table contains continuous (numeric) columns that must be binarized before Scoary2 genome-wide association, and uncertain isolates (posterior probability < 0.85) should be excluded rather than forced into a binary label.

## When NOT to use

- Trait columns are already binary (0/1) labels — binarization via k-means/GMM is unnecessary and could distort defined phenotypes
- Using the original Scoary, which lacks the ability to pre-process numeric traits and cannot handle this automatic binarization
- When a trait's distribution is unimodal or the two-component GMM is a poor fit, since forced two-cluster separation of uncertain isolates would be misleading

## Inputs

- Phenotypic/omics trait table (e.g. yogurt metabolome measurements) with continuous numeric trait columns
- Isolate IDs keyed to the pan-genome / gene presence-absence matrix produced by OrthoFinder or Roary

## Outputs

- binary_traits.csv — binary (0/1) or missing values per trait column, keyed by isolate ID
- trait_probabilities.csv — per-isolate posterior probabilities of GMM class assignment for each trait

## How to apply

Load the trait table and identify the continuous (numeric) trait columns requiring binarization via Scoary2's Python library interface (github.com/MrTomRod/scoary-2). For each column, fit a k-means model with k=2 to assign each isolate a hard binary label from cluster membership, and separately fit a two-component Gaussian mixture model. Compute the posterior probability of each isolate's assigned GMM class and apply the Scoary2 retention rule: keep the binary assignment only when the posterior probability is at least 85%, otherwise mark the isolate's trait value as missing/discarded, since Scoary2 by default ignores isolates classified with less than 85% predicted posterior probability. Assemble retained calls into a binary trait table keyed by isolate ID and write the per-isolate posterior probabilities to a separate table, producing binary_traits.csv and trait_probabilities.csv.

## Related tools

- **Scoary2** (Provides the automatic binarization of continuous phenotypes and the 85% posterior-probability retention rule; used via its Python library interface to produce binary trait tables for mGWAS) — https://github.com/MrTomRod/scoary-2
- **OrthoFinder** (Generates the orthogenes that the binarized traits are subsequently associated against in Scoary2) — https://github.com/davidemms/OrthoFinder
- **Roary** (Alternative tool to OrthoFinder for generating orthogenes used in the association step after trait binarization) — https://sanger-pathogens.github.io/Roary/

## Examples

```
from sklearn.mixture import GaussianMixture; prob = GaussianMixture(n_components=2).fit_predict_proba(X); keep = prob.max(axis=1) >= 0.85
```

## Evaluation signals

- binary_traits.csv contains only binary (0/1) or missing values per trait column — no forced labels for uncertain isolates
- Every retained binary assignment has a recorded posterior probability >= 0.85 in trait_probabilities.csv
- Row counts of both output tables match the number of isolates in the input dataset
- Every continuous (numeric) trait column from the input table is represented in the output binary trait table
- Discarded isolates (posterior < 0.85) appear as missing values rather than binary labels

## Limitations

- Isolates classified with less than 85% predicted posterior probability are ignored from further analysis by default, reducing effective sample size for affected traits
- Binarization assumes each continuous trait separates into two classes; traits with more complex distributions may be poorly represented by a k=2 / two-component model
- The original Scoary could not pre-process numeric traits at all — this preprocessing capability is specific to Scoary2

## Evidence

- [other] Scoary2 automatic binarization retention rule: [evidence span withheld — anchor status: not_found]
- [intro] Scoary limitation motivating numeric trait preprocessing: "Three limitations prevent Scoary from such high-throughput analyses: performance constraints, the inability to pre-process numeric traits, and a lack of post-GWAS methods"
- [abstract] Scoary2 enables large phenotypic datasets: "With its data exploration app and improved performance, Scoary2 is the first tool to enable the study of large phenotypic datasets using mGWAS"
- [readme] Scoary2 output context from repository README: "It reports a list of genes sorted by strength of association per trait. The results can be explored interactively with a simple, static HTML/JS app."
