---
name: multi-omics-protein-activity-inference
description: Use when matched case/control transcriptomics, proteomics, and phosphoproteomics
  tables are available and the goal is an unbiased activity score (mf/final_score)
  per signaling protein before network reconstruction or phenotyping.
when_to_use_negative:
- Input lacks matched transcriptomics and phosphoproteomics — TFEA requires transcripts
  and KSEA/PhosphoScore require phosphoproteomics and proteomics
- No case/control perturbation structure — activity inference contrasts signaling
  remodeling between two conditions
- Only metabolomics data are available — SignalingProfiler 2.0, unlike COSMOS, does
  not include metabolomic data
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2258
- http://edamontology.org/topic_0203
tools:
- name: SignalingProfiler 2.0
  role: R pipeline providing run_footprint_based_analysis, phosphoscore_computation,
    and combine_footprint_and_phosphoscore for Step 1
  repo: https://github.com/SaccoPerfettoLab/SignalingProfiler/
- name: VIPER
  role: Footprint-based algorithm underlying TFEA (transcripts) and KSEA (phosphopeptides)
    protein activity estimation
- name: PhosphoScore
  role: Method deriving activity of signaling proteins from regulatory phosphosites
    (SIGNOR/PhosphoSitePlus annotations)
merged_aliases:
- multi-omics-protein-activity-score-integration
merged_alias_records:
- alias: multi-omics-protein-activity-score-integration
  slug: multi-omics-protein-activity-score-integration
  cosine_score: 0.8726
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_005
  - task_007
  - task_002
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:c6ed67012ca29ba5a7fb3af2a254774224b79d2b57e39c9d4cfff547d65c7c38
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SaccoPerfettoLab/SignalingProfiler
  tools:
  - PhosphoScore
  - PhosphoSitePlus
  - SIGNOR
  - SignalingProfiler 2.0
  - VIPER
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2258
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# multi-omics-protein-activity-inference

## Summary

SignalingProfiler 2.0's Step 1 procedure for inferring protein (TF, kinase/phosphatase, and other phosphoprotein) activities from matched transcriptomics, proteomics, and phosphoproteomics by combining VIPER-based footprint methods (TFEA, KSEA) with PhosphoScore regulatory phosphosite analysis.

## When to use

Use when matched case/control transcriptomics, proteomics, and phosphoproteomics tables are available and the goal is an unbiased activity score (mf/final_score) per signaling protein before network reconstruction or phenotyping.

## When NOT to use

- Input lacks matched transcriptomics and phosphoproteomics — TFEA requires transcripts and KSEA/PhosphoScore require phosphoproteomics and proteomics
- No case/control perturbation structure — activity inference contrasts signaling remodeling between two conditions
- Only metabolomics data are available — SignalingProfiler 2.0, unlike COSMOS, does not include metabolomic data

## Inputs

- Case/control transcriptomics table (e.g., metformin MCF7 dataset, ref. 30)
- Case/control proteomics table
- Deduplicated phosphoproteomics table (one occurrence per phosphosite)
- tfea_db_human DoRothEA/SIGNOR TF regulons
- ksea_db_human OmniPath/SIGNOR/Kinome Atlas kinase/phosphatase regulons
- good_phos_df_human_act regulatory phosphosite annotations (SIGNOR, PhosphoSitePlus)

## Outputs

- Protein activity table per grid condition (UNIPROT, gene_name, mf, final_score, method)
- TF activity scores from TFEA and kinase/phosphatase scores from KSEA
- PhosphoScore-derived activities for 'other' phosphorylated proteins
- Precision, recall, and RMSE metrics against the protein gold standard (parameter_grid_metrics.csv)

## How to apply

Run run_footprint_based_analysis twice: TFEA on transcriptomics with reg_minsize=10, hypergeom_corr=TRUE, correct_proteomics=TRUE using tfea_db_human DoRothEA/SIGNOR regulons (VIPER), and KSEA on phosphoproteomics with reg_minsize=5, integrated_regulons=TRUE using ksea_db_human OmniPath/SIGNOR/Ser-Thr-Tyr Kinome Atlas regulons. Separately compute phosphoscore_computation(activatory=TRUE, GO_annotation=TRUE) with good_phos_df_human_act regulatory phosphosite annotations from SIGNOR and PhosphoSitePlus for phosphorylated proteins other than kinases/phosphatases. Combine footprint and phosphoscore outputs with combine_footprint_and_phosphoscore separately for TFs and for kinases/phosphatases, labeling remaining PhosphoScore proteins as 'other'. Rationale: merging footprint-based and phosphosite-based evidence lets the pipeline estimate activity for ~27 'other' proteins that no other tool infers from phosphoproteomics alone; the best parameter combination was selected by sweeping 64 TFEA and 32 KSEA conditions against a 74-protein gold standard (7/17 TFs, 8/20 kinases, 8/27 other recovered).

## Related tools

- **SignalingProfiler 2.0** (R pipeline providing run_footprint_based_analysis, phosphoscore_computation, and combine_footprint_and_phosphoscore for Step 1) — https://github.com/SaccoPerfettoLab/SignalingProfiler/
- **VIPER** (Footprint-based algorithm underlying TFEA (transcripts) and KSEA (phosphopeptides) protein activity estimation)
- **PhosphoScore** (Method deriving activity of signaling proteins from regulatory phosphosites (SIGNOR/PhosphoSitePlus annotations))

## Examples

```
run_footprint_based_analysis(analysis='ksea', organism='human', reg_minsize=5, integrated_regulons=TRUE, hypergeom_corr=TRUE, correct_proteomics=TRUE)
```

## Evaluation signals

- Against the 74-protein metformin gold standard, the best parameter combination recovers 7/17 TFs, 8/20 kinases, and 8/27 other phosphorylated proteins (~40% TF recovery)
- Every grid condition's output table must carry columns UNIPROT, gene_name, mf, final_score, and method
- Precision, recall, and RMSE are computed for every one of the 64 TFEA x 32 KSEA x 4 PhosphoScore grid conditions and stored in parameter_grid_metrics.csv
- Robustness check: Step 1 predictions remain precise/recallable at 25% BiRewire regulon shuffling but completely fail (no results) at 50-100% shuffling
- Default (training-derived) settings show comparable precision/recall on the independent murine ac220 validation dataset, indicating no overfitting

## Limitations

- Regulon databases and causal interaction resources cover <50% (about 9,000 proteins) of the UniProt-SwissProt proteome, bounding recoverable activities
- Step 1 predictions are extremely sensitive to regulon randomization: shuffling 50% or 100% of regulon entries yields no results
- PsP Regulatory_sites and Kinase_Substrates files must be manually downloaded from the PsP database

## Evidence

- [intro] Step 1 derives the activity of signaling proteins from regulatory phosphosites (PhosphoScore method) and direct transcripts/phosphopeptides using the VIPER algorithm: "Step 1 derives the activity of signaling proteins from regulatory phosphosites (PhosphoScore method) and direct transcripts/phosphopeptides using the VIPER algorithm (footprint-based methods)25"
- [results] Parameter tuning recovered up to 7 of 17 gold-standard TFs: [evidence span withheld — anchor status: not_found]
- [methods] Full TFEA invocation with default-best parameters: [evidence span withheld — anchor status: not_found]
- [discussion] Training-derived settings generalize to validation datasets: "the parameter settings determined during the training process exhibited comparable performance (Fig. 6A) when applied to the validation datasets, demonstrating their broad applicability and limited"
- [readme] README installation command and R version requirement: "devtools::install_github('https://github.com/SaccoPerfettoLab/SignalingProfiler/')"
