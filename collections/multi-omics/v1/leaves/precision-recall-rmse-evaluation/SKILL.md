---
name: precision-recall-rmse-evaluation
description: Use when a multi-omics inference pipeline (e.g. SignalingProfiler 2.0
  Step 1) has been run under multiple parameter or resource conditions and you must
  quantify recovery of known gold-standard protein activities via precision, recall,
  and RMSE.
when_to_use_negative:
- No gold-standard activity list exists for the perturbation context — precision/recall/RMSE
  against a reference cannot be computed and only unsupervised plausibility checks
  apply
- You need mechanistic network outputs (Step 2 CARNIVAL networks or Step 3 ProxPath
  phenotypes) rather than Step 1 activity-inference quality metrics
- Input data includes metabolomics, which SignalingProfiler 2.0 explicitly does not
  support (COSMOS covers that case)
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: SignalingProfiler 2.0
  role: Generates the protein activity predictions (TFEA + KSEA + PhosphoScore) whose
    parameter conditions are scored for precision/recall/RMSE against the gold standard
  repo: github.com/SaccoPerfettoLab/SignalingProfiler
- name: VIPER
  role: Footprint-based algorithm underlying TFEA/KSEA activity inference evaluated
    by the benchmarking metrics
- name: CARNIVAL
  role: Step 2 optimization rerun on shuffled PKNs whose phosphorylation-edge coverage
    is the robustness metric
  repo: saezlab.github.io/CARNIVAL
merged_aliases:
- precision-recall-benchmark-evaluation
merged_alias_records:
- alias: precision-recall-benchmark-evaluation
  slug: precision-recall-benchmark-evaluation
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:397ae826758dd1890a37f206a090c9274f257d8caf2ad3740c0259507f84fdce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SaccoPerfettoLab/SignalingProfiler
  tools:
  - CARNIVAL
  - SignalingProfiler 2.0
  - VIPER
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# precision-recall-rmse-evaluation

## Summary

Benchmark SignalingProfiler 2.0 protein-activity predictions by sweeping parameter grids (TFEA, KSEA, PhosphoScore) and scoring each condition's inferred activities against a gold-standard protein activity list using precision, recall, and RMSE, optionally under data/regulon/PKN randomization.

## When to use

Use when a multi-omics inference pipeline (e.g. SignalingProfiler 2.0 Step 1) has been run under multiple parameter or resource conditions and you must quantify recovery of known gold-standard protein activities via precision, recall, and RMSE.

## When NOT to use

- No gold-standard activity list exists for the perturbation context — precision/recall/RMSE against a reference cannot be computed and only unsupervised plausibility checks apply
- You need mechanistic network outputs (Step 2 CARNIVAL networks or Step 3 ProxPath phenotypes) rather than Step 1 activity-inference quality metrics
- Input data includes metabolomics, which SignalingProfiler 2.0 explicitly does not support (COSMOS covers that case)

## Inputs

- Metformin MCF7 transcriptomics, proteomics, and phosphoproteomics tables (ref. 30)
- 74-protein gold-standard activity list (17 TFs, 20 kinases/phosphatases, 27 other proteins)
- tfea_db_human and ksea_db_human regulon sets (DoRothEA/SIGNOR, OmniPath/SIGNOR/Ser-Thr-Tyr Kinome Atlas)
- good_phos_df_human_act regulatory phosphosite annotations (SIGNOR, PhosphoSitePlus)
- PKN_human_dir prior knowledge network
- 100 BiRewire-shuffled regulon/PKN replicates per 25%/50%/100% level

## Outputs

- parameter_grid_metrics.csv with precision, recall, and RMSE for every grid condition
- sensitivity_analysis_results.csv aggregating precision/recall and phosphorylation-edge coverage over shuffled replicates
- Best-performing parameter combination adopted as SignalingProfiler default settings
- Final merged activity tables (UNIPROT, gene_name, mf, final_score, method) per condition

## How to apply

