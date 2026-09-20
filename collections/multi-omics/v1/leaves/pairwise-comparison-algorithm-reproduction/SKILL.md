---
name: pairwise-comparison-algorithm-reproduction
description: Use when validating or reproducing a runtime/complexity benchmark between
  a reimplemented algorithm and its predecessor — specifically when timing Scoary2's
  pairwise-comparison code against original Scoary on synthetic gene-by-genome presence/absence
  matrices of varying dimensions.
when_to_use_negative:
- The goal is running an actual mGWAS association analysis on real pan-genome and
  phenotype data with Scoary2's CLI/data-exploration app rather than benchmarking
  algorithm runtimes
- The analysis requires inferring orthogroups from raw protein sequences first (use
  OrthoFinder or Roary upstream); this skill starts from already-binary presence/absence
  matrices
- Only a single runtime measurement on one dataset is needed — this skill is specifically
  a comparative benchmarking protocol across a 5–100 × 5–100 gene/genome grid
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Scoary2
  role: Reimplemented pairwise-comparison algorithm invoked as a Python library per
    the Scoary2 wiki to record wall-clock runtimes on synthetic configurations
  repo: github.com/MrTomRod/scoary-2
- name: Scoary
  role: Original pairwise-comparison algorithm run five times per configuration as
    the baseline whose runtime is compared against Scoary2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:e4720ee6d6548925c066f574838c944544d1ee9f450e7f685be52d0998c6135b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Scoary
  - Scoary2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# pairwise-comparison-algorithm-reproduction

## Summary

Reproduces the Scoary2 paper's benchmark showing its reimplemented pairwise-comparison algorithm is ~30x faster than original Scoary, with runtime scaling proportionally to n_genes × n_genomes, by timing both implementations on randomly generated binary gene presence/absence matrices.

## When to use

Use when validating or reproducing a runtime/complexity benchmark between a reimplemented algorithm and its predecessor — specifically when timing Scoary2's pairwise-comparison code against original Scoary on synthetic gene-by-genome presence/absence matrices of varying dimensions.

## When NOT to use

- The goal is running an actual mGWAS association analysis on real pan-genome and phenotype data with Scoary2's CLI/data-exploration app rather than benchmarking algorithm runtimes
- The analysis requires inferring orthogroups from raw protein sequences first (use OrthoFinder or Roary upstream); this skill starts from already-binary presence/absence matrices
- Only a single runtime measurement on one dataset is needed — this skill is specifically a comparative benchmarking protocol across a 5–100 × 5–100 gene/genome grid

## Inputs

- Randomly generated binary gene presence/absence matrices (gene counts 5–100, genome counts 5–100)
- Random binary trait vectors paired with each matrix configuration
- Original Scoary pairwise-comparison algorithm
- Scoary2 pairwise-comparison algorithm used as a Python library (per the Scoary2 wiki)

## Outputs

- benchmark_results.csv containing the timing table, per-configuration speedup ratios (Scoary runtime / Scoary2 runtime), and the symbolic-regression-fitted runtime formula
- Reproduction of Additional file 1: Fig. S1 (runtime scaling across the 5–100 × 5–100 configuration grid)

## How to apply

Generate random binary gene presence/absence matrices spanning gene counts from 5 to 100 and genome counts from 5 to 100, each paired with a random binary trait vector. Run the original Scoary pairwise-comparison algorithm five times per configuration and record wall-clock runtimes; do the same for the reimplemented Scoary2 pairwise-comparison algorithm invoked as a Python library per the Scoary2 wiki. Compute per-configuration speedup ratios (Scoary runtime / Scoary2 runtime) and check the median is approximately 30x. Apply symbolic regression to the collected runtime measurements to recover a parsimonious closed-form runtime formula, which should confirm runtime proportional to n_genes × n_genomes, reproducing Additional file 1: Fig. S1. Write the timing table, speedup ratios, and fitted formula to benchmark_results.csv.

## Related tools

- **Scoary2** (Reimplemented pairwise-comparison algorithm invoked as a Python library per the Scoary2 wiki to record wall-clock runtimes on synthetic configurations) — github.com/MrTomRod/scoary-2
- **Scoary** (Original pairwise-comparison algorithm run five times per configuration as the baseline whose runtime is compared against Scoary2)

## Evaluation signals

- Median speedup of Scoary2 over original Scoary across configurations is approximately 30x
- Symbolic-regression formula parsimoniously fits measured runtimes across the [5..100] × [5..100] configuration grid as in Additional file 1: Fig. S1
- Runtimes scale with n_genes × n_genomes for both algorithms
- Each configuration was timed five times per algorithm with wall-clock runtimes recorded
- All timings, speedup ratios, and the fitted formula are written to benchmark_results.csv

## Limitations

- Speedup (~30x) and the n_genes × n_genomes scaling were established on randomly generated datasets, not real pan-genomes
- The wiki-based Python-library usage of Scoary2 requires reading the 'Usage as Python library' wiki page; the public README itself only links to it
- The article's discussion section notes performance was increased to enable datasets containing tens of thousands of traits, so benchmarking only up to 100 genes/genomes may not extrapolate to the largest supported scales

## Evidence

- [abstract] we introduce Scoary2, an ultra-fast microbial genome-wide association studies (mGWAS) software: "we introduce Scoary2, an ultra-fast microbial genome-wide association studies (mGWAS) software"
- [intro] Three limitations prevent Scoary from such high-throughput analyses: "Three limitations prevent Scoary from such high-throughput analyses: performance constraints, the inability to pre-process numeric traits, and a lack of post-GWAS methods optimized for the"
- [discussion] We expanded Scoary's applicability to datasets containing tens of thousands of traits by significantly increasing the performance of the algorithm: "We expanded Scoary’s applicability to datasets containing tens of thousands of traits by significantly increasing the performance of the algorithm."
- [readme] Usage as Python library (Scoary2 wiki link): "- [Usage as Python library](https://github.com/MrTomRod/scoary-2/wiki/Usage-as-Python-library)"
- [readme] Scoary2 associates orthogenes generated using OrthoFinder or Roary to traits: "Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder]
or [Roary][roary] to traits."
