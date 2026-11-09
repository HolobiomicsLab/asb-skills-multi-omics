---
name: phosphoproteomics-data-preprocessing
description: Use when starting a SignalingProfiler 2.0 analysis from phosphoproteomic
  data and you need to verify or reformat the input table (one row per phosphosite,
  required identifier/differential columns) before footprint-based or PhosphoScore
  inference.
when_to_use_negative:
- When the goal is metabolomic integration — SignalingProfiler 2.0 explicitly does
  not include metabolomic data (unlike COSMOS).
- When input is already a network or pathway annotation (e.g. an existing NDEx network)
  rather than raw phosphosite-level differential data — the pipeline requires phosphoproteomic
  rows as starting point.
- 'When the target system has phosphosite coverage absent from SIGNOR/PsP/Kinome Atlas:
  regulon and causal-interaction resources cover <50% (~9,000 proteins) of the UniProt-SwissProt
  proteome.'
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0157
tools:
- name: SignalingProfiler 2.0
  role: Primary R pipeline whose first step (footprint-based activity estimation and
    PhosphoScore computation) requires the preprocessed phosphoproteomic table; consumes
    UniProt/gene_name/phosphosite columns and significance flags.
  repo: https://github.com/SaccoPerfettoLab/SignalingProfiler/
- name: PhosphoSitePlus
  role: External resource for regulatory phosphosites (Regulatory_sites and Kinase_Substrates
    files) that must be manually downloaded and aligned to the input table before
    phosphoscore_computation.
- name: CARNIVAL
  role: Downstream optimizer that only sees preprocessed data via the prior knowledge
    network; its two-shot ILP run (mipGAP=0.05, timelimit=3600, cplexMemoryLimit=8192)
    presumes the phosphosite table passed upstream validation.
  repo: https://saezlab.github.io/CARNIVAL/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:6b975f5b5c588dd1b2630488c1d80765d988f55644bdef35360a508acea3ce9c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SaccoPerfettoLab/SignalingProfiler
  tools:
  - CARNIVAL
  - PhosphoSitePlus
  - SignalingProfiler 2.0
  - VIPER
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0157
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# phosphoproteomics-data-preprocessing

## Summary

Prepare a processed phosphoproteomic table for SignalingProfiler 2.0 by ensuring each phosphosite occurs only once and carries the required columns (UniProt ID, gene_name, aminoacid, position, sequence window, difference, logpval, significant), since the footprint-based (KSEA/VIPER), PhosphoScore and CARNIVAL steps all consume this table.

## When to use

Use when starting a SignalingProfiler 2.0 analysis from phosphoproteomic data and you need to verify or reformat the input table (one row per phosphosite, required identifier/differential columns) before footprint-based or PhosphoScore inference.

## When NOT to use

- When the goal is metabolomic integration — SignalingProfiler 2.0 explicitly does not include metabolomic data (unlike COSMOS).
- When input is already a network or pathway annotation (e.g. an existing NDEx network) rather than raw phosphosite-level differential data — the pipeline requires phosphoproteomic rows as starting point.
- When the target system has phosphosite coverage absent from SIGNOR/PsP/Kinome Atlas: regulon and causal-interaction resources cover <50% (~9,000 proteins) of the UniProt-SwissProt proteome.

## Inputs

- Processed phosphoproteomic table with one occurrence per phosphosite and columns: UniProt ID, gene_name, aminoacid, position, sequence window, difference, logpval, significant
- Optional: PhosphoSitePlus Regulatory_sites and Kinase_Substrates files (manually downloaded)
- Optional: accompanying proteomics table for correct_proteomics correction

## Outputs

- A valid SignalingProfiler input table consumed by run_footprint_based_analysis and phosphoscore_computation
- Activity estimates for kinases, phosphatases, transcription factors and other proteins inferred via KSEA/VIPER and PhosphoScore

## How to apply

