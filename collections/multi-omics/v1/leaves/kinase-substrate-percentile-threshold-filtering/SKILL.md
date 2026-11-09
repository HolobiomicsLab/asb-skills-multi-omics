---
name: kinase-substrate-percentile-threshold-filtering
description: Use when building a prior knowledge network for SignalingProfiler 2.0
  that must include kinase–substrate edges from the Ser/Thr Kinome Atlas, and you
  need to decide which atlas relations are stringent enough for PKN edges versus regulon-level
  inclusion.
when_to_use_negative:
- Your PKN should contain only curated direct causal interactions from SIGNOR/PsP
  and you do not need the extended Ser/Thr Kinome Atlas kinase–substrstrate layer
- You need metabolomic data integration — SignalingProfiler 2.0 explicitly does not
  include metabolomic data compared to tools such as COSMOS
- The kinase–substrate relations already come pre-filtered as a valid SignalingProfiler
  PKN table (binary, directed, signed), so re-thresholding would discard edges
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SignalingProfiler 2.0
  role: Consumes the percentile-filtered, normalized interactions via create_PKN()
    and provides the compliant RDS PKN format validated with choose_PKN(custom = TRUE,
    custom_path = ...)
  repo: https://github.com/SaccoPerfettoLab/SignalingProfiler/
- name: Ser/Thr Kinome Atlas
  role: Source of kinase–substrate relations that are ranked and percentile-thresholded
    (88/99 cutoffs) for regulon and PKN inclusion
- name: PhosphoSitePlus
  role: Complementary source of Regulatory_sites and Kinase_Substrates records merged
    with atlas-derived edges into the PKN
- name: SIGNOR
  role: Causal interaction database queried alongside the atlas-filtered edges when
    assembling the PKN with create_PKN(database = c('SIGNOR', 'PsP'))
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:011ff1efa556080d5bff3f330104711b0515036f27af3cf262f3340121044902
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SaccoPerfettoLab/SignalingProfiler
  tools:
  - PhosphoSitePlus
  - SIGNOR
  - Ser/Thr Kinome Atlas
  - SignalingProfiler 2.0
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# kinase-substrate-percentile-threshold-filtering

## Summary

Filters kinase–substrate relations from the Ser/Thr Kinome Atlas by percentile cutoffs (88th percentile for regulon-level inclusion, 99th percentile for PKN edge inclusion), then normalizes accepted interactions to a binary, directed (regulator-to-target), signed (1 activatory / -1 inhibitory) form so they can be assembled into SignalingProfiler-compliant PKNs alongside SIGNOR and PhosphoSitePlus records.

## When to use

Use when building a prior knowledge network for SignalingProfiler 2.0 that must include kinase–substrate edges from the Ser/Thr Kinome Atlas, and you need to decide which atlas relations are stringent enough for PKN edges versus regulon-level inclusion.

## When NOT to use

- Your PKN should contain only curated direct causal interactions from SIGNOR/PsP and you do not need the extended Ser/Thr Kinome Atlas kinase–substrstrate layer
- You need metabolomic data integration — SignalingProfiler 2.0 explicitly does not include metabolomic data compared to tools such as COSMOS
- The kinase–substrate relations already come pre-filtered as a valid SignalingProfiler PKN table (binary, directed, signed), so re-thresholding would discard edges

## Inputs

- Ser/Thr Kinome Atlas kinase–substrate relation table (with association scores for percentile ranking)
- PhosphoSitePlus Regulatory_sites and Kinase_Substrates files (manually downloaded)
- SIGNOR 3.0 causal interaction records for the chosen organism(s) (human, mouse)

## Outputs

- Percentile-filtered kinase–substrate edge list (88th percentile regulon set; 99th percentile PKN edge set)
- Normalized binary, directed, signed interaction table (regulator-to-target; 1/-1 sign)
- SignalingProfiler-compliant PKN serialized to './custom_regulons/custom_PKN.RDS'

## How to apply

