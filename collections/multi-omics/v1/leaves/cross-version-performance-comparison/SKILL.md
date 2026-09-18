---
name: cross-version-performance-comparison
description: Use when a paper claims a rewritten tool is faster than its predecessor
  (e.g. Scoary2 vs Scoary) and you must reproduce the reported speedup (~30x) and
  the asymptotic runtime scaling on synthetic data.
when_to_use_negative:
- The goal is to benchmark a tool with no comparable predecessor algorithm — the speedup-ratio
  design requires an original-version baseline on identical inputs
- The analysis concerns association results (gene–trait p values, e.g. carnitine metabolism
  findings) rather than algorithm runtime; use Scoary2's association workflow instead
- Only a single dataset size is available — the speedup ratio and symbolic-regression
  complexity fit require the [5..100] x [5..100] configuration grid
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Scoary2
  role: Reimplemented pairwise-comparison algorithm timed as the faster candidate
    (used as a Python library per the Scoary2 wiki)
  repo: github.com/MrTomRod/scoary-2
- name: Scoary
  role: Original pairwise-comparison algorithm timed as the slower baseline for computing
    speedup ratios
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:9bc193c9fb17b0748df23fe69da1d118b6597a6f12246ed21689dd33682ebbdd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MrTomRod/scoary-2
  tools:
  - Scoary
  - Scoary2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cross-version-performance-comparison

## Summary

A benchmarking skill for quantifying the runtime improvement of a reimplemented algorithm (Scoary2's pairwise comparison) against its original version (Scoary), across a grid of dataset sizes, and recovering the runtime's closed-form complexity (here proportional to n_genes × n_genomes) via symbolic regression.

## When to use

Use when a paper claims a rewritten tool is faster than its predecessor (e.g. Scoary2 vs Scoary) and you must reproduce the reported speedup (~30x) and the asymptotic runtime scaling on synthetic data.

## When NOT to use

- The goal is to benchmark a tool with no comparable predecessor algorithm — the speedup-ratio design requires an original-version baseline on identical inputs
- The analysis concerns association results (gene–trait p values, e.g. carnitine metabolism findings) rather than algorithm runtime; use Scoary2's association workflow instead
- Only a single dataset size is available — the speedup ratio and symbolic-regression complexity fit require the [5..100] x [5..100] configuration grid

## Inputs

- Random binary gene presence/absence matrices (gene counts 5–100 × genome counts 5–100)
- Random binary trait vectors paired with each matrix configuration
- Original Scoary pairwise-comparison algorithm
- Scoary2 pairwise-comparison module used as a Python library

## Outputs

- benchmark_results.csv containing the timing table, per-configuration speedup ratios, and the symbolic-regression runtime formula
- Median speedup value (~30x) and fitted runtime complexity (proportional to n_genes × n_genomes)
- Reproduction of Additional file 1: Fig. S1 (runtime formula fit across the [5..100] x [5..100] grid)

## How to apply

Generate random binary gene presence/absence matrices spanning gene counts 5–100 and genome counts 5–100, each paired with a random binary trait vector. Run the original tool's pairwise-comparison algorithm (Scoary) five times per configuration and record wall-clock runtimes, then run the reimplemented algorithm (Scoary2, used as a Python library per its wiki) five times on identical configurations. Compute per-configuration speedup ratios (Scoary runtime / Scoary2 runtime) and confirm the median ratio is approximately 30x. Finally, apply symbolic regression to the collected runtime measurements to recover a parsimonious closed-form runtime formula, reproducing Additional file 1: Fig. S1, and write the timing table, ratios, and fitted formula to benchmark_results.csv.

## Related tools

- **Scoary2** (Reimplemented pairwise-comparison algorithm timed as the faster candidate (used as a Python library per the Scoary2 wiki)) — github.com/MrTomRod/scoary-2
- **Scoary** (Original pairwise-comparison algorithm timed as the slower baseline for computing speedup ratios)

## Evaluation signals

- Median speedup of Scoary2 over original Scoary across the configuration grid is approximately 30x
- Symbolic-regression formula parsimoniously fits measured runtimes across the [5..100] x [5..100] gene-by-genome grid, matching Additional file 1: Fig. S1
- Each configuration is timed 5 times with both algorithms, giving 5 replicate wall-clock runtimes per tool per configuration
- Fitted runtime formula scales proportionally to n_genes × n_genomes
- benchmark_results.csv is produced containing the timing table, speedup ratios, and fitted formula

## Limitations

- Speedup measurements depend on randomly generated synthetic datasets over gene counts 5–100 and genome counts 5–100; the ~30x ratio and fitted complexity may not extrapolate to very large pan-genomes or real-world data characteristics
- Scoary2's performance advantage addresses only Scoary's performance constraint; Scoary also had limitations in numeric-trait pre-processing and post-GWAS methods that this benchmark does not evaluate
- Wall-clock runtimes are sensitive to hardware and implementation details; both algorithms must be run on identical configurations for the ratio to be meaningful

## Evidence

- [abstract] Scoary2 is ultra-fast mGWAS software: "we introduce Scoary2, an ultra-fast microbial genome-wide association studies (mGWAS) software"
- [intro] Scoary's performance limitation motivates the comparison: "Three limitations prevent Scoary from such high-throughput analyses: performance constraints, the inability to pre-process numeric traits, and a lack of post-GWAS methods"
- [discussion] Performance increase is the headline improvement over original Scoary: "We expanded Scoary’s applicability to datasets containing tens of thousands of traits by significantly increasing the performance of the algorithm."
- [readme] Scoary2 usage pattern for library-based benchmarking: "Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder] or [Roary][roary] to traits. It reports a list of genes sorted by strength of association per trait."
