---
name: computational-performance-verification
description: Use when a paper claims a quantitative speedup (e.g. '59x faster') and
  you need to independently verify runtimes by re-executing the tool on the deposited
  benchmark data with the paper's parameters.
when_to_use_negative:
- The paper reports accuracy or association results rather than runtime claims — use
  a results-reproduction workflow instead
- The tool's benchmark datasets or exact parameter settings are not available, so
  an equivalent-settings comparison cannot be run
- The question is about statistical correctness of mGWAS calls (p-values, enriched
  genes) rather than computational performance
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Scoary2
  role: 'Tool under test: ultra-fast mGWAS software whose runtime on the yogurt dataset
    is being benchmarked'
  repo: https://github.com/MrTomRod/scoary-2
- name: Scoary
  role: 'Baseline tool: original Scoary run with equivalent settings to establish
    the comparison runtime'
- name: OrthoFinder
  role: Upstream generator of orthogroups / hierarchical orthogroups used as the Scoary2
    pan-genome input
  repo: https://github.com/davidemms/OrthoFinder/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:35be0c56b4c35e03794cdd6683c5fb4f31ce351ccc71499cc50918138fd13c8a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MrTomRod/scoary-2
  tools:
  - OrthoFinder
  - Scoary
  - Scoary2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# computational-performance-verification

## Summary

Reproduces and validates published runtime benchmarks by running a tool (here, Scoary2 vs original Scoary) under equivalent settings on the paper's own datasets and comparing measured wall-clock times against reported figures.

## When to use

Use when a paper claims a quantitative speedup (e.g. '59x faster') and you need to independently verify runtimes by re-executing the tool on the deposited benchmark data with the paper's parameters.

## When NOT to use

- The paper reports accuracy or association results rather than runtime claims — use a results-reproduction workflow instead
- The tool's benchmark datasets or exact parameter settings are not available, so an equivalent-settings comparison cannot be run
- The question is about statistical correctness of mGWAS calls (p-values, enriched genes) rather than computational performance

## Inputs

- Scoary2 source code or Docker image (github.com/MrTomRod/scoary-2)
- Original Scoary command-line tool
- Full yogurt dataset: 3889 traits, 182 isolates, 10358 hierarchical orthogroups (e.g. from OrthoFinder/Roary output)
- Reduced dataset: 44 isolates, 1466 orthogroups
- Benchmark phenotype subset of 100 randomly selected traits

## Outputs

- benchmark_runtimes.csv with measured wall-clock times per tool per dataset subset
- Comparison table of measured vs reported runtimes (23 s vs 22 min full; 43 s vs 16 min reduced)
- Speedup ratio estimate (reported as ~59x on the 100-trait subset)

## How to apply

Obtain the tool source and the deposited benchmark datasets — for Scoary2, the yogurt dataset from github.com/MrTomRod/scoary-2 (full: 3889 traits, 182 isolates, 10,358 hierarchical orthogroups; reduced: 44 isolates, 1466 orthogroups) plus the original Scoary command-line tool. Prepare the phenotype subset exactly as the paper did (100 randomly selected traits). Run both tools with equivalent settings on the same subsets, recording wall-clock time for each run. Tabulate measured runtimes (e.g. benchmark_runtimes.csv) and compare against the reported figures: Scoary2 ~23 s vs Scoary ~22 min on the full dataset, and ~43 s (Scoary2) vs 16 min (Scoary) on the reduced dataset. Accept the benchmark only if the measurements approximate the reported values within reasonable variance; otherwise flag hardware or parameter mismatches before concluding the claim is wrong.

## Related tools

- **Scoary2** (Tool under test: ultra-fast mGWAS software whose runtime on the yogurt dataset is being benchmarked) — https://github.com/MrTomRod/scoary-2
- **Scoary** (Baseline tool: original Scoary run with equivalent settings to establish the comparison runtime)
- **OrthoFinder** (Upstream generator of orthogroups / hierarchical orthogroups used as the Scoary2 pan-genome input) — https://github.com/davidemms/OrthoFinder/

## Evaluation signals

- Scoary2 completes the full-dataset (3889 traits, 182 isolates) association run in approximately 23 s versus Scoary's approximately 22 min
- Scoary2 completes the reduced-dataset (44 isolates, 1466 orthogroups) run in approximately 43 s, and Scoary in approximately 16 min
- Measured speedup on the 100-trait benchmark subset is on the order of the reported 59x
- Both tools were run with equivalent settings on identical input subsets, so runtimes are directly comparable
- All wall-clock measurements are recorded in benchmark_runtimes.csv and tabulated against the paper's reported figures

## Limitations

- Absolute runtimes depend on the benchmarking hardware; only the relative speedup between tools is robust
- Equivalent parameter settings between Scoary2 and original Scoary must be confirmed, otherwise the comparison is invalid
- Runtime on the reduced dataset (16 min for Scoary) is long enough that measurement noise and system load can obscure the reported figures

## Evidence

- [results] Scoary2 processes the full yogurt dataset (3889 traits, 182 isolates, 10,358 hierarchical orthogroups) in only 16 minutes.: [evidence span withheld — anchor status: not_found]
- [discussion] We expanded Scoary's applicability ... by significantly increasing the performance of the algorithm.: "We expanded Scoary’s applicability to datasets containing tens of thousands of traits by significantly increasing the performance of the algorithm."
- [abstract] we introduce Scoary2, an ultra-fast microbial genome-wide association studies (mGWAS) software: "we introduce Scoary2, an ultra-fast microbial genome-wide association studies (mGWAS) software"
- [readme] Scoary2 associates orthogenes ... to traits and reports a list of genes sorted by strength of association per trait.: "Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder]
or [Roary][roary] to traits. It reports a list of genes sorted by strength of
association per trait."
