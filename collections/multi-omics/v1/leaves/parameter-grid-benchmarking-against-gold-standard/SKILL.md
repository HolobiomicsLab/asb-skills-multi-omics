---
name: parameter-grid-benchmarking-against-gold-standard
description: Use when tuning a multi-omics activity-inference pipeline (e.g., SignalingProfiler
  2.0) against a curated gold-standard protein/TF activity list to pick default parameters
  before applying the tool to new datasets.
when_to_use_negative:
- When the goal is mechanistic network reconstruction or phenotype connection (Steps
  2–3 with CARNIVAL/ProxPath) rather than benchmarking Step 1 activity inference
- When no curated gold-standard activity list exists for the training data — precision/recall/RMSE
  cannot be computed
- When metabolomics or other omics modalities need integration, which SignalingProfiler
  2.0 does not support
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3599
tools:
- name: SignalingProfiler 2.0
  role: Pipeline whose Step 1 (TFEA, KSEA, PhosphoScore) is swept across the parameter
    grid and scored against the gold standard
  repo: https://github.com/SaccoPerfettoLab/SignalingProfiler/
- name: VIPER
  role: Footprint-based algorithm underlying the TFEA transcription-factor activity
    estimation on transcriptomics data
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:5e24865dfb960799590192ad0acf349a1161595c3d8dc2c3da8701946cfec458
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SaccoPerfettoLab/SignalingProfiler
  tools:
  - SignalingProfiler 2.0
  - VIPER
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3599
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# parameter-grid-benchmarking-against-gold-standard

## Summary

Systematically sweep SignalingProfiler 2.0 analysis parameters (reference databases, regulon sizes, correction flags) across a condition grid on a training multi-omics dataset, then score each condition's inferred protein activities against a curated gold-standard list using precision, recall, and RMSE to select default settings.

## When to use

Use when tuning a multi-omics activity-inference pipeline (e.g., SignalingProfiler 2.0) against a curated gold-standard protein/TF activity list to pick default parameters before applying the tool to new datasets.

## When NOT to use

- When the goal is mechanistic network reconstruction or phenotype connection (Steps 2–3 with CARNIVAL/ProxPath) rather than benchmarking Step 1 activity inference
- When no curated gold-standard activity list exists for the training data — precision/recall/RMSE cannot be computed
- When metabolomics or other omics modalities need integration, which SignalingProfiler 2.0 does not support

## Inputs

- Metformin MCF7 training dataset transcriptomics table (ref. 30)
- Metformin MCF7 proteomics table (ref. 30)
- Metformin MCF7 phosphoproteomics table (ref. 30)
- 74-protein gold-standard activity list (17 TFs, 20 kinases/phosphatases, 27 other phosphorylated proteins)
- tfea_db_human DoRothEA/SIGNOR TF regulons
- ksea_db_human OmniPath/SIGNOR/Ser-Thr-Tyr Kinome Atlas kinase/phosphatase regulons
- good_phos_df_human_act regulatory phosphosite annotations (SIGNOR, PhosphoSitePlus)

## Outputs

- Per-condition protein activity tables (UNIPROT, gene_name, mf, final_score, method) for each grid condition
- parameter_grid_metrics.csv with precision, recall, and RMSE per condition
- Best-performing parameter combination recorded as SignalingProfiler 2.0 default settings

## How to apply

