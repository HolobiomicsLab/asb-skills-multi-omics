---
name: causal-interaction-database-parsing
description: Use when assembling a custom prior knowledge network for SignalingProfiler
  2.0 (or a CARNIVAL-style optimizer) from SIGNOR, PhosphoSitePlus, or Ser/Thr Kinome
  Atlas causal-interaction records in human or mouse.
when_to_use_negative:
- You need metabolomic or epigenetic/acetylomic/ubiquitylomic regulation integrated
  into the network — SignalingProfiler 2.0 PKNs do not include these data types
- You already have a valid SignalingProfiler PKN RDS accepted by choose_PKN — re-parsing
  the source databases would be redundant
- You need comprehensive proteome coverage — regulon and causal-interaction resources
  cover <50% (~9,000 proteins) of the UniProt-SwissProt proteome
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0128
tools:
- name: SignalingProfiler 2.0
  role: Provides create_PKN() to query SIGNOR/PsP and assemble custom PKNs in compliant
    format; choose_PKN(custom = TRUE) validates the assembled file
  repo: https://github.com/SaccoPerfettoLab/SignalingProfiler/
- name: SIGNOR
  role: Source of causal interaction records parsed into the PKN
- name: PhosphoSitePlus
  role: Source of Regulatory_sites and Kinase_Substrates files manually downloaded
    and parsed into the PKN
- name: Ser/Thr Kinome Atlas
  role: Source of kinase-substrate relations integrated with 88/99 percentile thresholds
- name: OmniPath
  role: Optional causal-interaction resource that SignalingProfiler 2.0 can query
    to assemble custom PKNs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:944a6dbb45aa95b736621ac3120dcebbbd90a345373f75e46575fdc130bea5bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SaccoPerfettoLab/SignalingProfiler
  tools:
  - OmniPath
  - PhosphoSitePlus
  - SIGNOR
  - Ser/Thr Kinome Atlas
  - SignalingProfiler 2.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# causal-interaction-database-parsing

## Summary

A method for building SignalingProfiler-compliant prior knowledge networks (PKNs) by parsing public causal-interaction resources (SIGNOR, PhosphoSitePlus, OmniPath, Ser/Thr Kinome Atlas) and normalizing every interaction to binary, directed (regulator-to-target), signed (1/-1) form.

## When to use

Use when assembling a custom prior knowledge network for SignalingProfiler 2.0 (or a CARNIVAL-style optimizer) from SIGNOR, PhosphoSitePlus, or Ser/Thr Kinome Atlas causal-interaction records in human or mouse.

## When NOT to use

- You need metabolomic or epigenetic/acetylomic/ubiquitylomic regulation integrated into the network — SignalingProfiler 2.0 PKNs do not include these data types
- You already have a valid SignalingProfiler PKN RDS accepted by choose_PKN — re-parsing the source databases would be redundant
- You need comprehensive proteome coverage — regulon and causal-interaction resources cover <50% (~9,000 proteins) of the UniProt-SwissProt proteome

## Inputs

- PhosphoSitePlus Regulatory_sites file (manually downloaded)
- PhosphoSitePlus Kinase_Substrates file (manually downloaded)
- SIGNOR 3.0 causal interaction records for the chosen organism(s)
- Ser/Thr Kinome Atlas kinase-substrate relations
- OmniPath interaction records (optional query resource)

## Outputs

- SignalingProfiler-compliant PKN table serialized as './custom_regulons/custom_PKN.RDS' with binary, directed, signed (1/-1) interactions
- Filtered PKN categories corresponding to built-in objects (PKN_human_dir, PKN_human_ind, and mouse/atlas counterparts)

## How to apply

Manually download the PhosphoSitePlus Regulatory_sites and Kinase_Substrates files and retrieve causal interaction records from SIGNOR 3.0 for the chosen organism(s). Parse Ser/Thr Kinome Atlas kinase-substrate relations with percentile thresholding, using the 88 percentile cutoff for regulon-level inclusion and the 99 percentile cutoff for PKN edge inclusion. Apply the documented value mappings and filters so that every interaction is normalized to a binary, directed (regulator-to-target), signed (1 activatory / -1 inhibitory) form. Run the SignalingProfiler create_PKN() routine with database = c('SIGNOR', 'PsP') and direct = TRUE (plus the indirect transcriptional-regulation variant) for human and mouse, producing the filtered PKN categories such as PKN_human_dir, PKN_human_ind and their mouse/atlas counterparts. Serialize each category in SignalingProfiler-compliant RDS format to './custom_regulons/custom_PKN.RDS' and validate by loading it via choose_PKN(custom = TRUE, custom_path = ...).

## Related tools

- **SignalingProfiler 2.0** (Provides create_PKN() to query SIGNOR/PsP and assemble custom PKNs in compliant format; choose_PKN(custom = TRUE) validates the assembled file) — https://github.com/SaccoPerfettoLab/SignalingProfiler/
- **SIGNOR** (Source of causal interaction records parsed into the PKN)
- **PhosphoSitePlus** (Source of Regulatory_sites and Kinase_Substrates files manually downloaded and parsed into the PKN)
- **Ser/Thr Kinome Atlas** (Source of kinase-substrate relations integrated with 88/99 percentile thresholds)
- **OmniPath** (Optional causal-interaction resource that SignalingProfiler 2.0 can query to assemble custom PKNs)

## Examples

```
create_PKN(database = c('SIGNOR', 'PsP'), direct = TRUE, organism = 'human')
```

## Evaluation signals

- choose_PKN(custom = TRUE, custom_path = ...) accepts the assembled file as a valid PKN table
- Every interaction in the assembled PKN is binary, directed (regulator-to-target), and signed (1 activatory / -1 inhibitory)
- Kinome Atlas relations respect the documented thresholds: 88 percentile for regulon-level inclusion and 99 percentile for PKN edge inclusion
- The serialized RDS loads at './custom_regulons/custom_PKN.RDS' and matches one of the six filtered PKN categories (e.g. PKN_human_dir, PKN_human_ind, mouse/atlas counterparts)

## Limitations

- PsP Regulatory_sites and Kinase_Substrates files must be manually downloaded from the PsP database before parsing
- Regulon databases and causal interaction resources are incomplete, offering information for <50% (~9,000 proteins) of the UniProt-SwissProt proteome
- The 'Kinome Atlas integration' flag is Boolean, indicating whether the PKN contains kinase-substrate relations from the Ser/Thr Kinome Atlas — PKNs built without it omit those edges

## Evidence

- [methods] SignalingProfiler 2.0 provides ready-to-use strategies to query SIGNOR, PhosphoSitePlus, and OmniPath and to assemble custom PKNs in a compliant format: [evidence span withheld — anchor status: not_found]
- [methods] PsP files must be manually downloaded: "if you want to include PsP information you have to manually download from PsP database the Regulatory_sites and Kinase_Substrates file"
- [intro] Kinome Atlas integration flag: "Kinome Atlas integration*Boolean, indicating if PKN contains kinase-substrate relations from theSer/Thr Kinome Atlas28"
- [methods] create_PKN invocation example: [evidence span withheld — anchor status: not_found]
- [abstract] PKN sourced from public databases in abstract-level description: "SignalingProfiler 2.0 derives context-specific signaling networks by integrating proteogenomic data with the prior knowledge-causal network."
- [readme] README package installation: "devtools::install_github('https://github.com/SaccoPerfettoLab/SignalingProfiler/')"