Run the inference pipeline across the full parameter grid — for SignalingProfiler 2.0 this means the 64-condition TFEA grid on transcriptomics (reg_minsize=10, tfea_db_human DoRothEA/SIGNOR regulons), the 32-condition KSEA grid on phosphoproteomics (reg_minsize=5, ksea_db_human OmniPath/SIGNOR/Kinome Atlas regulons), and the 4-condition phosphoscore grid — then merge footprint and phosphoscore outputs with combine_footprint_and_phosphoscore into a final activity table. For each grid condition, compare inferred TF/kinase/other-protein activities against the gold standard (e.g. the 74-protein metformin MCF7 list with 17 TFs, 20 kinases, 27 other proteins) computing precision, recall, and RMSE. For robustness, generate 100 BiRewire edge-shuffled replicates of regulons and PKNs at 25%, 50%, and 100% levels (preserving degree structure), rerun Step 1 and the two-shots CARNIVAL optimization (mipGAP=0.05, timelimit=3600), and aggregate precision/recall against the unshuffled reference plus phosphorylation-edge coverage (is_quantified / is_significant edges). Record the best-performing combination (e.g. 7/17 TFs, 8/20 kinases, 8/27 other proteins recovered) as defaults in parameter_grid_metrics.csv.

## Related tools

- **SignalingProfiler 2.0** (Generates the protein activity predictions (TFEA + KSEA + PhosphoScore) whose parameter conditions are scored for precision/recall/RMSE against the gold standard) — github.com/SaccoPerfettoLab/SignalingProfiler
- **VIPER** (Footprint-based algorithm underlying TFEA/KSEA activity inference evaluated by the benchmarking metrics)
- **CARNIVAL** (Step 2 optimization rerun on shuffled PKNs whose phosphorylation-edge coverage is the robustness metric) — saezlab.github.io/CARNIVAL

## Examples

```
run_footprint_based_analysis(analysis='tfea', organism='human', reg_minsize=10, hypergeom_corr=TRUE, correct_proteomics=TRUE, collectri=FALSE)
```

## Evaluation signals

- Reported Step 1 recoveries reproduced: 7/17 gold-standard TFs (~40%), 8/20 kinases, 8/27 other proteins
- parameter_grid_metrics.csv contains precision, recall, and RMSE for every condition of the 64-condition TFEA, 32-condition KSEA, and 4-condition phosphoscore grids
- Regulon randomization at 50% and 100% shuffling yields no Step 1 results (complete failure), matching the reported sensitivity profile
- Merged activity tables carry the expected schema (UNIPROT, gene_name, mf, final_score, method)
- PKN shuffling shows reduced phosphorylation-edge coverage (is_quantified / is_significant edges) aggregated over 100 replicates

## Limitations

- Regulon and causal-interaction resources cover <50% (about 9,000 proteins) of the UniProt-SwissProt proteome, capping recoverable gold-standard proteins
- Predictions are extremely sensitive to regulon randomization — 50–100% shuffling produces complete failure, so metrics degrade sharply under degraded prior knowledge
- Parameter settings tuned on the training dataset transfer comparably to murine and human validation datasets, but the gold standard itself is partial (17 of potentially many TFs)

## Evidence

- [other] Compute precision, recall, and RMSE of every condition against the gold-standard protein activities.: [evidence span withheld — anchor status: not_found]
- [intro] Step 1 derives the activity of signaling proteins from regulatory phosphosites: "Step 1 derives the activity of signaling proteins from regulatory phosphosites (PhosphoScore method) and direct transcripts/phosphopeptides using the VIPER algorithm (footprint-based methods)25"
- [discussion] the parameter settings determined during the training process exhibited comparable performance: "the parameter settings determined during the training process exhibited comparable performance (Fig. 6A) when applied to the validation datasets"
- [readme] devtools::install_github('https://github.com/SaccoPerfettoLab/SignalingProfiler/'): "devtools::install_github('https://github.com/SaccoPerfettoLab/SignalingProfiler/')"
