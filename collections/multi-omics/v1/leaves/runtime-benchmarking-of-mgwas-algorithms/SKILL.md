---
name: runtime-benchmarking-of-mgwas-algorithms
description: Use when comparing the computational performance of two GWAS/association
  algorithms whose runtimes are expected to scale with the number of genes and genomes,
  and you need a reproducible speedup ratio plus a closed-form runtime formula.
when_to_use_negative:
- The task is assessing association accuracy (e.g. discovering genes like the two
  carnitine-metabolism genes in Propionibacterium reichii yogurt strains), not algorithm
  runtime — use the standard Scoary2 association workflow instead
- Input data is a real pan-genome from tools like OrthoFinder or Roary and the goal
  is biological interpretation with Scoary2's data exploration app rather than a controlled
  performance comparison
- The two implementations differ in output semantics or statistical treatment (e.g.
  Scoary2's wiki-described p values), so a runtime ratio alone would not be a fair
  benchmark
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Scoary2
  role: Reimplemented ultra-fast mGWAS pairwise-comparison algorithm whose runtime
    is benchmarked; invoked as a Python library per the Scoary2 wiki
  repo: https://github.com/MrTomRod/scoary-2
- name: Scoary
  role: Original pairwise-comparison algorithm serving as the baseline whose wall-clock
    runtimes establish the denominator of the ~30x speedup ratio
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:a878b11721361702d1fa23c91e37df26154e32379f0852164a11db5e1f55d91e
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

# runtime-benchmarking-of-mgwas-algorithms

## Summary

Benchmarking skill for quantifying the wall-clock speedup and time complexity of a reimplemented mGWAS algorithm (Scoary2's pairwise-comparison routine) against its predecessor (original Scoary) on randomized synthetic datasets, reproducing Additional file 1: Fig. S1.

## When to use

Use when comparing the computational performance of two GWAS/association algorithms whose runtimes are expected to scale with the number of genes and genomes, and you need a reproducible speedup ratio plus a closed-form runtime formula.

## When NOT to use

- The task is assessing association accuracy (e.g. discovering genes like the two carnitine-metabolism genes in Propionibacterium reichii yogurt strains), not algorithm runtime — use the standard Scoary2 association workflow instead
- Input data is a real pan-genome from tools like OrthoFinder or Roary and the goal is biological interpretation with Scoary2's data exploration app rather than a controlled performance comparison
- The two implementations differ in output semantics or statistical treatment (e.g. Scoary2's wiki-described p values), so a runtime ratio alone would not be a fair benchmark

## Inputs

- Random binary gene presence/absence matrices with gene counts from 5 to 100 and genome counts from 5 to 100
- Random binary trait vectors paired with each generated dataset configuration
- Original Scoary pairwise-comparison algorithm executable code
- Scoary2 pairwise-comparison algorithm, used as a Python library per the Scoary2 wiki

## Outputs

- Per-configuration wall-clock runtimes for Scoary (5 replicates) and Scoary2 (5 replicates)
- Per-configuration speedup ratios (Scoary runtime / Scoary2 runtime)
- Symbolic-regression closed-form runtime formula reproducing Additional file 1: Fig. S1
- benchmark_results.csv containing the timing table, speedup ratios, and fitted formula

## How to apply

Generate random binary gene presence/absence matrices spanning gene counts from 5 to 100 and genome counts from 5 to 100, each paired with a random binary trait vector, so both algorithms see identical inputs. Run the original Scoary pairwise-comparison algorithm five times per configuration and record wall-clock runtimes, then run Scoary2's reimplemented pairwise comparison (imported and used as a Python library per the Scoary2 wiki) five times on the same configurations. Compute per-configuration speedup ratios (Scoary runtime / Scoary2 runtime) and check that the median ratio is approximately 30x. Finally, apply symbolic regression to the collected runtime measurements to recover a parsimonious closed-form runtime formula proportional to n_genes x n_genomes, and write the timing table, speedup ratios, and fitted formula to benchmark_results.csv.

## Related tools

- **Scoary2** (Reimplemented ultra-fast mGWAS pairwise-comparison algorithm whose runtime is benchmarked; invoked as a Python library per the Scoary2 wiki) — https://github.com/MrTomRod/scoary-2
- **Scoary** (Original pairwise-comparison algorithm serving as the baseline whose wall-clock runtimes establish the denominator of the ~30x speedup ratio)

## Evaluation signals

- Median speedup ratio (Scoary runtime / Scoary2 runtime) across the [5..100] x [5..100] configuration grid is approximately 30x
- Symbolic-regression formula parsimoniously fits the measured runtimes across the full configuration grid, reproducing Additional file 1: Fig. S1
- Each of the 5 vs 5 through 100 vs 100 gene/genome configurations has 5 timing replicates for both algorithms on identical random inputs
- benchmark_results.csv is produced and contains the timing table, speedup ratios, and the fitted runtime formula
- Fitted runtime formula is consistent with scaling proportional to n_genes x n_genomes

## Limitations

- Speedup and complexity results derive from randomly generated datasets of up to 100 genes and 100 genomes; extrapolation to very large real pan-genomes should be validated separately
- Wall-clock runtimes are hardware- and environment-dependent, so the absolute timing values in benchmark_results.csv are machine-specific even if the ~30x ratio is reproducible
- Scoary2's performance gains address only Scoary's performance constraints; Scoary's other limitations (inability to pre-process numeric traits, lack of post-GWAS methods) are outside the scope of this benchmark

## Evidence

- [abstract] we introduce Scoary2, an ultra-fast microbial genome-wide association studies (mGWAS) software: "we introduce Scoary2, an ultra-fast microbial genome-wide association studies (mGWAS) software"
- [intro] Three limitations prevent Scoary from such high-throughput analyses: performance constraints, the inability to pre-process numeric traits, and a lack of post-GWAS methods: "Three limitations prevent Scoary from such high-throughput analyses: performance constraints, the inability to pre-process numeric traits, and a lack of post-GWAS methods"
- [discussion] We expanded Scoary's applicability to datasets containing tens of thousands of traits by significantly increasing the performance of the algorithm.: "We expanded Scoary’s applicability to datasets containing tens of thousands of traits by significantly increasing the performance of the algorithm."
- [readme] Usage as Python library (https://github.com/MrTomRod/scoary-2/wiki/Usage-as-Python-library): "- [Usage as Python library](https://github.com/MrTomRod/scoary-2/wiki/Usage-as-Python-library)"
