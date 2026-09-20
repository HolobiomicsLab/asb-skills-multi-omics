---
name: symbolic-regression-of-runtime-scaling
description: Use when re-implementation of an association algorithm is claimed to
  be faster than an original, and you must quantify the speedup (e.g. ~30x for Scoary2
  vs Scoary) and recover the runtime scaling law empirically.
when_to_use_negative:
- The goal is gene-trait association results rather than algorithm performance characterization
  — use Scoary2's normal association mode on real pan-genomes instead
- The datasets are real pan-genomes with phylogenetic structure (e.g. the Agroscope
  strain collection) rather than randomized binary matrices, since the synthetic grid
  is required to isolate runtime scaling
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Scoary2
  role: Reimplemented pairwise-comparison algorithm benchmarked for wall-clock runtime
    (used as a Python library per the Scoary2 wiki)
  repo: github.com/MrTomRod/scoary-2
- name: Scoary
  role: Original pairwise-comparison algorithm whose wall-clock runtime provides the
    baseline for speedup ratios
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:bed3a04d974beee1204b0349ce2b628170a6a696ddfe50f47adde7c01c961a3e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Scoary
  - Scoary2
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# symbolic-regression-of-runtime-scaling

## Summary

Benchmarks Scoary vs. Scoary2 pairwise-comparison runtimes across a grid of randomly generated gene/genome configurations, computes speedup ratios, and fits a parsimonious closed-form runtime formula (runtime ∝ n_genes × n_genomes) via symbolic regression, as reported in Additional file 1: Fig. S1 of the Scoary2 paper.

## When to use

Use when re-implementation of an association algorithm is claimed to be faster than an original, and you must quantify the speedup (e.g. ~30x for Scoary2 vs Scoary) and recover the runtime scaling law empirically.

## When NOT to use

- The goal is gene-trait association results rather than algorithm performance characterization — use Scoary2's normal association mode on real pan-genomes instead
- The datasets are real pan-genomes with phylogenetic structure (e.g. the Agroscope strain collection) rather than randomized binary matrices, since the synthetic grid is required to isolate runtime scaling

## Inputs

- Random binary gene presence/absence matrices with gene counts 5-100 and genome counts 5-100
- Random binary trait vectors paired with each matrix configuration
- Original Scoary pairwise-comparison implementation
- Scoary2 pairwise-comparison algorithm used as a Python library

## Outputs

- benchmark_results.csv containing per-configuration wall-clock runtimes, speedup ratios, and the fitted formula
- Median Scoary2 speedup over original Scoary (expected ~30x)
- Parsimonious closed-form runtime scaling law (runtime ∝ n_genes × n_genomes)

## How to apply

Generate random binary gene presence/absence matrices spanning gene counts from 5 to 100 and genome counts from 5 to 100, each paired with a random binary trait vector. Run the original Scoary pairwise-comparison algorithm and the reimplemented Scoary2 algorithm (used as a Python library per the Scoary2 wiki) five times on every configuration, recording wall-clock runtimes. Compute per-configuration speedup ratios (Scoary runtime / Scoary2 runtime) and check that the median ratio is approximately 30x. Apply symbolic regression to the collected runtime measurements to recover a parsimonious closed-form formula consistent with runtime scaling linearly in n_genes × n_genomes, reproducing Additional file 1: Fig. S1. Write the timing table, speedup ratios, and fitted formula to benchmark_results.csv.

## Related tools

- **Scoary2** (Reimplemented pairwise-comparison algorithm benchmarked for wall-clock runtime (used as a Python library per the Scoary2 wiki)) — github.com/MrTomRod/scoary-2
- **Scoary** (Original pairwise-comparison algorithm whose wall-clock runtime provides the baseline for speedup ratios)

## Evaluation signals

- Median Scoary2 speedup over original Scoary is approximately 30x
- Symbolic-regression formula parsimoniously fits measured runtimes across the [5..100] x [5..100] gene/genome configuration grid as shown in Additional file 1: Fig. S1
- Each configuration is run five times for both algorithms and wall-clock runtimes are recorded
- benchmark_results.csv contains the timing table, speedup ratios, and fitted formula

## Limitations

- Scoary's original performance constraints motivated the rewrite: performance limitations prevent high-throughput analyses with large numbers of microbes or large phenotypic datasets
- Speedup is quantified on randomly generated synthetic datasets over a 5-100 x 5-100 grid; behavior outside this range is not established by the benchmark

## Evidence

- [abstract] we introduce Scoary2, an ultra-fast microbial genome-wide association studies (mGWAS) software: "we introduce Scoary2, an ultra-fast microbial genome-wide association studies (mGWAS) software"
- [intro] Three limitations prevent Scoary from such high-throughput analyses: performance constraints: "Three limitations prevent Scoary from such high-throughput analyses: performance constraints, the inability to pre-process numeric traits, and a lack of post-GWAS methods"
- [discussion] We expanded Scoary's applicability to datasets containing tens of thousands of traits by significantly increasing the performance of the algorithm.: "We expanded Scoary’s applicability to datasets containing tens of thousands of traits by significantly increasing the performance of the algorithm."
- [readme] Scoary2 associates orthogenes (e.g. generated using OrthoFinder or Roary) to traits: "Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder]
or [Roary][roary] to traits."
- [readme] Usage as Python library: "- [Usage as Python library](https://github.com/MrTomRod/scoary-2/wiki/Usage-as-Python-library)"
