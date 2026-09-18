---
name: multi-database-sign-format-compliance
description: Use when you need to build or refresh a SignalingProfiler-compliant PKN
  from public causal-interaction resources (SIGNOR, PsP, OmniPath, Ser/Thr Kinome
  Atlas) for human or mouse contexts.
when_to_use_negative:
- When the input is already a valid SignalingProfiler PKN table — rebuilding would
  only duplicate the built-in objects
- When the analysis requires metabolomic data integration — SignalingProfiler 2.0
  (and its PKN mechanism) does not include metabolites
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3407
tools:
- name: SignalingProfiler 2.0
  role: Provides create_PKN() and ready-to-use strategies to query SIGNOR, PhosphoSitePlus
    and OmniPath and assemble custom PKNs in a compliant format
  repo: https://github.com/SaccoPerfettoLab/SignalingProfiler/
- name: SIGNOR
  role: Source of causal interaction records queried to build the PKN
- name: PhosphoSitePlus
  role: Source of Regulatory_sites and Kinase_Substrates files manually downloaded
    and integrated into the PKN
- name: Ser/Thr Kinome Atlas
  role: Extended background database of kinase-substrate relations included via percentile-thresholded
    edges
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:03d0abd185394f6333305e3aba6ca5ff4b8e39b18d1fe2ea0a215f0e8fce39c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SaccoPerfettoLab/SignalingProfiler
  tools:
  - PhosphoSitePlus
  - SIGNOR
  - Ser/Thr Kinome Atlas
  - SignalingProfiler 2.0
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3407
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# multi-database-sign-format-compliance

## Summary

Assembles custom prior knowledge networks (PKNs) by querying SIGNOR, PhosphoSitePlus, and the Ser/Thr Kinome Atlas and normalizing every interaction to a binary, directed (regulator-to-target), signed (1 activatory / -1 inhibitory) form that SignalingProfiler 2.0 accepts.

## When to use

Use when you need to build or refresh a SignalingProfiler-compliant PKN from public causal-interaction resources (SIGNOR, PsP, OmniPath, Ser/Thr Kinome Atlas) for human or mouse contexts.

## When NOT to use

- When the input is already a valid SignalingProfiler PKN table — rebuilding would only duplicate the built-in objects
- When the analysis requires metabolomic data integration — SignalingProfiler 2.0 (and its PKN mechanism) does not include metabolites

## Inputs

- SIGNOR 3.0 causal interaction records (per organism)
- PhosphoSitePlus Regulatory_sites file (manually downloaded)
- PhosphoSitePlus Kinase_Substrates file (manually downloaded)
- Ser/Thr Kinome Atlas kinase-substrate relations

## Outputs

- SignalingProfiler-compliant PKN table serialized as './custom_regulons/custom_PKN.RDS'
- Six filtered PKN categories (e.g. PKN_human_dir, PKN_human_ind and mouse/atlas counterparts)
- Binary, directed, signed interaction list (1 activatory / -1 inhibitory)

## How to apply

Manually download the PhosphoSitePlus Regulatory_sites and Kinase_Substrates files and retrieve causal interaction records from SIGNOR 3.0 for the chosen organism(s). Parse the Ser/Thr Kinome Atlas kinase-substrate relations with percentile thresholding — the 88 percentile cutoff for regulon-level inclusion and the 99 percentile cutoff for PKN edge inclusion. Apply the documented value mappings so every interaction is normalized to a binary, directed (regulator-to-target), signed form (1 activatory / -1 inhibitory), then run the SignalingProfiler create_PKN() routine with database = c('SIGNOR','PsP'), direct = TRUE (and the indirect transcriptional-regulation variant) for human and mouse, producing the six filtered PKN categories such as PKN_human_dir, PKN_human_ind and their mouse/atlas counterparts. Serialize each category in SignalingProfiler-compliant RDS format to './custom_regulons/custom_PKN.RDS'. Rationale: the CARNIVAL optimization step in SignalingProfiler only retains sign-coherent interactions, so upstream format compliance (binary, directed, signed) is a prerequisite for a valid context-specific network.

## Related tools

- **SignalingProfiler 2.0** (Provides create_PKN() and ready-to-use strategies to query SIGNOR, PhosphoSitePlus and OmniPath and assemble custom PKNs in a compliant format) — https://github.com/SaccoPerfettoLab/SignalingProfiler/
- **SIGNOR** (Source of causal interaction records queried to build the PKN)
- **PhosphoSitePlus** (Source of Regulatory_sites and Kinase_Substrates files manually downloaded and integrated into the PKN)
- **Ser/Thr Kinome Atlas** (Extended background database of kinase-substrate relations included via percentile-thresholded edges)

## Examples

```
PKN <- create_PKN(database = c('SIGNOR', 'PsP'), direct = TRUE, organism = 'human')
```

## Evaluation signals

- Load the assembled file via choose_PKN(custom = TRUE, custom_path = ...) and confirm it is accepted as a valid PKN table
- Verify every interaction in the PKN is binary (no missing/multi-valued signs)
- Verify all edges are directed from regulator to target
- Verify signs are restricted to 1 (activatory) and -1 (inhibitory)
- Confirm Kinome Atlas edges respect the percentile cutoffs (88 for regulons, 99 for PKN edges)

## Limitations

- PsP Regulatory_sites and Kinase_Substrates files must be manually downloaded from the PsP database before building the PKN
- Regulon databases and causal interaction resources are incomplete and cover <50% (about 9,000 proteins) of the UniProt-SwissProt proteome
- Prior knowledge information requires regular updates from source databases to stay current

## Evidence

- [methods] create_PKN: [evidence span withheld — anchor status: not_found]
- [methods] PsP manual download: "if you want to include PsP information you have to manually download from PsP database the Regulatory_sites and Kinase_Substrates file"
- [intro] Kinome Atlas flag: "Kinome Atlas integration*Boolean, indicating if PKN contains kinase-substrate relations from theSer/Thr Kinome Atlas28"
- [abstract] PKN integration abstract: "SignalingProfiler 2.0 derives context-specific signaling networks by integrating proteogenomic data with the prior knowledge-causal network."
- [discussion] regular updates: "we also implemented regular updates of the prior knowledge information from source databases to ensure up-to-date data"
