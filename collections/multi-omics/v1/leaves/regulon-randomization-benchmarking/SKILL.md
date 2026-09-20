---
name: regulon-randomization-benchmarking
description: Use when stress-testing SignalingProfiler 2.0 predictions against degraded
  prior knowledge — e.g., quantifying how partially shuffled regulons (tfea/ksea)
  or PKN_human_dir rewirings affect Step 1 activity inference or Step 2 network reconstruction.
when_to_use_negative:
- The goal is benchmarking prediction accuracy on real validation data (murine ref.
  20 / human ref. 39 datasets) rather than synthetic robustness to prior-knowledge
  degradation
- The prior-knowledge resource lacks edge/degree structure amenable to BiRewire rewiring
  (e.g., purely tabular scoring tables without a regulon or PKN graph)
- Metabolomic or epigenetic/acetylomic/ubiquitylomic data are in scope — SignalingProfiler
  2.0 does not include metabolomic data and these regulation types cannot be integrated
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SignalingProfiler 2.0
  role: Pipeline under test; Step 1 (TFEA/KSEA footprint-based analysis + PhosphoScore)
    and Step 2 (PKN preprocessing, two_layer_naive_network, CARNIVAL optimization)
    are rerun on each shuffled resource
  repo: https://github.com/SaccoPerfettoLab/SignalingProfiler/
- name: VIPER
  role: Footprint-based algorithm inside Step 1 whose TF/kinase activity estimates
    are recomputed against shuffled tfea/ksea regulons to measure precision/recall
    degradation
- name: CARNIVAL
  role: ILP optimization (cplex solver, mipGAP = 0.05, timelimit = 3600) rerun on
    shuffled PKN_human_dir to measure how network context-specificity loss reduces
    phosphorylation-edge coverage
  repo: https://saezlab.github.io/CARNIVAL/
- name: BiRewire
  role: Edge-shuffling engine generating degree-preserving rewired regulon/PKN replicates
    at 25%, 50%, and 100% levels
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:76e6528e47b9160656fa5fab785a0f9e35a7e0994cf9c0ca4ce0353df38d68bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SaccoPerfettoLab/SignalingProfiler
  tools:
  - BiRewire
  - CARNIVAL
  - SignalingProfiler 2.0
  - VIPER
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# regulon-randomization-benchmarking

## Summary

A robustness benchmarking skill for SignalingProfiler 2.0 that perturbs its prior-knowledge resources (TF/kinase regulons and the PKN) via BiRewire edge shuffling and reruns the pipeline to quantify how precision/recall of inferred protein activities and phosphorylation-edge coverage degrade with increasing randomization.

## When to use

Use when stress-testing SignalingProfiler 2.0 predictions against degraded prior knowledge — e.g., quantifying how partially shuffled regulons (tfea/ksea) or PKN_human_dir rewirings affect Step 1 activity inference or Step 2 network reconstruction.

## When NOT to use

- The goal is benchmarking prediction accuracy on real validation data (murine ref. 20 / human ref. 39 datasets) rather than synthetic robustness to prior-knowledge degradation
- The prior-knowledge resource lacks edge/degree structure amenable to BiRewire rewiring (e.g., purely tabular scoring tables without a regulon or PKN graph)
- Metabolomic or epigenetic/acetylomic/ubiquitylomic data are in scope — SignalingProfiler 2.0 does not include metabolomic data and these regulation types cannot be integrated

## Inputs

- Reference multi-omics dataset (transcriptomics, proteomics, phosphoproteomics; case vs. control)
- tfea_db_human / tfea_db_human_collectri transcription factor regulons
- ksea_db_human kinase/phosphatase regulons
- PKN_human_dir prior knowledge network (SIGNOR/PsP-derived, directed)
- BiRewire-shuffled regulon and PKN versions (25%, 50%, 100% edge shuffling; 100 replicates per level per resource)

## Outputs

- sensitivity_analysis_results.csv with per-replicate and aggregated sensitivity metrics
- Precision/recall of inferred protein activities vs. the unshuffled reference per shuffle level
- Phosphorylation-edge coverage counts (is_quantified / is_significant edges) for each shuffled PKN
- Aggregated robustness profile across shuffle levels and replicates

## How to apply

