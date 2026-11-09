---
name: prior-knowledge-network-assembly
description: Use when building or updating the causal prior knowledge network that
  SignalingProfiler's Step 2 (CARNIVAL network reconstruction) consumes, or when assembling
  a custom PKN from SIGNOR/PsP/OmniPath resources.
when_to_use_negative:
- The analysis goal includes metabolomic data integration — SignalingProfiler 2.0's
  PKN-based pipeline (unlike COSMOS) does not include metabolomics
- A task already supplies a ready-made context-specific network — this skill only
  builds the prior knowledge layer, not the data-conditioned network (that is done
  later by preprocess_PKN/CARNIVAL)
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0203
tools:
- name: SignalingProfiler 2.0
  role: Provides create_PKN() and choose_PKN()/preprocess_PKN() routines to assemble,
    serialize, and validate the compliant PKN table
  repo: https://github.com/SaccoPerfettoLab/SignalingProfiler/
- name: SIGNOR
  role: Source of causal (directed, signed) protein interaction records queried during
    PKN assembly
- name: PhosphoSitePlus
  role: Source of kinase-substrate and regulatory-site evidence (Regulatory_sites
    and Kinase_Substrates files, manually downloaded)
- name: Ser/Thr Kinome Atlas
  role: Extended kinase-substrate resource integrated into PKNs via 88/99 percentile
    cutoffs (with_atlas option)
provenance:
  source_task_ids:
  - task_001
  - task_007
  - task_005
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:1f05eeb99fd8c28dfb614e7b8aec50e54448a354ba2efeaf7e802adca7b3976a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SaccoPerfettoLab/SignalingProfiler
  tools:
  - PhosphoSitePlus
  - SIGNOR
  - Ser/Thr Kinome Atlas
  - SignalingProfiler 2.0
  - VIPER
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# prior-knowledge-network-assembly

## Summary

Assembles SignalingProfiler 2.0 prior knowledge networks (PKNs) by querying SIGNOR, PhosphoSitePlus, and the Ser/Thr Kinome Atlas and normalizing every interaction into a binary, directed (regulator-to-target), signed (+1 activatory / -1 inhibitory) table serialized as SignalingProfiler-compliant RDS.

## When to use

Use when building or updating the causal prior knowledge network that SignalingProfiler's Step 2 (CARNIVAL network reconstruction) consumes, or when assembling a custom PKN from SIGNOR/PsP/OmniPath resources.

## When NOT to use

- The analysis goal includes metabolomic data integration — SignalingProfiler 2.0's PKN-based pipeline (unlike COSMOS) does not include metabolomics
- A task already supplies a ready-made context-specific network — this skill only builds the prior knowledge layer, not the data-conditioned network (that is done later by preprocess_PKN/CARNIVAL)

## Inputs

- SIGNOR 3.0 causal interaction records for the chosen organism(s)
- PhosphoSitePlus Regulatory_sites file (manually downloaded)
- PhosphoSitePlus Kinase_Substrates file (manually downloaded)
- Ser/Thr Kinome Atlas kinase-substrate relations
- Organism choice (human or mouse) and direct/indirect regulation flag

## Outputs

- SignalingProfiler-compliant PKN RDS table (e.g. './custom_regulons/custom_PKN.RDS')
- Built-in PKN categories: PKN_human_dir, PKN_human_ind and mouse/atlas counterparts
- A PKN table whose interactions are all binary, directed and signed

## How to apply

Manually download the PhosphoSitePlus Regulatory_sites and Kinase_Substrates files and retrieve causal interaction records from SIGNOR 3.0 for the chosen organism(s). Parse Ser/Thr Kinome Atlas kinase-substrate relations with percentile thresholding, using the 88 percentile cutoff for regulon-level inclusion and the 99 percentile cutoff for PKN edge inclusion. Apply the documented value mappings and filters so every interaction is normalized to a binary, directed (regulator-to-target) and signed (1/-1) form, then run SignalingProfiler's create_PKN() with database = c('SIGNOR', 'PsP') and direct = TRUE (and the indirect transcriptional-regulation variant) for both human and mouse, yielding the built-in categories such as PKN_human_dir, PKN_human_ind and their mouse/atlas counterparts. Serialize each category to './custom_regulons/custom_PKN.RDS' in SignalingProfiler-compliant RDS format.

## Related tools

- **SignalingProfiler 2.0** (Provides create_PKN() and choose_PKN()/preprocess_PKN() routines to assemble, serialize, and validate the compliant PKN table) — https://github.com/SaccoPerfettoLab/SignalingProfiler/
- **SIGNOR** (Source of causal (directed, signed) protein interaction records queried during PKN assembly)
- **PhosphoSitePlus** (Source of kinase-substrate and regulatory-site evidence (Regulatory_sites and Kinase_Substrates files, manually downloaded))
- **Ser/Thr Kinome Atlas** (Extended kinase-substrate resource integrated into PKNs via 88/99 percentile cutoffs (with_atlas option))

## Examples

```
create_PKN(database = c('SIGNOR', 'PsP'), direct = TRUE, organism = 'human')
```

## Evaluation signals

- Load the assembled file via choose_PKN(custom = TRUE, custom_path = ...) and confirm it is accepted as a valid PKN table
- Verify every interaction in the output table is binary, directed (regulator-to-target) and signed (1 activatory / -1 inhibitory)
- Confirm the six filtered PKN categories (PKN_human_dir, PKN_human_ind and their mouse/atlas counterparts) are each serialized to './custom_regulons/custom_PKN.RDS'
- Check Kinome Atlas edges respect the percentile cutoffs: 88 percentile for regulon-level inclusion, 99 percentile for PKN edge inclusion
- After preprocess_PKN against quantified analytes, the restricted PKN should still support two-layer naive network construction (e.g. max_length_1=3, max_length_2=4) in the murine ac220 use case

## Limitations

- PsP Regulatory_sites and Kinase_Substrates files must be manually downloaded from the PsP database before assembly
- Prior knowledge coverage is incomplete: regulon databases and causal interaction resources cover <50% (~9,000 proteins) of the UniProt-SwissProt proteome
- The ILP solver path (needed downstream for CARNIVAL) must be manually specified by the user; SignalingProfiler is under active development and should be reinstalled regularly to exploit new features

## Evidence

- [methods] SignalingProfiler 2.0 provides ready-to-use strategies to query SIGNOR, PhosphoSitePlus, and OmniPath and to assemble custom PKNs in a compliant format.: [evidence span withheld — anchor status: not_found]
- [methods] create_PKN call: [evidence span withheld — anchor status: not_found]
- [methods] PsP manual download requirement: "if you want to include PsP information you have to manually download from PsP database the Regulatory_sites and Kinase_Substrates file"
- [intro] Kinome Atlas integration flag: "Kinome Atlas integration*Boolean, indicating if PKN contains kinase-substrate relations from theSer/Thr Kinome Atlas"
- [intro] PKN built from knowledge databases: "integrates transcriptomics, proteomics, and phosphoproteomics data with the existing knowledge of molecular interactions sourced from databases such as SIGNOR17"
- [discussion] Regular updates of prior knowledge: "we also implemented regular updates of the prior knowledge information from source databases to ensure up-to-date data"