Load the training dataset's transcriptomics, proteomics, and phosphoproteomics tables (here the metformin-treated MCF7 data, ref. 30) together with the gold-standard activity list (74 proteins: 17 TFs, 20 kinases/phosphatases, 27 other proteins), and install SignalingProfiler 2.0 from GitHub. Run a 64-condition footprint-based TFEA grid on transcriptomics (run_footprint_based_analysis with analysis='tfea', organism='human', tfea_db_human DoRothEA/SIGNOR regulons, varying reg_minsize (e.g., 10), hypergeom_corr=TRUE, correct_proteomics=TRUE, collectri=FALSE) and a 32-condition KSEA grid on phosphoproteomics (analysis='ksea', reg_minsize=5, integrated_regulons=TRUE, ksea_db_human OmniPath/SIGNOR/Ser-Thr-Tyr Kinome Atlas regulons). Sweep the 4-condition phosphoscore grid for other phosphorylated proteins with phosphoscore_computation(activatory=TRUE, GO_annotation=TRUE) using good_phos_df_human_act regulatory phosphosites from SIGNOR and PhosphoSitePlus. Combine outputs with combine_footprint_and_phosphoscore, classifying remaining PhosphoScore proteins as OTHER, then compute precision, recall, and RMSE of every grid condition against the gold standard. Record the best-performing combination in parameter_grid_metrics.csv and adopt it as the SignalingProfiler defaults; the tuned settings should then transfer to validation datasets with comparable performance, indicating limited overfitting.

## Related tools

- **SignalingProfiler 2.0** (Pipeline whose Step 1 (TFEA, KSEA, PhosphoScore) is swept across the parameter grid and scored against the gold standard) — https://github.com/SaccoPerfettoLab/SignalingProfiler/
- **VIPER** (Footprint-based algorithm underlying the TFEA transcription-factor activity estimation on transcriptomics data)

## Examples

```
run_footprint_based_analysis(analysis='tfea', organism='human', reg_minsize=10, hypergeom_corr=TRUE, correct_proteomics=TRUE, collectri=FALSE)
```

## Evaluation signals

- Tuned Step 1 recovers 7/17 (40%) gold-standard transcription factors, 8/20 kinases/phosphatases, and 8/27 other phosphorylated proteins on the metformin training data
- Every grid condition yields a complete activity table with columns UNIPROT, gene_name, mf, final_score, method, and TF/kinase/OTHER assignments
- parameter_grid_metrics.csv contains precision, recall, and RMSE for all 64 TF-grid, 32 kinase-grid, and 4 phosphoscore-grid conditions
- The best-performing parameter combination, when applied to murine (ref. 20) and human (ref. 39) validation datasets, shows comparable performance, indicating limited overfitting

## Limitations

- Regulon and causal-interaction databases are incomplete, covering <50% (~9,000 proteins) of the UniProt-SwissProt proteome, capping recoverable gold-standard proteins
- PsP Regulatory_sites and Kinase_Substrates files must be manually downloaded from PhosphoSitePlus to include PsP information
- Transcription factor recovery is partial (up to 7 of 17, i.e., 40%), so the benchmark ceiling is bounded by prior-knowledge coverage

## Evidence

- [results] Step 1 parameter tuning enabled SignalingProfiler 2.0 to infer the activity of up to 7 of the 17 (40%) transcription factors in the protein gold standard.: [evidence span withheld — anchor status: not_found]
- [discussion] a benchmarking approach consisting of a parameter tuning phase on a training dataset30 and a subsequent validation phase: "a benchmarking approach consisting of a parameter tuning phase on a training dataset30 and a subsequent validation phase on murine20 and human39 datasets"
- [discussion] the parameter settings determined during the training process exhibited comparable performance (Fig. 6A) when applied to the validation datasets, demonstrating their broad applicability and limited: "the parameter settings determined during the training process exhibited comparable performance (Fig. 6A) when applied to the validation datasets, demonstrating their broad applicability and limited"
- [readme] devtools::install_github('https://github.com/SaccoPerfettoLab/SignalingProfiler/'): "devtools::install_github('https://github.com/SaccoPerfettoLab/SignalingProfiler/')"
- [intro] Step 1 derives the activity of signaling proteins from regulatory phosphosites (PhosphoScore method) and direct transcripts/phosphopeptides using the VIPER algorithm (footprint-based methods)25: "Step 1 derives the activity of signaling proteins from regulatory phosphosites (PhosphoScore method) and direct transcripts/phosphopeptides using the VIPER algorithm (footprint-based methods)25"
