---
name: pan-genome-presence-absence-simulation
description: Use when you need synthetic pan-genome presence/absence matrices to measure
  how an mGWAS pairwise-comparison algorithm's runtime scales with n_genes × n_genomes,
  or to reproduce Scoary2's reported ~30x speedup over Scoary.
when_to_use_negative:
- The goal is biological association discovery rather than algorithm performance measurement
  — random synthetic matrices carry no real genotype–trait signal and cannot yield
  meaningful candidate genes
- The input is already a real pan-genome presence/absence matrix from tools like OrthoFinder
  or Roary; this skill generates synthetic matrices specifically for runtime benchmarking
- Only a single fixed dataset configuration is of interest — the skill's value comes
  from sweeping the [5..100] × [5..100] grid to expose scaling behavior
tools:
- name: Scoary2
  role: Target implementation whose pairwise-comparison algorithm is benchmarked,
    called as a Python library on each synthetic configuration
  repo: github.com/MrTomRod/scoary-2
- name: Scoary
  role: Original pairwise-comparison baseline whose wall-clock runtimes are recorded
    five times per configuration to compute speedup ratios
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:858ba3243120d1b198753a720df15c827fc1fd42ed3e5851ecc44a327dbcde36
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Scoary
  - Scoary2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# pan-genome-presence-absence-simulation

## Summary

Generate random binary gene presence/absence matrices (pan-genomes) paired with random binary trait vectors to benchmark the pairwise-comparison runtime of Scoary2 against original Scoary, recovering the ~30x speedup and the n_genes × n_genomes scaling law via symbolic regression.

## When to use

Use when you need synthetic pan-genome presence/absence matrices to measure how an mGWAS pairwise-comparison algorithm's runtime scales with n_genes × n_genomes, or to reproduce Scoary2's reported ~30x speedup over Scoary.

## When NOT to use

- The goal is biological association discovery rather than algorithm performance measurement — random synthetic matrices carry no real genotype–trait signal and cannot yield meaningful candidate genes
- The input is already a real pan-genome presence/absence matrix from tools like OrthoFinder or Roary; this skill generates synthetic matrices specifically for runtime benchmarking
- Only a single fixed dataset configuration is of interest — the skill's value comes from sweeping the [5..100] × [5..100] grid to expose scaling behavior

## Inputs

- Randomly generated binary gene presence/absence matrices (gene counts 5–100 × genome counts 5–100)
- Random binary trait vector paired with each matrix configuration
- Original Scoary pairwise-comparison implementation (baseline)
- Scoary2 pairwise-comparison algorithm invoked as a Python library

## Outputs

- benchmark_results.csv containing the timing table, per-configuration speedup ratios, and the fitted runtime formula
- Symbolic-regression-derived closed-form runtime expression (runtime ∝ n_genes × n_genomes) reproducing Additional file 1: Fig. S1
- Median Scoary2-vs-Scoary speedup ratio (~30x)

## How to apply

Generate random binary gene presence/absence matrices spanning gene counts from 5 to 100 and genome counts from 5 to 100, pairing each configuration with a random binary trait vector. For every configuration, run the original Scoary pairwise-comparison algorithm five times and the reimplemented Scoary2 pairwise-comparison algorithm (used as a Python library per the Scoary2 wiki 'Usage as Python library' page) five times, recording wall-clock runtimes for each replicate. Compute per-configuration speedup ratios (Scoary runtime / Scoary2 runtime) and check the median is approximately 30x. Apply symbolic regression to the collected runtime measurements to recover a parsimonious closed-form runtime formula (runtime ∝ n_genes × n_genomes), reproducing Additional file 1: Fig. S1. Write the timing table, speedup ratios, and fitted formula to benchmark_results.csv.

## Related tools

- **Scoary2** (Target implementation whose pairwise-comparison algorithm is benchmarked, called as a Python library on each synthetic configuration) — github.com/MrTomRod/scoary-2
- **Scoary** (Original pairwise-comparison baseline whose wall-clock runtimes are recorded five times per configuration to compute speedup ratios)

## Evaluation signals

- Median speedup (Scoary runtime / Scoary2 runtime) across the [5..100] × [5..100] configuration grid is approximately 30x
- Fitted symbolic-regression formula is parsimonious and consistent with runtime scaling proportional to n_genes × n_genomes across the full grid, as shown in Additional file 1: Fig. S1
- Each configuration was run five times with both algorithms and all wall-clock runtimes are recorded in the timing table
- benchmark_results.csv exists and contains the timing table, speedup ratios, and fitted formula as specified by the workflow

## Limitations

- Synthetic random presence/absence matrices lack the structure of real pan-genomes (e.g. clonal structure, linkage), so runtimes may differ from those on biological data
- The ~30x speedup is a median over the configuration grid and depends on the hardware and environment used for the wall-clock measurements
- The benchmark grid spans only gene and genome counts of 5–100; scaling beyond this range was not validated here

## Evidence

- [methods] Generate random binary gene presence/absence matrices spanning gene counts from 5 to 100 and genome counts from 5 to 100, each paired with a random binary trait vector.: [evidence span withheld — anchor status: not_found]
- [results] Scoary2's reimplemented pairwise comparison algorithm is around 30x faster than original Scoary.: [evidence span withheld — anchor status: not_found]
- [supplementary] Apply symbolic regression to the collected runtime measurements to recover a parsimonious closed-form runtime formula, reproducing Additional file 1: Fig. S1.: [evidence span withheld — anchor status: not_found]
- [readme] Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder] or [Roary][roary] to traits.: "Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder] or [Roary][roary] to traits."
