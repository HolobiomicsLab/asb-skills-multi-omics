---
name: statistical-power-estimation-across-isolate-counts
description: Use when you must determine how many bacterial isolates are needed to
  detect genes of a given effect size from continuously measured (numeric) phenotypes
  with pan-genome mGWAS, i.e. before committing samples to a Scoary2-based association
  study.
when_to_use_negative:
- The trait is already binary (present/absent) — automatic binarization of a numeric
  trait is not needed and the power curve calibrated on discretized continuous phenotypes
  does not apply.
- You need analytic power formulas for quantitative-trait association rather than
  simulation-based power for Scoary2's binarize-then-test workflow.
- The pan-genome was not built from ortholog clustering (e.g. with OrthoFinder or
  Roary), so the presence/absence structure Scoary2 associates against orthogenes
  does not match your data.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0625
tools:
- name: Scoary2
  role: Performs automatic binarization of the simulated numeric phenotype and the
    mGWAS association test that yields ranked gene p-values; invoked via its Python
    library API
  repo: https://github.com/MrTomRod/scoary-2
- name: OrthoFinder
  role: Example tool for generating the orthogene groups that make up the simulated
    presence/absence pan-genome structure
  repo: https://github.com/davidemms/OrthoFinder
- name: Roary
  role: Alternative pan-genome pipeline whose presence/absence output defines the
    orthogene matrix format Scoary2 consumes
  repo: https://sanger-pathogens.github.io/Roary/
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:1aab3c50f49d0a9b7694584b7c5319f7fdb956a186b275b70aabe4a8c0a4a12c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MrTomRod/scoary-2
  tools:
  - OrthoFinder
  - Roary
  - Scoary2
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# statistical-power-estimation-across-isolate-counts

## Summary

Empirically estimates mGWAS statistical power by simulating pan-genome presence/absence matrices with planted causal genes and paired numeric phenotypes across dataset sizes (25–200 isolates) and effect sizes (0.5–3 σ), then measuring the rank of the causal gene after Scoary2's automatic binarization and association testing.

## When to use

Use when you must determine how many bacterial isolates are needed to detect genes of a given effect size from continuously measured (numeric) phenotypes with pan-genome mGWAS, i.e. before committing samples to a Scoary2-based association study.

## When NOT to use

- The trait is already binary (present/absent) — automatic binarization of a numeric trait is not needed and the power curve calibrated on discretized continuous phenotypes does not apply.
- You need analytic power formulas for quantitative-trait association rather than simulation-based power for Scoary2's binarize-then-test workflow.
- The pan-genome was not built from ortholog clustering (e.g. with OrthoFinder or Roary), so the presence/absence structure Scoary2 associates against orthogenes does not match your data.

## Inputs

- Simulated pan-genome presence/absence matrix (25–200 isolates) with a planted causal gene
- Paired numeric (continuously measured) phenotype vector per isolate
- Effect size specification (0.5–3 σ) and dataset size grid for the simulation sweep

## Outputs

- Ranked gene-level association p-values from the Scoary2 mGWAS run per simulated condition
- Rank of the planted causal gene per (isolate count, effect size) condition
- Aggregated causal-gene rank table (fig1_causal_gene_ranks.csv)

## How to apply

Generate simulated presence/absence matrices with paired numeric phenotypes spanning 25–200 genomes, planting a causal gene with effect sizes of 0.5–3 σ, and call Scoary2 through its Python library API. Run Scoary2's automatic binarization on the numeric trait to discretize it, then execute the mGWAS association test to obtain gene-level p-values. Extract the rank of the planted causal gene from the ranked p-value results for each (isolate count, effect size) condition. Aggregate ranks across conditions and check the reproduction criteria: top rank at 3 σ with 50 isolates and top-four rank at 1.5 σ with 75 isolates, each with 90% probability. The key decision point is the 90%-probability rank threshold, which converts simulation output into a concrete sample-size recommendation (e.g. 75 isolates suffice for a 1.5 σ effect in the top four).

## Related tools

- **Scoary2** (Performs automatic binarization of the simulated numeric phenotype and the mGWAS association test that yields ranked gene p-values; invoked via its Python library API) — https://github.com/MrTomRod/scoary-2
- **OrthoFinder** (Example tool for generating the orthogene groups that make up the simulated presence/absence pan-genome structure) — https://github.com/davidemms/OrthoFinder
- **Roary** (Alternative pan-genome pipeline whose presence/absence output defines the orthogene matrix format Scoary2 consumes) — https://sanger-pathogens.github.io/Roary/

## Evaluation signals

- The planted causal gene reaches top rank with 90% probability at 3 σ effect size and 50 isolates.
- The planted causal gene reaches top-four rank with 90% probability at 1.5 σ effect size and 75 isolates.
- The output CSV (fig1_causal_gene_ranks.csv) contains a causal-gene rank value for every simulated (isolate count, effect size) condition in the 25–200 / 0.5–3 σ grid.
- Ranks are extracted from Scoary2's ranked p-value output, not from raw effect sizes, ensuring the power estimate reflects the actual association-test statistic.
- The 1.5 σ / 75-isolate condition can be stated as a usable recommendation: weak-effect (1.5 σ) genes land within the top four ranks 90% of the time with only 75 isolates.

## Limitations

- Power estimates are grounded in simulated pan-genomes and planted effects; real isolate collections with population structure and linkage may shift the rank distribution.
- The benchmark addresses automatic binarization of continuous phenotypes; original Scoary lacked numeric-trait pre-processing, so results should not be extrapolated to tools without a binarization step.
- Validation targets are probability-based (90% rank targets), so a single replicate per condition is insufficient — ranks must be aggregated over simulation replicates.

## Evidence

- [methods] 90%-probability rank targets at 3σ/50 isolates and 1.5σ/75 isolates: [evidence span withheld — anchor status: not_found]
- [results] 75 isolates suffice for weak-effect genes via automatic binarization: [evidence span withheld — anchor status: not_found]
- [readme] Scoary2 output structure supporting rank extraction: "It reports a list of genes sorted by strength of association per trait."
- [abstract] Scoary2 is an mGWAS tool enabling the simulated power analysis: "we introduce Scoary2, an ultra-fast microbial genome-wide association studies (mGWAS) software"
