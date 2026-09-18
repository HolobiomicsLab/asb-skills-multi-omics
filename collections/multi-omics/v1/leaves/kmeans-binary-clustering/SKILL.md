---
name: kmeans-binary-clustering
description: Use when a phenotypic/omics trait table contains continuous numeric measurements
  (e.g. yogurt metabolome traits) that Scoary2 requires as binary (0/1) presence/absence
  labels before mGWAS association.
when_to_use_negative:
- Trait columns are already binary/categorical — k-means and GMM binarization would
  distort them and is unnecessary
- 'You cannot tolerate discarding isolates: with Scoary2''s default rule, isolates
  classified with <85% posterior probability are ignored from further analysis, shrinking
  the effective sample size'
- The trait distribution is clearly unimodal, so a two-component (k=2) split would
  impose an artificial binary division
tools:
- name: Scoary2
  role: Provides the Python library interface and the automatic binarization rule
    (k-means k=2 plus two-component GMM with 85% posterior-probability retention cutoff)
    that converts continuous traits into binary labels for mGWAS
  repo: https://github.com/MrTomRod/scoary-2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:d081bd01a02d3ee6062e97f0fb80d94891d140b0a3f5636b0e6edbde79ce90a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MrTomRod/scoary-2
  tools:
  - Scoary2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# kmeans-binary-clustering

## Summary

Binarizes continuous (numeric) metabolite or phenotype trait columns so they can be fed into Scoary2's microbial GWAS, by fitting a k=2 k-means clustering for hard labels and a two-component Gaussian mixture model (GMM) whose posterior probabilities gate which binary assignments are retained.

## When to use

Use when a phenotypic/omics trait table contains continuous numeric measurements (e.g. yogurt metabolome traits) that Scoary2 requires as binary (0/1) presence/absence labels before mGWAS association.

## When NOT to use

- Trait columns are already binary/categorical — k-means and GMM binarization would distort them and is unnecessary
- You cannot tolerate discarding isolates: with Scoary2's default rule, isolates classified with <85% posterior probability are ignored from further analysis, shrinking the effective sample size
- The trait distribution is clearly unimodal, so a two-component (k=2) split would impose an artificial binary division

## Inputs

- Phenotypic trait table (e.g. yogurt metabolome trait table) with continuous numeric trait columns, keyed by isolate ID
- Isolate ID list matching the pan-genome/orthogroup matrix used by Scoary2

## Outputs

- binary_traits.csv: per-isolate binary (0/1 or missing) trait calls, one column per continuous trait
- trait_probabilities.csv: per-isolate GMM posterior probabilities of the assigned class for each trait

## How to apply

For each continuous trait column, fit a k-means model with k=2 using Scoary2's Python library interface (github.com/MrTomRod/scoary-2) and assign each isolate a hard binary label from cluster membership. In parallel, fit a two-component Gaussian mixture model on the same column and compute the posterior probability of each isolate's assigned class. Apply the Scoary2 retention rule: keep a binary assignment only when the posterior probability is at least 85%; assignments below this cutoff are treated as missing/ignored rather than forced to a label, because Scoary2 by default ignores isolates classified with less than 85% predicted posterior probability. Assemble the retained calls into a binary trait table keyed by isolate ID and write per-isolate posterior probabilities to a separate table (e.g. binary_traits.csv and trait_probabilities.csv). The GMM gate is the rationale: hard k-means labels alone are unreliable near cluster boundaries, so only confident classifications enter the downstream association.

## Related tools

- **Scoary2** (Provides the Python library interface and the automatic binarization rule (k-means k=2 plus two-component GMM with 85% posterior-probability retention cutoff) that converts continuous traits into binary labels for mGWAS) — https://github.com/MrTomRod/scoary-2

## Evaluation signals

- binary_traits.csv contains only binary or missing values in every trait column — no forced labels for low-confidence isolates
- Every retained binary assignment has a recorded posterior probability >= 0.85 in trait_probabilities.csv
- Row counts of both output tables match the number of isolates in the input dataset
- Every continuous trait column from the input is represented in the binary trait table (no silently dropped columns)
- Discarded assignments appear as missing values rather than arbitrary 0/1 labels

## Limitations

- Isolates classified with less than 85% predicted posterior probability are by default ignored from further analysis, so ambiguous isolates near cluster boundaries are lost rather than force-assigned
- The approach assumes a two-class (k=2) structure per trait; it is not suited to multi-class or ordinal phenotypes
- Missing-signal extraction notes that discussion-section limitations were not available in the provided document, so broader caveats beyond the 85% rule are undocumented here

## Evidence

- [methods] In Scoary2's automatic binarization, isolates that are classified with less than 85% predicted posterior probability are by default ignored from further analysis.: [evidence span withheld — anchor status: not_found]
- [readme] Scoary2 associates orthogenes ... to traits: "Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder] or [Roary][roary] to traits."
- [intro] Existing mGWAS tools lack numeric trait pre-processing: "Three limitations prevent Scoary from such high-throughput analyses: performance constraints, the inability to pre-process numeric traits, and a lack of post-GWAS methods optimized for the"
