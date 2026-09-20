---
name: gaussian-mixture-posterior-thresholding
description: Use when continuous numeric trait columns (e.g., yogurt metabolite concentrations)
  must be converted to binary case/control labels for Scoary2 association testing
  and uncertain isolates need a principled exclusion rule.
when_to_use_negative:
- Trait columns are already binary or categorical — posterior-probability thresholding
  would needlessly discard valid labels.
- The analysis needs traits kept as continuous quantitative values (e.g., QTL-style
  or regression-based association) rather than the case/control contrast that binarized
  Scoary2 traits provide.
- Input is a gene/orthogroup presence-absence matrix rather than a phenotype table;
  this skill applies only to phenotypic trait columns.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3391
tools:
- name: Scoary2
  role: Implements the automatic binarization pipeline (k-means hard labels plus two-component
    Gaussian mixture posterior probabilities with the 85% retention cutoff) that turns
    continuous trait columns into binary mGWAS inputs
  repo: https://github.com/MrTomRod/scoary-2
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:4d860c26cee83e4ba370ead016cc96b2945cbd2f555e945042e8cb83715b74c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MrTomRod/scoary-2
  tools:
  - Scoary2
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3391
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# gaussian-mixture-posterior-thresholding

## Summary

Scoary2's automatic binarization converts continuous phenotype columns (e.g., metabolite traits from yogurt omics) into binary mGWAS labels by fitting a two-component Gaussian mixture model and, by default, ignoring isolates whose assigned class has less than 85% predicted posterior probability.

## When to use

Use when continuous numeric trait columns (e.g., yogurt metabolite concentrations) must be converted to binary case/control labels for Scoary2 association testing and uncertain isolates need a principled exclusion rule.

## When NOT to use

- Trait columns are already binary or categorical — posterior-probability thresholding would needlessly discard valid labels.
- The analysis needs traits kept as continuous quantitative values (e.g., QTL-style or regression-based association) rather than the case/control contrast that binarized Scoary2 traits provide.
- Input is a gene/orthogroup presence-absence matrix rather than a phenotype table; this skill applies only to phenotypic trait columns.

## Inputs

- Continuous (numeric) trait columns from an isolate-by-phenotype table (e.g., metabolite abundances from the yogurt multi-omics dataset)
- Isolate identifiers that align trait rows with the pan-genome presence/absence matrix

## Outputs

- binary_traits.csv: binary/missing trait table keyed by isolate ID, one column per binarized continuous trait
- trait_probabilities.csv: per-isolate posterior probabilities for each binarized trait
- Missing-value annotations for isolates whose posterior probability falls below 0.85

## How to apply

For each continuous (numeric) trait column in the isolate-by-trait table, first fit a k-means model with k=2 to obtain hard binary cluster labels, then fit a two-component Gaussian mixture model on the same values and compute each isolate's posterior probability for its assigned class. Apply Scoary2's retention rule: keep the binary assignment only when the posterior probability is at least 0.85; isolates below the cutoff are marked missing/discarded by default rather than forced into a class. Assemble the retained calls into a binary trait table keyed by isolate ID and write the per-isolate posterior probabilities to a companion table (e.g., binary_traits.csv and trait_probabilities.csv). The rationale is that numeric trait pre-processing was a stated limitation of the original Scoary, and the 85% posterior cutoff ensures only confident case/control calls feed the association test on the Propionibacterium reichii yogurt pan-genome.

## Related tools

- **Scoary2** (Implements the automatic binarization pipeline (k-means hard labels plus two-component Gaussian mixture posterior probabilities with the 85% retention cutoff) that turns continuous trait columns into binary mGWAS inputs) — https://github.com/MrTomRod/scoary-2

## Examples

```
from sklearn.mixture import GaussianMixture; p = GaussianMixture(n_components=2).fit(trait[:, None]).predict_proba(trait[:, None]).max(axis=1)  # keep p >= 0.85
```

## Evaluation signals

- Each trait column in binary_traits.csv contains only binary or missing values — no continuous values remain.
- Every retained binary assignment has a recorded posterior probability >= 0.85 in trait_probabilities.csv.
- Row counts of binary_traits.csv and trait_probabilities.csv match the number of isolates in the input dataset.
- Discarded isolates appear as missing values, never as forced 0/1 labels.
- k-means (k=2) hard labels agree with the GMM-assigned class for all retained isolates.

## Limitations

- By default, isolates classified with less than 85% predicted posterior probability are ignored from further analysis, which shrinks the effective sample size per trait.
- Isolates lying between the two mixture modes are discarded under the default cutoff, so traits with many intermediate phenotypes may lose a large fraction of isolates.

## Evidence

- [other] Scoary2's default retention rule keeps a GMM-based binary assignment only at >=85% posterior probability; lower-confidence isolates are marked missing/discarded.: [evidence span withheld — anchor status: not_found]
- [intro] Inability to pre-process numeric traits was a stated limitation of the original Scoary that motivated Scoary2.: "Three limitations prevent Scoary from such high-throughput analyses: performance constraints, the inability to pre-process numeric traits, and a lack of post-GWAS methods optimized for the"
- [abstract] The continuous traits being binarized are metabolome measurements of yogurts produced with different Propionibacterium reichii strains.: "we explore the metabolome of yogurts, each produced with a different Propionibacterium reichii strain and discover two genes affecting carnitine metabolism"
- [readme] Scoary2 associates orthogenes with traits, so phenotype columns must be in association-ready binary form.: "Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder] or [Roary][roary] to traits. It reports a list of genes sorted by strength of association per trait."
