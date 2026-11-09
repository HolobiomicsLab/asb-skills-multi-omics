---
name: phenotype-trait-binarization
description: Use when a trait table contains continuous (numeric) phenotype columns
  that must be dichotomized for a binary GWAS tool such as Scoary2, and isolates with
  ambiguous trait classification should be excluded rather than forced into a class.
when_to_use_negative:
- The trait columns are already binary (present/absent or 0/1) — Scoary2 can consume
  them directly without binarization
- A high-confidence discrete phenotype is available and continuous modeling would
  only add uncertainty (e.g. manually curated categorical labels)
- The goal is quantitative-trait (continuous) association rather than binary classification
  of isolates
tools:
- name: Scoary2
  role: Target mGWAS tool; performs automatic binarization of continuous traits and
    ignores isolates classified below the 85% posterior-probability cutoff; also usable
    as a Python library for the binarization workflow
  repo: https://github.com/MrTomRod/scoary-2
- name: OrthoFinder
  role: Upstream orthogroup generator whose orthogenes the binarized traits are subsequently
    associated against by Scoary2
  repo: https://github.com/davidemms/OrthoFinder
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:3024b8ea3e607a598533649275b7e30495013cf112a15d40b32614dbd3d8591b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MrTomRod/scoary-2
  tools:
  - OrthoFinder
  - Scoary2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# phenotype-trait-binarization

## Summary

Converts continuous phenotypic traits (e.g. yogurt metabolome measurements analyzed with Scoary2) into binary labels by combining k-means clustering with a two-component Gaussian mixture model, discarding isolates whose class assignment falls below an 85% posterior-probability cutoff.

## When to use

Use when a trait table contains continuous (numeric) phenotype columns that must be dichotomized for a binary GWAS tool such as Scoary2, and isolates with ambiguous trait classification should be excluded rather than forced into a class.

## When NOT to use

- The trait columns are already binary (present/absent or 0/1) — Scoary2 can consume them directly without binarization
- A high-confidence discrete phenotype is available and continuous modeling would only add uncertainty (e.g. manually curated categorical labels)
- The goal is quantitative-trait (continuous) association rather than binary classification of isolates

## Inputs

- Continuous (numeric) phenotypic trait table keyed by isolate ID (e.g. yogurt metabolome trait measurements)
- Isolate/strain identifiers matching the pan-genome sample set

## Outputs

- binary_traits.csv: binary (0/1 or missing) trait table, one column per continuous trait, one row per isolate
- trait_probabilities.csv: per-isolate posterior probabilities from the Gaussian mixture model for each trait

## How to apply

Load the phenotypic trait table (e.g. the Agroscope yogurt trait data used with Scoary2) and identify the numeric columns requiring binarization via Scoary2's Python library interface (github.com/MrTomRod/scoary-2). For each continuous trait, fit k-means with k=2 to assign a hard binary label per isolate, then fit a two-component Gaussian mixture model and compute the posterior probability of each isolate's assigned class. Apply Scoary2's default retention rule: keep a binary assignment only when the predicted posterior probability is at least 85%; otherwise treat the trait value as missing/discarded. Rationale: GWAS tools like Scoary2 operate on binary traits, and forcing uncertain classifications into binary labels would inject noise into gene-trait association, so isolates with less than 85% predicted posterior probability are by default ignored from further analysis.

## Related tools

- **Scoary2** (Target mGWAS tool; performs automatic binarization of continuous traits and ignores isolates classified below the 85% posterior-probability cutoff; also usable as a Python library for the binarization workflow) — https://github.com/MrTomRod/scoary-2
- **OrthoFinder** (Upstream orthogroup generator whose orthogenes the binarized traits are subsequently associated against by Scoary2) — https://github.com/davidemms/OrthoFinder

## Evaluation signals

- binary_traits.csv contains only binary or missing values in each trait column — no raw continuous values remain
- Every retained (non-missing) binary assignment has a recorded posterior probability >= 0.85 in trait_probabilities.csv
- Row counts of binary_traits.csv and trait_probabilities.csv match the number of isolates in the input trait table
- Every continuous (numeric) trait column in the input is represented in the binary trait table
- Discarded assignments appear as missing values rather than forced 0/1 labels

## Limitations

- Isolates classified with less than 85% predicted posterior probability are by default ignored, so highly uncertain traits reduce sample size and statistical power
- Scoary's inability to pre-process numeric traits motivated this binarization step; without it, continuous traits cannot enter the binary association analysis
- The 85% cutoff is a default, not a data-driven optimum; traits with poorly separated bimodal distributions may discard many isolates

## Evidence

- [methods] In Scoary2's automatic binarization, isolates that are classified with less than 85% predicted posterior probability are by default ignored from further analysis.: [evidence span withheld — anchor status: not_found]
- [readme] Scoary2 associates orthogenes generated with OrthoFinder or Roary to traits, so binarized phenotype tables feed directly into the gene-trait association.: "Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder]
or [Roary][roary] to traits."
