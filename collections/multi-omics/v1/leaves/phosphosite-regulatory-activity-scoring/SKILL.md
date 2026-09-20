---
name: phosphosite-regulatory-activity-scoring
description: Use when a phosphoproteomic table (case vs control log-ratios per phosphosite)
  must be converted into protein-level activity scores for kinases, phosphatases,
  and other phosphorylated proteins via regulatory phosphosite annotations.
when_to_use_negative:
- The input is transcriptomics-only and the target proteins are transcription factors
  — use the VIPER-based TFEA branch (tfea_db_human DoRothEA/SIGNOR regulons) instead
  of PhosphoScore
- The dataset contains no phosphoproteomics (e.g. metabolomics questions) — SignalingProfiler
  2.0 explicitly does not include metabolomic data
- Only a handful of phosphosites are quantified and no SIGNOR/PsP regulatory-site
  coverage exists for them, leaving PhosphoScore without substrate annotations
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3308
tools:
- name: SignalingProfiler 2.0
  role: R package providing get_phosphoscore_info, phosphoscore_computation (activatory,
    GO_annotation flags) and combine_footprint_and_phosphoscore for the PhosphoScore
    scoring step
  repo: https://github.com/SaccoPerfettoLab/SignalingProfiler/
- name: SIGNOR
  role: Source database of regulatory (activatory) phosphosite annotations used by
    PhosphoScore
- name: PhosphoSitePlus
  role: Source of Regulatory_sites and Kinase_Substrates files merged into the regulatory
    phosphosite annotation set
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:7c98fd6f09b6457638727774ed41b0d16610d438c2c7eae356845a45ed1341f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SaccoPerfettoLab/SignalingProfiler
  tools:
  - PhosphoSitePlus
  - SIGNOR
  - SignalingProfiler 2.0
  - VIPER
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# phosphosite-regulatory-activity-scoring

## Summary

SignalingProfiler 2.0's PhosphoScore-based step that converts phosphoproteomic log-ratios into protein-level activity scores for kinases, phosphatases, and 'other' phosphorylated proteins, using regulatory phosphosite annotations curated from SIGNOR and PhosphoSitePlus.

## When to use

Use when a phosphoproteomic table (case vs control log-ratios per phosphosite) must be converted into protein-level activity scores for kinases, phosphatases, and other phosphorylated proteins via regulatory phosphosite annotations.

## When NOT to use

- The input is transcriptomics-only and the target proteins are transcription factors — use the VIPER-based TFEA branch (tfea_db_human DoRothEA/SIGNOR regulons) instead of PhosphoScore
- The dataset contains no phosphoproteomics (e.g. metabolomics questions) — SignalingProfiler 2.0 explicitly does not include metabolomic data
- Only a handful of phosphosites are quantified and no SIGNOR/PsP regulatory-site coverage exists for them, leaving PhosphoScore without substrate annotations

## Inputs

- Processed phosphoproteomic table with exactly one occurrence of each phosphosite: UniProt ID, gene_name, aminoacid, position, sequence window, difference, logpval, significant
- Regulatory phosphosite annotations (good_phos_df_human_act) sourced from SIGNOR and PhosphoSitePlus
- Optional proteomics table for correction (correct_proteomics=TRUE)
- Gold-standard protein activity list for benchmarking (e.g. the 74-protein metformin MCF7 gold standard)

## Outputs

- Per-protein activity scores (PhosphoScore final_score) for kinases, phosphatases, and OTHER phosphorylated proteins
- Combined activity table with columns UNIPROT, gene_name, mf, final_score, method after combine_footprint_and_phosphoscore
- Benchmark metrics (precision, recall, RMSE) per parameter-grid condition, recorded in parameter_grid_metrics.csv

## How to apply

