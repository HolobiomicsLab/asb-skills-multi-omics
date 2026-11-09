---
name: transcription-factor-activity-inference-from-target-genes
description: Use when a multi-omics case-vs-control study (e.g., metformin-treated
  MCF7 transcriptomics) needs TF activity inferred from target-gene expression signatures
  as Step 1 of a SignalingProfiler 2.0 pipeline.
when_to_use_negative:
- Input is phosphoproteomics-only without transcriptomics — TFEA needs target-gene
  expression; phosphorylated-protein activity should instead go through phosphoscore_computation
- Kinase/phosphatase activity is the target — use the KSEA branch (run_footprint_based_analysis(analysis='ksea',
  reg_minsize=5) on ksea_db_human OmniPath/SIGNOR/Ser-Thr-Tyr Kinome Atlas regulons)
- Phenotypic interpretation or network reconstruction (Steps 2–3, CARNIVAL/ProxPath)
  rather than Step 1 protein activity inference is the goal
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SignalingProfiler 2.0
  role: R pipeline providing run_footprint_based_analysis, create_tfea_regulons, combine_footprint_and_phosphoscore,
    and the parameter-grid benchmarking framework for TF activity inference
  repo: https://github.com/SaccoPerfettoLab/SignalingProfiler/
- name: VIPER
  role: Footprint-based algorithm underlying TFEA that infers TF activity from transcriptomics/phosphopeptide
    target-gene signatures
- name: DoRothEA
  role: Source of TF-target regulons combined with SIGNOR into tfea_db_human
- name: SIGNOR
  role: Causal interaction database supplying TF-target and regulatory phosphosite
    information alongside DoRothEA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:02b7d4c44ee36dfd3a1f8254afa92d74431919a9b2eb678154cd5459994ada8f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SaccoPerfettoLab/SignalingProfiler
  tools:
  - DoRothEA
  - SIGNOR
  - SignalingProfiler 2.0
  - VIPER
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# transcription-factor-activity-inference-from-target-genes

## Summary

Infers transcription factor (TF) activity from transcriptomics data using footprint-based target-gene enrichment (TFEA) implemented in SignalingProfiler 2.0 via the VIPER algorithm, with regulons built from DoRothEA and SIGNOR resources. In the paper this was the mechanism by which 7 of 17 (40%) gold-standard TFs were recovered during parameter tuning on the metformin-treated MCF7 breast cancer training dataset.

## When to use

Use when a multi-omics case-vs-control study (e.g., metformin-treated MCF7 transcriptomics) needs TF activity inferred from target-gene expression signatures as Step 1 of a SignalingProfiler 2.0 pipeline.

## When NOT to use

- Input is phosphoproteomics-only without transcriptomics — TFEA needs target-gene expression; phosphorylated-protein activity should instead go through phosphoscore_computation
- Kinase/phosphatase activity is the target — use the KSEA branch (run_footprint_based_analysis(analysis='ksea', reg_minsize=5) on ksea_db_human OmniPath/SIGNOR/Ser-Thr-Tyr Kinome Atlas regulons)
- Phenotypic interpretation or network reconstruction (Steps 2–3, CARNIVAL/ProxPath) rather than Step 1 protein activity inference is the goal

## Inputs

- Case-vs-control transcriptomics table (e.g., metformin MCF7 dataset, ref. 30)
- tfea_db_human regulon set built from DoRothEA and SIGNOR via create_tfea_regulons
- Protein gold-standard activity list (74 proteins, 17 transcription factors)
- Proteomics table (used by correct_proteomics for TF abundance correction)

## Outputs

- TF activity scores (final_score) with method labels, per grid condition
- Final activity table with columns UNIPROT, gene_name, mf, final_score, method after combine_footprint_and_phosphoscore
- parameter_grid_metrics.csv with precision, recall, and RMSE for each of the 64 TF grid conditions
- Recovered-gold-standard counts (7/17 TFs, 40% recovery in the metformin training dataset)

## How to apply