First check that the phosphoproteomic table contains ONLY ONE OCCURRENCE OF EACH PHOSPHOSITE — duplicated phosphosite rows must be collapsed or filtered out, since create_ksea_regulons, create_tfea_regulons and PhosphoScore computation map each row onto kinase/phosphatase and TF regulons. Ensure the table includes the columns UniProt ID, gene_name, aminoacid, position, sequence window, difference, logpval, and significant, because these fields drive the mapping of phosphosites to SIGNOR/PsP regulatory sites and the downstream activity scoring. When PsP support is desired, manually download the Regulatory_sites and Kinase_Substrates files from the PhosphoSitePlus database, as SignalingProfiler does not fetch them automatically. Load the data after installing the package with devtools::install_github('https://github.com/SaccoPerfettoLab/SignalingProfiler/'); correct_proteomics=TRUE in run_footprint_based_analysis can help harmonize proteomics-derived inputs with the phosphosite table. Correctness is judged by verifying the schema (required columns), phosphosite uniqueness, and that the table reproduces the published HeLa EGF benchmark input (56 kinases, 5 phosphatases, 3 TFs and 10 other proteins inferred; 84-node/256-edge network with ~14% phosphorylation-mapped edges).

## Related tools

- **SignalingProfiler 2.0** (Primary R pipeline whose first step (footprint-based activity estimation and PhosphoScore computation) requires the preprocessed phosphoproteomic table; consumes UniProt/gene_name/phosphosite columns and significance flags.) — https://github.com/SaccoPerfettoLab/SignalingProfiler/
- **PhosphoSitePlus** (External resource for regulatory phosphosites (Regulatory_sites and Kinase_Substrates files) that must be manually downloaded and aligned to the input table before phosphoscore_computation.)
- **CARNIVAL** (Downstream optimizer that only sees preprocessed data via the prior knowledge network; its two-shot ILP run (mipGAP=0.05, timelimit=3600, cplexMemoryLimit=8192) presumes the phosphosite table passed upstream validation.) — https://saezlab.github.io/CARNIVAL/

## Examples

```
devtools::install_github('https://github.com/SaccoPerfettoLab/SignalingProfiler/')
```

## Evaluation signals

- Schema check: the phosphoproteomic table contains exactly the required columns — UniProt ID, gene_name, aminoacid, position, sequence window, difference, logpval, significant — with no duplicate phosphosite rows.
- Reproduction benchmark: applied to the Olsen et al. EGF-stimulated HeLa dataset, the pipeline yields a network of 84 nodes and 256 edges, of which ~14% are experimentally detected phosphorylation events.
- Activity-inference sanity: inferred node composition matches the published benchmark (56 kinases, 5 phosphatases, 3 TFs, 10 other proteins).
- Cross-check the resulting network against the deposited EGF network on NDEx and the SIGNOR/PsP prior knowledge annotations.

## Limitations

- Regulon databases and causal interaction resources are incomplete, covering <50% (about 9,000 proteins) of the UniProt-SwissProt proteome.
- If PsP information is desired, the Regulatory_sites and Kinase_Substrates files must be manually downloaded from the PsP database.
- The ILP solver path must be manually specified by the user before running CARNIVAL.
- Epigenetic, acetylomic and ubiquitylomic data cannot be integrated into the signaling analysis (a future challenge).
- Metabolomic data are not supported (unlike COSMOS).

## Evidence

- [results] Processed phosphoproteomic table schema requirement: [evidence span withheld — anchor status: not_found]
- [methods] PsP manual download requirement: "if you want to include PsP information you have to manually download from PsP database the Regulatory_sites and Kinase_Substrates file"
- [discussion] Coverage limitation of prior knowledge resources: "regulon databases and causal interaction resources are incomplete and offer information for <50% (about 9,000 proteins) of the Uniprot-SwissProt proteome"
- [readme] Installation command from README: "devtools::install_github('https://github.com/SaccoPerfettoLab/SignalingProfiler/')"
- [abstract] Pipeline description (input types) from article: "incorporates statistical, footprint-based, and graph algorithms to accelerate the integration and interpretation of multi-omics data"
