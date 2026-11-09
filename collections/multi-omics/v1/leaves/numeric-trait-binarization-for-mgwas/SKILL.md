---
name: numeric-trait-binarization-for-mgwas
description: Use when the phenotype to associate with a bacterial pan-genome is measured
  on a continuous scale (e.g. metabolome or other numeric omics values per isolate)
  and a binary gene-present/gene-absent association test is required.
when_to_use_negative:
- The phenotype is already binary (e.g. resistance/susceptibility label) — no automatic
  binarization is needed; run the association directly
- The trait is a human-scale GWAS/QTL problem — the article notes 'Existing human
  GWAS (hGWAS) or QTL methods cannot be directly applied to microbes', but conversely
  Scoary2's binarization targets microbial pan-genome data, not human cohort data
- The task requires modeling the numeric trait quantitatively (linear models, QTL-style
  effect estimation) rather than a presence/absence association
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: Scoary2
  role: Performs the automatic binarization of numeric traits and the subsequent mGWAS
    association test against the pan-genome; usable via CLI, Docker image, or Python
    library API
  repo: https://github.com/MrTomRod/scoary-2
- name: OrthoFinder
  role: Generates the orthogroups/orthogenes presence-absence input that the binarized
    trait is associated against
  repo: https://github.com/davidemms/OrthoFinder
- name: Roary
  role: Alternative pan-genome pipeline to build the presence/absence matrix consumed
    by the association step
  repo: https://sanger-pathogens.github.io/Roary/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:5ca83d5431d14903afafebe285dd943191228a94c575e978e36e15ec2aaf940e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MrTomRod/scoary-2
  tools:
  - OrthoFinder
  - Roary
  - Scoary2
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# Numeric trait binarization for mGWAS

## Summary

Discretizes continuously measured (numeric/omics) phenotypes into binary traits via Scoary2's automatic binarization so they can be subjected to Scoary2's pan-genome mGWAS association testing, with known detection-power behavior as a function of isolate count and effect size.

## When to use

Use when the phenotype to associate with a bacterial pan-genome is measured on a continuous scale (e.g. metabolome or other numeric omics values per isolate) and a binary gene-present/gene-absent association test is required.

## When NOT to use

- The phenotype is already binary (e.g. resistance/susceptibility label) — no automatic binarization is needed; run the association directly
- The trait is a human-scale GWAS/QTL problem — the article notes 'Existing human GWAS (hGWAS) or QTL methods cannot be directly applied to microbes', but conversely Scoary2's binarization targets microbial pan-genome data, not human cohort data
- The task requires modeling the numeric trait quantitatively (linear models, QTL-style effect estimation) rather than a presence/absence association

## Inputs

- Per-isolate numeric (continuous) phenotype measurements, e.g. metabolome/omics trait table keyed by isolate ID
- Pan-genome presence/absence matrix of orthogenes (e.g. from OrthoFinder or Roary), keyed by the same isolate IDs
- Optional simulation parameters when benchmarking: isolate count (e.g. 25-200 genomes) and causal-gene effect size (e.g. 0.5-3 sigma)

## Outputs

- Binarized (case/control) trait column suitable for Scoary2 mGWAS input
- Gene-level association p-values with genes ranked by association strength per trait
- When benchmarking: rank of the planted causal gene per condition (e.g. aggregated to a CSV such as fig1_causal_gene_ranks.csv)

## How to apply

Take the per-isolate numeric phenotype matrix (e.g. continuous omics measurements from the Agroscope yogurt strain collection type of data) and run Scoary2's automatic binarization step to discretize the trait into a binary case/control status compatible with Scoary's presence/absence association logic. Scoary2 exists precisely because the original Scoary lacked 'the inability to pre-process numeric traits', so this pre-processing must happen inside Scoary2 rather than upstream ad hoc thresholding. After binarization, run Scoary2's mGWAS association test against the orthogene presence/absence matrix (e.g. orthogroups from OrthoFinder or Roary) and inspect the gene list 'sorted by strength of association per trait'. Power depends on cohort size and effect size: benchmarks show a causal gene of moderate (1.5 sigma) effect lands in the top four ranked genes with 90% probability at ~75 isolates, while strong (3 sigma) effects are recovered at top rank with as few as ~50 isolates.

## Related tools

- **Scoary2** (Performs the automatic binarization of numeric traits and the subsequent mGWAS association test against the pan-genome; usable via CLI, Docker image, or Python library API) — https://github.com/MrTomRod/scoary-2
- **OrthoFinder** (Generates the orthogroups/orthogenes presence-absence input that the binarized trait is associated against) — https://github.com/davidemms/OrthoFinder
- **Roary** (Alternative pan-genome pipeline to build the presence/absence matrix consumed by the association step) — https://sanger-pathogens.github.io/Roary/

## Evaluation signals

- The binarized trait yields a two-level (present/absent) encoding with no isolate dropped, and the number of isolates matches the input numeric table
- In simulation-based checks, a planted causal gene with a 3 sigma effect reaches top rank at ~50 isolates, and a 1.5 sigma effect reaches top-four rank with 90% probability at ~75 isolates
- The ranked output contains gene-level p-values for every orthogene, sorted by association strength per trait
- When aggregating across conditions, every simulated dataset size x effect-size combination has a recorded causal-gene rank in the output CSV
- Results are explorable in Scoary2's static HTML/JS results app, confirming the output schema is well-formed

## Limitations

- Power is limited for weak effects: a 1.5 sigma causal gene requires roughly 75 isolates to appear in the top four ranks with 90% probability, so very small cohorts (<~50 isolates) may miss moderate effects
- Binarization discards the continuous (magnitude) information in the trait; the association is presence/absence-based rather than a quantitativetrait model
- The document's discussion section is not available, so broader caveats of the automatic binarization procedure could not be extracted from primary text

## Evidence

- [abstract] we introduce Scoary2, an ultra-fast microbial genome-wide association studies (mGWAS) software: "we introduce Scoary2, an ultra-fast microbial genome-wide association studies (mGWAS) software"
- [intro] performance constraints, the inability to pre-process numeric traits: "Three limitations prevent Scoary from such high-throughput analyses: performance constraints, the inability to pre-process numeric traits, and a lack of post-GWAS methods optimized for the"
- [readme] Scoary2 associates orthogenes ... to traits: "Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder] or [Roary][roary] to traits. It reports a list of genes sorted by strength of"
- [abstract] Scoary2 is the first tool to enable the study of large phenotypic datasets using mGWAS: "With its data exploration app and improved performance, Scoary2 is the first tool to enable the study of large phenotypic datasets using mGWAS"
