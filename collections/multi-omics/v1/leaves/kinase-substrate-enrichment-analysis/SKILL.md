---
name: kinase-substrate-enrichment-analysis
description: Use when you have phosphoproteomic data (UniProt ID, gene_name, aminoacid,
  position, sequence window, difference, logpval) from a perturbation experiment and
  need kinase/phosphatase activity estimates as input to network reconstruction.
when_to_use_negative:
- The dataset is transcriptomics-only — footprint-based TF activity (TFEA, VIPER)
  with DoRothEA/SIGNOR regulons is the appropriate step, not KSEA on phosphoproteomics
- You need to integrate metabolomic data — SignalingProfiler 2.0 (and thus this KSEA
  step) does not include metabolomics, unlike COSMOS
- The input regulation type is epigenetic, acetylomic or ubiquitylomic — such data
  cannot be integrated into the signaling analysis in this pipeline
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3520
tools:
- name: SignalingProfiler 2.0
  role: R package providing run_footprint_based_analysis(analysis='ksea'), phosphoscore_computation
    and combine_footprint_and_phosphoscore for kinase/phosphatase activity inference
  repo: https://github.com/SaccoPerfettoLab/SignalingProfiler/
- name: VIPER
  role: Footprint-based algorithm underlying the activity-inference step for transcripts/phosphopeptides
    in SignalingProfiler 2.0
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:04cdaa8281f79b033253b3a43c5ac1401ffc80deae94de802939773c2d617c80
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SaccoPerfettoLab/SignalingProfiler
  tools:
  - SignalingProfiler 2.0
  - VIPER
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# kinase-substrate-enrichment-analysis

## Summary

Infers kinase and phosphatase activities from case-vs-control phosphoproteomics using footprint-based Kinase-Substrate Enrichment Analysis (KSEA) with SIGNOR/OmniPath/Ser-Thr-Tyr Kinome Atlas regulons, complemented by PhosphoScore on regulatory phosphosites.

## When to use

Use when you have phosphoproteomic data (UniProt ID, gene_name, aminoacid, position, sequence window, difference, logpval) from a perturbation experiment and need kinase/phosphatase activity estimates as input to network reconstruction.

## When NOT to use

- The dataset is transcriptomics-only — footprint-based TF activity (TFEA, VIPER) with DoRothEA/SIGNOR regulons is the appropriate step, not KSEA on phosphoproteomics
- You need to integrate metabolomic data — SignalingProfiler 2.0 (and thus this KSEA step) does not include metabolomics, unlike COSMOS
- The input regulation type is epigenetic, acetylomic or ubiquitylomic — such data cannot be integrated into the signaling analysis in this pipeline

## Inputs

- Case-vs-control phosphoproteomic table with columns UniProt ID, gene_name, aminoacid, position, sequence window, difference, logpval, significant (one row per phosphosite)
- Kinase/phosphatase regulons from SIGNOR, OmniPath and Ser/Thr-Tyr Kinome Atlas (ksea_db_human)
- Regulatory phosphosite annotations from SIGNOR and PhosphoSitePlus (good_phos_df_human_act); PsP Regulatory_sites and Kinase_Substrates files downloaded manually

## Outputs

- Kinase/phosphatase activity table (UNIPROT, gene_name, mf, final_score, method) per parameter-grid condition
- Merged footprint+PhosphoScore activity table via combine_footprint_and_phosphoscore (analysis='ksea')
- Precision, recall and RMSE metrics against a protein gold standard (parameter_grid_metrics.csv)

## How to apply

Build kinase/phosphatase regulons with create_ksea_regulons(resources=c('SIGNOR','Omnipath'), organism='human') and, in SignalingProfiler 2.0, run run_footprint_based_analysis(analysis='ksea', organism='human', reg_minsize=5, integrated_regulons=TRUE, hypergeom_corr=TRUE, correct_proteomics=TRUE) sweeping the 32-condition parameter grid (training showed best performance at the defaults). Ensure the input phosphoproteomic table contains only one occurrence of each phosphosite. In parallel, compute PhosphoScore with phosphoscore_computation(activatory=TRUE) on regulatory phosphosites from SIGNOR and PhosphoSitePlus, then merge the footprint and phosphoscore outputs with combine_footprint_and_phosphoscore(analysis='ksea') so each remaining protein is classified as kinase/phosphatase or OTHER in a final table (UNIPROT, gene_name, mf, final_score, method). Parameter choices were tuned on the metformin MCF7 training dataset against a 74-protein gold standard, where KSEA recovered 8/20 kinases.

## Related tools

- **SignalingProfiler 2.0** (R package providing run_footprint_based_analysis(analysis='ksea'), phosphoscore_computation and combine_footprint_and_phosphoscore for kinase/phosphatase activity inference) — https://github.com/SaccoPerfettoLab/SignalingProfiler/
- **VIPER** (Footprint-based algorithm underlying the activity-inference step for transcripts/phosphopeptides in SignalingProfiler 2.0)

## Examples

```
run_footprint_based_analysis(analysis='ksea', organism='human', reg_minsize=5, integrated_regulons=TRUE, hypergeom_corr=TRUE, correct_proteomics=TRUE)
```

## Evaluation signals

- On the metformin MCF7 training dataset, the best grid condition recovers 8 of the 20 kinases in the 74-protein gold standard (7/17 TFs, 8/27 other proteins)
- Input table validation: only one occurrence of each phosphosite before running KSEA
- Regulon constraint check: reg_minsize=5 for KSEA regulons (vs reg_minsize=10 for TFEA)
- Merged output schema check: final table contains UNIPROT, gene_name, mf, final_score, method, with leftover PhosphoScore proteins classified as OTHER
- Metrics computed for every grid condition: precision, recall and RMSE against gold-standard protein activities, recorded in parameter_grid_metrics.csv

## Limitations

- Regulon databases and causal interaction resources are incomplete and cover <50% (~9,000 proteins) of the UniProt-SwissProt proteome, limiting which kinases/phosphatases can be inferred
- PhosphoSitePlus Regulatory_sites and Kinase_Substrates files must be manually downloaded if PsP information is included
- SignalingProfiler is under active development and users are advised to delete and re-install the package regularly; a prior-knowledge update step is needed to keep regulons current

## Evidence

- [methods] KSEA regulon construction and resources: "ksea_regulons <- create_ksea_regulons(resources = c('SIGNOR', 'Omnipath'), organism = 'human')"
- [intro] Step 1 protein activity inference rationale: "Step 1 derives the activity of signaling proteins from regulatory phosphosites (PhosphoScore method) and direct transcripts/phosphopeptides using the VIPER algorithm (footprint-based methods)25"
- [discussion] Unique capability covering proteins beyond kinases/phosphatases: "is the sole tool capable of estimating the activity of proteins, aside from kinases and phosphatases, from the phosphoproteomic data"
- [readme] Pipeline input data types: [evidence span withheld — anchor status: not_found]
- [results] Training benchmark result used to set KSEA defaults: [evidence span withheld — anchor status: not_found]
