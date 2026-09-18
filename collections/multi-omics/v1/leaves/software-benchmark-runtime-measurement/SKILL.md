---
name: software-benchmark-runtime-measurement
description: Use when a paper claims a tool is substantially faster than an existing
  one and the runtime claim (e.g. Scoary2 being 59x faster on 100 traits) must be
  reproduced on the deposited benchmark data.
when_to_use_negative:
- The goal is association results themselves (gene-trait hits such as the two carnitine-metabolism
  genes) rather than runtime comparison — a single Scoary2 run suffices
- The pan-genome input has not been clustered into orthogroups by OrthoFinder or Roary;
  runtime benchmarking requires the ready orthogroup/traits matrices
- The evaluation question concerns Scoary2's numeric-trait pre-processing or post-GWAS
  features, which Scoary lacks entirely and thus cannot be compared on equal footing
edam_operation: http://edamontology.org/operation_0326
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0625
tools:
- name: Scoary2
  role: Benchmarked tool whose wall-clock runtime on the yogurt datasets is measured
    and compared against the reported 43 s (reduced) / 23 s (full 100-trait) figures
  repo: https://github.com/MrTomRod/scoary-2
- name: Scoary
  role: Baseline tool run with equivalent parameters on the same subsets to establish
    the ~22 min (full) / ~16 min (reduced) reference runtimes
- name: OrthoFinder
  role: Generator of the hierarchical orthogroups (10358 full / 1466 reduced) that
    form the fixed genotype input held constant across benchmark runs
  repo: https://github.com/davidemms/OrthoFinder/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:69c633bcfff47ac5471fee3b50dbb80ad5e260c1bb42559bdaa79dbbf2453aea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MrTomRod/scoary-2
  tools:
  - OrthoFinder
  - Scoary
  - Scoary2
  edam_operation: http://edamontology.org/operation_0326
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# software-benchmark-runtime-measurement

## Summary

Head-to-head wall-clock benchmarking of a new bioinformatics tool (Scoary2) against its predecessor (Scoary) on the paper's own yogurt pan-genome datasets, using equivalent parameters and pre-defined success thresholds.

## When to use

Use when a paper claims a tool is substantially faster than an existing one and the runtime claim (e.g. Scoary2 being 59x faster on 100 traits) must be reproduced on the deposited benchmark data.

## When NOT to use

- The goal is association results themselves (gene-trait hits such as the two carnitine-metabolism genes) rather than runtime comparison — a single Scoary2 run suffices
- The pan-genome input has not been clustered into orthogroups by OrthoFinder or Roary; runtime benchmarking requires the ready orthogroup/traits matrices
- The evaluation question concerns Scoary2's numeric-trait pre-processing or post-GWAS features, which Scoary lacks entirely and thus cannot be compared on equal footing

## Inputs

- Scoary2 source code and deposited yogurt dataset (github.com/MrTomRod/scoary-2)
- Original Scoary command-line tool
- Yogurt pan-genome matrix: 10358 hierarchical orthogroups x 182 isolates (full) and 1466 orthogroups x 44 isolates (reduced)
- Randomly selected 100-trait benchmark phenotype subset
- Paper's association run parameters for both tools

## Outputs

- benchmark_runtimes.csv with wall-clock times per tool per dataset subset
- Measured speedup ratio compared against the reported figures (23 s vs 22 min full; 43 s and 16 min reduced)

## How to apply

Obtain both tools — Scoary2 from github.com/MrTomRod/scoary-2 and the original Scoary command-line tool — plus the deposited yogurt dataset (3889 traits, 182 isolates, 10358 hierarchical orthogroups) generated with OrthoFinder. Prepare two dataset subsets matching the paper: the full dataset (3889 traits, 182 isolates) and a reduced dataset (44 isolates, 1466 orthogroups), and randomly select 100 traits as the benchmark phenotype subset. Run Scoary2 and original Scoary with equivalent parameters on both subsets, recording wall-clock time for each of the four runs, and tabulate the measured runtimes in benchmark_runtimes.csv. Decision point: accept the benchmark if Scoary2 completes the full-dataset association in approximately 23 s versus Scoary's approximately 22 min (about a 59x speedup on the 100-trait subset), and the reduced-dataset runs finish in approximately 43 s (Scoary2) and 16 min (Scoary), matching the paper's reported performance figures. The rationale is that Scoary's documented performance constraints prevent high-throughput analyses, so runtime parity of parameters is essential for a fair comparison.

## Related tools

- **Scoary2** (Benchmarked tool whose wall-clock runtime on the yogurt datasets is measured and compared against the reported 43 s (reduced) / 23 s (full 100-trait) figures) — https://github.com/MrTomRod/scoary-2
- **Scoary** (Baseline tool run with equivalent parameters on the same subsets to establish the ~22 min (full) / ~16 min (reduced) reference runtimes)
- **OrthoFinder** (Generator of the hierarchical orthogroups (10358 full / 1466 reduced) that form the fixed genotype input held constant across benchmark runs) — https://github.com/davidemms/OrthoFinder/

## Evaluation signals

- Scoary2 finishes the full-dataset (3889 traits, 182 isolates, 10358 orthogroups) association in approximately 23 s on the 100-trait subset
- Original Scoary takes approximately 22 min on the equivalent full-dataset run, yielding the reported ~59x speedup
- Reduced-dataset (44 isolates, 1466 orthogroups) runtimes land at approximately 43 s (Scoary2) and 16 min (Scoary)
- benchmark_runtimes.csv contains four wall-clock measurements covering both tools on both dataset subsets
- Both tools were invoked with the paper's parameters so the comparison is configuration-equivalent

## Limitations

- Runtime figures are hardware-dependent; the ~59x speedup and absolute times (23 s, 43 s, 16 min, 22 min) must be re-measured on the local machine rather than assumed
- Scoary cannot pre-process numeric traits and lacks post-GWAS methods, so only the core association runtime is comparable, not full-pipeline performance
- Discussion-section text was not fully available in the extracted document, so no additional runtime caveats from the authors' discussion could be captured

## Evidence

- [abstract] Scoary2's performance claim: "we introduce Scoary2, an ultra-fast microbial genome-wide association studies (mGWAS) software"
- [intro] Scoary's performance constraints motivating the benchmark: "Three limitations prevent Scoary from such high-throughput analyses: performance constraints, the inability to pre-process numeric traits, and a lack of post-GWAS methods optimized for the"
- [discussion] Performance expansion enabling large-trait datasets: "We expanded Scoary’s applicability to datasets containing tens of thousands of traits by significantly increasing the performance of the algorithm."
- [readme] Scoary2 function and citation grounding the benchmarked tool: "Roder, T. _et al._ _Scoary2_: rapid association of phenotypic multi-omics data with microbial pan-genomes."