Load the reference multi-omics data plus the built-in resources (tfea_db_human/tfea_db_human_collectri TF regulons, ksea_db_human kinase/phosphatase regulons, and PKN_human_dir). Generate rewired resource versions with BiRewire edge shuffling at 25%, 50%, and 100% levels — 100 replicate rewirings per level per resource — preserving degree structure while disrupting edge specificity. Rerun Step 1 on each shuffled regulon set with default parameters: run_footprint_based_analysis for TFEA (reg_minsize = 10) and KSEA (reg_minsize = 5), then phosphoscore_computation and combine_footprint_and_phosphoscore; for PKN shuffling, rerun Step 2 via preprocess_PKN, two_layer_naive_network, and two-shots CARNIVAL optimization (default_CARNIVAL_options, mipGAP = 0.05, timelimit = 3600) followed by expand_and_map_edges. Compute precision/recall of inferred protein activities against the unshuffled reference per shuffle level, and phosphorylation-edge coverage (is_quantified / is_significant edges) for the PKN arm, aggregating over the 100 replicates into sensitivity_analysis_results.csv. The rationale mirrors the paper's benchmarking philosophy — a parameter tuning phase on a training dataset followed by validation on murine and human datasets — extended here to prior-knowledge perturbation.

## Related tools

- **SignalingProfiler 2.0** (Pipeline under test; Step 1 (TFEA/KSEA footprint-based analysis + PhosphoScore) and Step 2 (PKN preprocessing, two_layer_naive_network, CARNIVAL optimization) are rerun on each shuffled resource) — https://github.com/SaccoPerfettoLab/SignalingProfiler/
- **VIPER** (Footprint-based algorithm inside Step 1 whose TF/kinase activity estimates are recomputed against shuffled tfea/ksea regulons to measure precision/recall degradation)
- **CARNIVAL** (ILP optimization (cplex solver, mipGAP = 0.05, timelimit = 3600) rerun on shuffled PKN_human_dir to measure how network context-specificity loss reduces phosphorylation-edge coverage) — https://saezlab.github.io/CARNIVAL/
- **BiRewire** (Edge-shuffling engine generating degree-preserving rewired regulon/PKN replicates at 25%, 50%, and 100% levels)

## Examples

```
ksea_regulons <- create_ksea_regulons(resources = c('SIGNOR', 'Omnipath'), organism = 'human')
```

## Evaluation signals

- At 25% regulon shuffling, Step 1 shows a measurable but partial decline in precision/recall vs. the unshuffled reference; at 50% and 100% shuffling, run_footprint_based_analysis produces no results (complete failure), matching the reported profile
- Each rewired resource preserves node degree structure (BiRewire invariant) while disrupting edge specificity — verify degree sequences are unchanged across all 100 replicates per level
- PKN shuffling yields monotonically reduced phosphorylation-edge coverage (is_quantified / is_significant edges) as shuffle level increases from 25% to 100%
- All metrics are aggregated over 100 replicates per shuffle level per resource and written to sensitivity_analysis_results.csv
- Default Step 1/Step 2 parameters are held fixed across conditions (reg_minsize = 10 for TFEA, 5 for KSEA; mipGAP = 0.05, timelimit = 3600) so degradation is attributable to shuffling alone

## Limitations

- Step 1 predictions are extremely sensitive to regulon randomization: shuffling 50% or 100% of regulon entries produced no results, so no robustness estimate exists at those levels
- Incomplete prior knowledge coverage: regulon and causal interaction databases cover <50% (~9,000 proteins) of the UniProt-SwissProt proteome, capping the rewirable edge space
- The ILP solver path must be manually specified and PsP Regulatory_sites / Kinase_Substrates files must be manually downloaded, adding setup friction to the Step 2 shuffling arm

## Evidence

- [intro] Step 1 derives the activity of signaling proteins from regulatory phosphosites (PhosphoScore method) and direct transcripts/phosphopeptides using the VIPER algorithm: "Step 1 derives the activity of signaling proteins from regulatory phosphosites (PhosphoScore method) and direct transcripts/phosphopeptides using the VIPER algorithm (footprint-based methods)25"
- [discussion] Benchmarking philosophy: parameter tuning then validation on murine and human datasets: "a parameter tuning phase on a training dataset30 and a subsequent validation phase on murine20 and human39 datasets"
- [methods] CARNIVAL options configured via default_CARNIVAL_options for the optimization step rerun on shuffled PKNs: "carnival_options = default_CARNIVAL_options(solver)"
- [methods] Regulon construction from SIGNOR and Omnipath resources via create_ksea_regulons: "ksea_regulons <- create_ksea_regulons(resources = c('SIGNOR', 'Omnipath'), organism = 'human')"
- [readme] SignalingProfiler is installed from the SaccoPerfettoLab GitHub repository: "devtools::install_github('https://github.com/SaccoPerfettoLab/SignalingProfiler/')"