Retrieve the Ser/Thr Kinome Atlas kinase–substrate relations and parse them alongside manually downloaded PhosphoSitePlus Regulatory_sites/Kinase_Substrates files and SIGNOR 3.0 causal interaction records for the chosen organism (human and/or mouse). Apply two percentile thresholds to the atlas relations: keep relations at or above the 88 percentile cutoff for regulon-level inclusion, and keep only those at or above the 99 percentile cutoff as candidate PKN edges, so that only the most confident kinase–substrate events enter the network backbone. Apply the documented value mappings and filters so every accepted interaction is normalized to a binary, directed (regulator-to-target), signed (1 activatory / -1 inhibitory) form. Feed the normalized tables into the SignalingProfiler create_PKN() routine (e.g. database = c('SIGNOR', 'PsP'), direct = TRUE) to assemble the filtered PKN categories (direct and indirect/transcriptional variants) for each organism, then serialize each category to './custom_regulons/custom_PKN.RDS' in SignalingProfiler-compliant RDS format. The two-tier cutoff rationale is that regulon definitions can tolerate somewhat weaker evidence than the causal network itself, mirroring the paper's use of extended background databases (Ser/Thr Kinome Atlas, CollectTRI) on top of SIGNOR and PsP.

## Related tools

- **SignalingProfiler 2.0** (Consumes the percentile-filtered, normalized interactions via create_PKN() and provides the compliant RDS PKN format validated with choose_PKN(custom = TRUE, custom_path = ...)) — https://github.com/SaccoPerfettoLab/SignalingProfiler/
- **Ser/Thr Kinome Atlas** (Source of kinase–substrate relations that are ranked and percentile-thresholded (88/99 cutoffs) for regulon and PKN inclusion)
- **PhosphoSitePlus** (Complementary source of Regulatory_sites and Kinase_Substrates records merged with atlas-derived edges into the PKN)
- **SIGNOR** (Causal interaction database queried alongside the atlas-filtered edges when assembling the PKN with create_PKN(database = c('SIGNOR', 'PsP')))

## Examples

```
create_PKN(database = c('SIGNOR', 'PsP'), direct = TRUE, organism = 'human')
```

## Evaluation signals

- All interactions in the assembled PKN table are binary, directed (regulator-to-target) and signed (1 activatory / -1 inhibitory)
- Atlas-derived PKN edges all sit at or above the 99 percentile cutoff; the regulon-level set includes relations down to the 88 percentile cutoff
- The serialized file loads via choose_PKN(custom = TRUE, custom_path = ...) and is accepted as a valid PKN table
- Each assembled category corresponds to one of the six filtered PKN categories (e.g. PKN_human_dir, PKN_human_ind and their mouse/atlas counterparts)
- The 'Kinome Atlas integration' flag on the resulting PKN indicates kinase–substrate relations from the Ser/Thr Kinome Atlas are present

## Limitations

- PsP Regulatory_sites and Kinase_Substrates files must be manually downloaded before merging with atlas-derived edges
- Regulon and causal interaction resources are incomplete, covering <50% (about 9,000 proteins) of the UniProt-SwissProt proteome
- The 88/99 percentile cutoffs are specific to regulon- versus PKN-level inclusion in SignalingProfiler 2.0; different downstream tools may require recalibration
- SignalingProfiler is under active development; the README suggests deleting and re-installing the package regularly to exploit new features

## Evidence

- [other] percentile cutoffs: [evidence span withheld — anchor status: not_found]
- [intro] Kinome Atlas integration flag: "Kinome Atlas integration*Boolean, indicating if PKN contains kinase-substrate relations from theSer/Thr Kinome Atlas28"
- [discussion] extended background databases: "extended background databases, such as CollectTRI28 and Ser/Thr Kinome Atlas27,29"
- [methods] PKN assembly call: [evidence span withheld — anchor status: not_found]
- [methods] manual PsP download: "if you want to include PsP information you have to manually download from PsP database the Regulatory_sites and Kinase_Substrates file"