Retrieve regulatory phosphosite annotations with get_phosphoscore_info(resources=c('SIGNOR','PsP')) (for human these come as good_phos_df_human_act; PsP Regulatory_sites/Kinase_Substrates files must be downloaded manually beforehand). Run phosphoscore_computation(activatory=TRUE, GO_annotation=TRUE) over the 4-condition parameter grid, ensuring the input phosphoproteomic table contains exactly one occurrence of each phosphosite with UniProt ID, gene_name, aminoacid, position, sequence window, difference, logpval and significant columns. Merge the PhosphoScore outputs with KSEA footprint-based kinase/phosphatase activities using combine_footprint_and_phosphoscore, classifying any remaining PhosphoScore proteins as OTHER. Benchmark the resulting activity table (precision, recall, RMSE against a gold-standard protein activity list) to pick the best-performing condition; in the metformin MCF7 tuning this recovered 8/20 kinases and 8/27 other proteins. Rationale: PhosphoScore aggregates regulatory-site evidence per protein, which the authors highlight as a capability no other tool offers for proteins that are neither kinases nor TFs.

## Related tools

- **SignalingProfiler 2.0** (R package providing get_phosphoscore_info, phosphoscore_computation (activatory, GO_annotation flags) and combine_footprint_and_phosphoscore for the PhosphoScore scoring step) — https://github.com/SaccoPerfettoLab/SignalingProfiler/
- **SIGNOR** (Source database of regulatory (activatory) phosphosite annotations used by PhosphoScore)
- **PhosphoSitePlus** (Source of Regulatory_sites and Kinase_Substrates files merged into the regulatory phosphosite annotation set)

## Examples

```
reg_phosphosites <- get_phosphoscore_info(resources = c('SIGNOR', 'PsP')); phosphoscore_computation(activatory=TRUE, GO_annotation=TRUE)
```

## Evaluation signals

- Input phosphoproteomic table passes the uniqueness check: ONLY ONE OCCURRENCE OF EACH PHOSPHOSITE with the full column set (UniProt ID, gene_name, aminoacid, position, sequence window, difference, logpval, significant)
- Against the 74-protein metformin gold standard, the best grid condition recovers 8/20 kinases and 8/27 other phosphorylated proteins (and 7/17 TFs for the full Step 1)
- Combined output table has exactly the columns UNIPROT, gene_name, mf, final_score, method, with non-kinase/phosphatase PhosphoScore proteins classified as OTHER
- activatory=TRUE and GO_annotation=TRUE were used, and the regulatory-site source files trace back to SIGNOR plus manually downloaded PsP Regulatory_sites/Kinase_Substrates
- Precision, recall and RMSE are computed for every grid condition and the best combination is written to parameter_grid_metrics.csv

## Limitations

- PhosphoSitePlus Regulatory_sites and Kinase_Substrates files must be manually downloaded from the PsP database — no automated fetch
- Prior knowledge coverage is incomplete: regulon and causal interaction resources cover <50% (~9,000 proteins) of the UniProt-SwissProt proteome, so many quantified phosphosites lack regulatory annotation
- The ILP solver path (needed downstream for CARNIVAL) must be manually specified, and the package is under active development so regular re-installation is recommended

## Evidence

- [intro] Step 1 derives the activity of signaling proteins from regulatory phosphosites (PhosphoScore method) and direct transcripts/phosphopeptides using the VIPER algorithm: "Step 1 derives the activity of signaling proteins from regulatory phosphosites (PhosphoScore method) and direct transcripts/phosphopeptides using the VIPER algorithm"
- [methods] get_phosphoscore_info SIGNOR PsP: "reg_phosphosites <- get_phosphoscore_info(resources = c('SIGNOR', 'PsP')"
- [discussion] sole tool for non-kinase/phosphatase activity from phosphoproteomics: "is the sole tool capable of estimating the activity of proteins, aside from kinases and phosphatases, from the phosphoproteomic data"
- [methods] if you want to include PsP information you have to manually download: "if you want to include PsP information you have to manually download from PsP database the Regulatory_sites and Kinase_Substrates file"
- [readme] README pipeline description: [evidence span withheld — anchor status: not_found]
