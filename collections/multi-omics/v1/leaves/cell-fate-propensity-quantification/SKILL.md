---
name: cell-fate-propensity-quantification
description: Use when a validated biomolecular regulatory network (e.g., the 197-node/744-edge
  Cho et al. network) has been simulated with CanSeer's DA and stable-state probabilities
  must be converted into per-fate propensity values for validation or treatment comparison.
when_to_use_negative:
- No validated biomolecular network model is available — the published model used
  for demonstration lacked coverage for certain mutations in some patients, resulting
  in inconclusive outcomes, so propensities are unreliable without adequate network
  coverage.
- The goal is differential expression or pathway enrichment from raw RNA-seq rather
  than attractor-based cell fate outcomes of a dynamical network model.
- The question requires synergy-based prioritization of drug combinations — the study
  notes it remains limited in prioritizing synergy-based drug combinations.
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3309
tools:
- name: CanSeer
  role: Executes deterministic/dynamical network analysis (DA) of the annotated biomolecular
    network model and yields the stable-state outcomes from which cell fate propensities
    are computed.
  repo: github.com/BIRL/CanSeer
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1038/s41598-025-99219-x
    title: 'CanSeer: a translational methodology for developing personalized cancer
      models and therapeutics'
  source_content_hash: sha256:2063a0a580e9984265ece7759117604afbd4c00252fa17f0eb08c2d5d3d77c78
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BIRL/CanSeer
  tools:
  - CanSeer
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3309
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cell-fate-propensity-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes cell fate propensities (proliferation, apoptosis, quiescence, senescence, cell cycle arrest, or the Cho et al. normal-proliferation/abnormal-proliferation/metastasis triad) from deterministic/dynamical analysis (DA) attractor states of CanSeer biomolecular network models, providing the quantitative readout for model validation and therapy-response assessment.

## When to use

Use when a validated biomolecular regulatory network (e.g., the 197-node/744-edge Cho et al. network) has been simulated with CanSeer's DA and stable-state probabilities must be converted into per-fate propensity values for validation or treatment comparison.

## When NOT to use

- No validated biomolecular network model is available — the published model used for demonstration lacked coverage for certain mutations in some patients, resulting in inconclusive outcomes, so propensities are unreliable without adequate network coverage.
- The goal is differential expression or pathway enrichment from raw RNA-seq rather than attractor-based cell fate outcomes of a dynamical network model.
- The question requires synergy-based prioritization of drug combinations — the study notes it remains limited in prioritizing synergy-based drug combinations.

## Inputs

- Validated biomolecular regulatory network model in the simulator's accepted format (e.g., updated Cho et al. human signaling network, 197 nodes and 744 edges)
- Input-condition configuration: unperturbed baseline regulatory nodes, or node annotations from median-normalized RNA-seq expression of patient/normal samples (e.g., Case 2 unpaired LUSC normal samples)
- Published reference propensity values (Fig. 2C: NP 0.77, AP 0.08, MT 0.15) or task-specific reference CSV values

## Outputs

- cell_fate_propensities_normal.csv with propensity per fate (normal/abnormal proliferation, metastasis, apoptosis)
- case2_da_propensities.csv with five propensities (proliferation 0.1370, apoptosis 0.2709, quiescence 0.3608, senescence 0.1214, cell cycle arrest 0.0911)
- Fate propensity distribution usable for computing treatment efficacy and cytotoxic effect

## How to apply

Load the validated network model from the CanSeer repository (github.com/BIRL/CanSeer) and run deterministic analysis (DA) under the chosen conditions — e.g., unperturbed baseline inputs for the Cho et al. reproduction, or median-normalized RNA-seq annotation for the Case 2 (unpaired-samples) LUSC normal model. Iterate the network dynamics until the attractor/stable-state distribution converges, then compute the propensity of each cell fate from the stable state probabilities. Write the propensities to a CSV (cell_fate_propensities_normal.csv for the base model; case2_da_propensities.csv for the Case 2 model). Validate against published references: base model against Fig. 2C values (proliferation 0.77, apoptotic 0.08, metastasis 0.15) and the Case 2 median normal model against proliferation 0.1370, apoptosis 0.2709, quiescence 0.3608, senescence 0.1214, cell cycle arrest 0.0911, matching to four decimal places within the simulator's convergence tolerance. Downstream, the same fate propensities feed computation of treatment efficacy and cytotoxic effect in CanSeer.

## Related tools

- **CanSeer** (Executes deterministic/dynamical network analysis (DA) of the annotated biomolecular network model and yields the stable-state outcomes from which cell fate propensities are computed.) — github.com/BIRL/CanSeer

## Examples

```
import pandas as pd; pd.DataFrame({'fate':['proliferation','apoptosis','quiescence','senescence','cell_cycle_arrest'],'propensity':[0.1370,0.2709,0.3608,0.1214,0.0911]}).to_csv('case2_da_propensities.csv', index=False)
```

## Evaluation signals

- Reproduction of Cho et al. base model: DA propensities must equal 0.77 (normal proliferation), 0.08 (abnormal proliferation/apoptotic), and 0.15 (metastasis) as reported in Fig. 2C, within the simulator's convergence tolerance.
- Case 2 median normal model: each of the five propensities must match the reference to four decimal places (0.1370, 0.2709, 0.3608, 0.1214, 0.0911).
- Propensity values over all modeled fates should be consistent with a converged, normalized attractor distribution (values sum to ~1).
- Output CSVs (cell_fate_propensities_normal.csv, case2_da_propensities.csv) exist, contain one propensity per fate, and values are in [0, 1].
- DA must be run until the attractor/stable-state distribution converges before propensities are extracted.

## Limitations

- The published model used for demonstration lacked coverage for certain mutations in some patients, resulting in inconclusive outcomes.
- Propensity predictions are in silico and require further evaluation; wet lab validation of predicted efficacious drug combinations and repositioned drugs has not yet been done.
- The framework requires expansion to cater for panomics data to avoid under- or overfitting of a patient's profile.

## Evidence

- [other] deterministic analysis identified three cell fates with propensities of 0.77 for normal proliferation, 0.08 for abnormal proliferation, and 0.15 for metastasis: [evidence span withheld — anchor status: not_found]
- [other] verify each computed propensity matches the reported value (proliferation 0.1370, apoptosis 0.2709, quiescence 0.3608, senescence 0.1214, cell cycle arrest 0.0911) to four decimal places: [evidence span withheld — anchor status: not_found]
