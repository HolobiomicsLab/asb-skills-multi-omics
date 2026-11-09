---
name: command-line-tool-parameter-reproduction
description: 'Use when a paper reports command-line tool runtimes (e.g. Scoary2: 23
  s vs Scoary''s 22 min on the full yogurt dataset; 43 s vs 16 min reduced) and you
  must verify them by re-executing both tools under equivalent settings.'
when_to_use_negative:
- The analysis goal is to re-derive or extend Scoary2's association algorithm rather
  than reproduce its published runtime figures under equivalent parameters.
- No deposited dataset or published reference runtime is available, so measured times
  cannot be validated against reported figures.
- Input is a numeric multi-omics trait table requiring Scoary2's pre-processing/exploration
  app rather than a straight runtime benchmark of the association step.
tools:
- name: Scoary2
  role: 'Tool whose runtime is being reproduced: run on the 100-trait subset of the
    full and reduced yogurt datasets with the paper''s parameters'
  repo: https://github.com/MrTomRod/scoary-2
- name: Scoary
  role: Baseline tool run with equivalent parameters on the same subsets to measure
    the speed-up (approx. 22 min vs 23 s on the full dataset)
- name: OrthoFinder
  role: Upstream generator of the hierarchical orthogroups (10,358 for the full yogurt
    set) that form the pan-genome input to both association runs
  repo: https://github.com/davidemms/OrthoFinder/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:71254e4101eb2b6bf86c25c1b77ff0885688c08f1c6eff6274d1347bd9ae7b9f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MrTomRod/scoary-2
  tools:
  - OrthoFinder
  - Scoary
  - Scoary2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# command-line-tool-parameter-reproduction

## Summary

Reproduce published runtime benchmarks by running Scoary2 and the original Scoary with equivalent command-line parameters on the paper's yogurt datasets and comparing measured wall-clock times against the reported figures.

## When to use

Use when a paper reports command-line tool runtimes (e.g. Scoary2: 23 s vs Scoary's 22 min on the full yogurt dataset; 43 s vs 16 min reduced) and you must verify them by re-executing both tools under equivalent settings.

## When NOT to use

- The analysis goal is to re-derive or extend Scoary2's association algorithm rather than reproduce its published runtime figures under equivalent parameters.
- No deposited dataset or published reference runtime is available, so measured times cannot be validated against reported figures.
- Input is a numeric multi-omics trait table requiring Scoary2's pre-processing/exploration app rather than a straight runtime benchmark of the association step.

## Inputs

- Scoary2 source / CLI from github.com/MrTomRod/scoary-2 plus deposited yogurt dataset
- Original Scoary command-line tool (comparison baseline)
- Full yogurt pan-genome matrix: 3889 traits, 182 isolates, 10,358 hierarchical orthogroups
- Reduced yogurt dataset: 44 isolates, 1466 orthogroups
- Randomly selected 100-trait benchmark phenotype subset

## Outputs

- benchmark_runtimes.csv with measured wall-clock times for Scoary2 and Scoary on full and reduced datasets
- Comparison of measured runtimes against paper-reported figures (23 s vs 22 min full; 43 s and 16 min reduced)

## How to apply

Obtain Scoary2 from github.com/MrTomRod/scoary-2 and the original Scoary command-line tool, plus the deposited yogurt data (full set: 3889 traits, 182 isolates, 10,358 hierarchical orthogroups; reduced set: 44 isolates, 1466 orthogroups). Randomly select 100 traits as the benchmark phenotype subset. Run Scoary2 on the 100-trait subset for both the full and reduced datasets using the paper's parameters, recording wall-clock time, then run original Scoary with equivalent parameters on the same subsets and record its wall-clock time. Tabulate measured runtimes in benchmark_runtimes.csv and compare against the reported figures (23 s vs 22 min full; 43 s and 16 min reduced). Accept the benchmark only if Scoary2 completes the full-dataset association in approximately 23 s versus Scoary's approximately 22 min, and the reduced-dataset runs take approximately 43 s (Scoary2) and 16 min (Scoary).

## Related tools

- **Scoary2** (Tool whose runtime is being reproduced: run on the 100-trait subset of the full and reduced yogurt datasets with the paper's parameters) — https://github.com/MrTomRod/scoary-2
- **Scoary** (Baseline tool run with equivalent parameters on the same subsets to measure the speed-up (approx. 22 min vs 23 s on the full dataset))
- **OrthoFinder** (Upstream generator of the hierarchical orthogroups (10,358 for the full yogurt set) that form the pan-genome input to both association runs) — https://github.com/davidemms/OrthoFinder/

## Evaluation signals

- Scoary2 completes the full-dataset association (3889 traits, 182 isolates, 10,358 orthogroups) in approximately 23 s.
- Original Scoary with equivalent parameters takes approximately 22 min on the full dataset and approximately 16 min on the reduced dataset.
- Scoary2 completes the reduced dataset (44 isolates, 1466 orthogroups) in approximately 43 s.
- Both tools are invoked with equivalent settings, so the runtime comparison is apples-to-apples.
- Measured wall-clock times for all four runs are tabulated in benchmark_runtimes.csv and match the paper's reported figures.

## Limitations

- Reported runtimes (23 s vs 22 min full; 43 s and 16 min reduced) are hardware-dependent; reproducing them requires comparable compute, and the discussion text notes only that performance was 'significantly increasing'-level improved without per-hardware specs.
- The runtime claims apply to the yogurt datasets described in the paper (3889 traits, 182 isolates); they may not transfer to other pan-genome/trait combinations.
- The benchmark requires both Scoary2 and the original Scoary to be runnable with equivalent parameters; differences in accepted parameters break the comparison.

## Evidence

- [abstract] Scoary2 performance claim: "we introduce Scoary2, an ultra-fast microbial genome-wide association studies (mGWAS) software"
- [discussion] Performance improvement rationale: "We expanded Scoary’s applicability to datasets containing tens of thousands of traits by significantly increasing the performance of the algorithm."
- [readme] Scoary2 input definition: "Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder] or [Roary][roary] to traits."