Build TF regulons with create_tfea_regulons(resources = c('SIGNOR', 'Dorothea'), organism = 'human') to produce tfea_db_human. Run run_footprint_based_analysis(analysis='tfea', organism='human', reg_minsize=10, hypergeom_corr=TRUE, correct_proteomics=TRUE, collectri=FALSE) on the transcriptomics table; reg_minsize=10 filters out regulons with fewer than 10 targets, hypergeom_corr applies hypergeometric correction, and correct_proteomics adjusts for proteomics-inferred TF abundances (CollectRI is deliberately excluded in the tuned default grid). Sweep the 64-condition TF parameter grid, then combine footprint results with PhosphoScore outputs using combine_footprint_and_phosphoscore to yield a final activity table with columns UNIPROT, gene_name, mf, final_score, and method. Assess each grid condition by computing precision, recall, and RMSE against a 74-protein gold-standard activity list, and record the best combination as the tool defaults in parameter_grid_metrics.csv.

## Related tools

- **SignalingProfiler 2.0** (R pipeline providing run_footprint_based_analysis, create_tfea_regulons, combine_footprint_and_phosphoscore, and the parameter-grid benchmarking framework for TF activity inference) — https://github.com/SaccoPerfettoLab/SignalingProfiler/
- **VIPER** (Footprint-based algorithm underlying TFEA that infers TF activity from transcriptomics/phosphopeptide target-gene signatures)
- **DoRothEA** (Source of TF-target regulons combined with SIGNOR into tfea_db_human)
- **SIGNOR** (Causal interaction database supplying TF-target and regulatory phosphosite information alongside DoRothEA)

## Examples

```
run_footprint_based_analysis(analysis='tfea', organism='human', reg_minsize=10, hypergeom_corr=TRUE, correct_proteomics=TRUE, collectri=FALSE)
```

## Evaluation signals

- Recovery against the 74-protein gold-standard activity list: the tuned TF grid should recover up to 7 of 17 (40%) transcription factors
- parameter_grid_metrics.csv exists and reports precision, recall, and RMSE for all 64 TF grid conditions, with the best performer flagged as default (reg_minsize=10, hypergeom_corr=TRUE, correct_proteomics=TRUE, collectri=FALSE)
- Output table schema check: columns UNIPROT, gene_name, mf, final_score, method; TF rows labeled with method = TFEA/VIPER footprint
- Regulon provenance check: tfea_db_human was built from resources c('SIGNOR', 'Dorothea') for organism='human'; CollectRI is off in the default grid
- TF scores integrate cleanly with PhosphoScore outputs via combine_footprint_and_phosphoscore, with residual PhosphoScore proteins classified as OTHER

## Limitations

- Regulon databases and causal interaction resources are incomplete and offer information for <50% (about 9,000 proteins) of the UniProt-SwissProt proteome, capping achievable TF recall
- Coverage depends on the regulon source: CollectRI was excluded from the tuned TF grid, so recoverable TFs are limited to DoRothEA/SIGNOR membership
- SignalingProfiler 2.0 is under active development; the README advises deleting and re-installing the package regularly to exploit new features (R >= 4.1 required)

## Evidence

- [intro] Step 1 derives the activity of signaling proteins from regulatory phosphosites (PhosphoScore method) and direct transcripts/phosphopeptides using the VIPER algorithm (footprint-based methods)25: "Step 1 derives the activity of signaling proteins from regulatory phosphosites (PhosphoScore method) and direct transcripts/phosphopeptides using the VIPER algorithm (footprint-based methods)25"
- [results] Step 1 parameter tuning enabled SignalingProfiler 2.0 to infer the activity of up to 7 of the 17 (40%) transcription factors in the protein gold standard.: [evidence span withheld — anchor status: not_found]
- [methods] tfea_regulons <- create_tfea_regulons(resources = c('SIGNOR', 'Dorothea'), organism = 'human'): "tfea_regulons <- create_tfea_regulons(resources = c('SIGNOR', 'Dorothea'), organism = 'human')"
- [discussion] regulon databases and causal interaction resources are incomplete and offer information for <50% (about 9,000 proteins) of the Uniprot-SwissProt proteome: "regulon databases and causal interaction resources are incomplete and offer information for <50% (about 9,000 proteins) of the Uniprot-SwissProt proteome"
- [readme] devtools::install_github('https://github.com/SaccoPerfettoLab/SignalingProfiler/'): "devtools::install_github('https://github.com/SaccoPerfettoLab/SignalingProfiler/')"
