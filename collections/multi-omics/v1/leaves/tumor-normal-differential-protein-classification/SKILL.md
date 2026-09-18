---
name: tumor-normal-differential-protein-classification
description: Use when you have Protrec2 per-sample recovery probabilities from matched
  tumor-normal pairs and need to binarize, classify differentially recovered proteins,
  and independently validate them in large clinical cohorts like CPTAC.
when_to_use_negative:
- Input is a single-condition proteome (e.g. HeLa or A549 benchmarking) with no matched
  tumor-normal pairs — the differential classification and direction-consistency checks
  are not applicable
- You need protein abundance, modification status, or subcellular localization inference
  — Protrec2 operates on a binary presence/absence basis
- Validation cohort lacks proteomic coverage of the tissue of interest, since reported
  recovery rates are lower bounds constrained by dataset coverage
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0121
tools:
- name: Protrec2
  role: Recovers missing proteins per sample via tissue-specific Bayesian inference;
    its per-sample probability CSVs are the basis for tumor/normal classification
    and CPTAC validation
  repo: https://github.com/miaomiao6606/Protrec2
- name: Python
  role: Runs the main.py pipeline and performs post-hoc binarization, class assignment,
    and CPTAC validation computations
- name: pip
  role: Installs Protrec2 dependencies via requirements.txt before running the recovery
    pipeline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:0b0c80033152f554d22e243be0ab8f74a5cedf67ea3f4c0fc1cb3c043fff4304
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/miaomiao6606/Protrec2
  repo_url: https://github.com/miaomiao6606/Protrec2
  tools:
  - Protrec2
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# tumor-normal-differential-protein-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies Protrec2-recovered missing proteins from matched tumor-normal pairs into cancer-specific and normal-exclusive groups, then validates them against independent CPTAC lung cancer proteomic cohorts (PDC000234 and PDC000153) and checks expression direction consistency.

## When to use

Use when you have Protrec2 per-sample recovery probabilities from matched tumor-normal pairs and need to binarize, classify differentially recovered proteins, and independently validate them in large clinical cohorts like CPTAC.

## When NOT to use

- Input is a single-condition proteome (e.g. HeLa or A549 benchmarking) with no matched tumor-normal pairs — the differential classification and direction-consistency checks are not applicable
- You need protein abundance, modification status, or subcellular localization inference — Protrec2 operates on a binary presence/absence basis
- Validation cohort lacks proteomic coverage of the tissue of interest, since reported recovery rates are lower bounds constrained by dataset coverage

## Inputs

- PXD000603 expression matrix (CSV, rows as proteins with UniProt ID or symbol, columns as six matched lung tumor-normal sample pairs)
- Protrec2 per-sample output CSVs with columns Protein and Probability
- Independent CPTAC lung cancer proteomic datasets PDC000234 and PDC000153
- Tissue-specific complex confidence file updated_tissue_complex_lung.csv (built-in)

## Outputs

- Cancer-specific recovered protein list (recovered in tumors only)
- Normal-exclusive recovered protein list (recovered only in normals)
- CPTAC validation support rates per class (target 87.18% cancer-specific, 85.45% normal-exclusive)
- Expression direction consistency rates (68.5% upregulated cancer-specific, 90.0% downregulated normal-exclusive)

## How to apply

Run Protrec2 (python main.py --tissue lung --fdr 0.01 --threshold 5) on the PXD000603 expression matrix of six matched lung tumor-normal pairs, using Bayesian inference that integrates prior detection probability, cocomplex structural support, and lung tissue-specific complex confidence from updated_tissue_complex_lung.csv. Collect per-sample CSV outputs (Protein,Probability rows), then binarize recovered proteins at probability cutoffs in the 0.50–0.95 range. Classify proteins recovered in tumors as cancer-specific and those recovered only in normals as normal-exclusive. Validate each class against the independent CPTAC cohorts PDC000234 and PDC000153 by computing the fraction of recovered proteins observed there, and compare tumor-vs-normal expression directions for consistency. Accept the run when CPTAC support rates reach ≈87.18% (cancer-specific) and ≈85.45% (normal-exclusive), with cancer-specific proteins predominantly upregulated (≈68.5%) and normal-exclusive proteins predominantly downregulated (≈90.0%).

## Related tools

- **Protrec2** (Recovers missing proteins per sample via tissue-specific Bayesian inference; its per-sample probability CSVs are the basis for tumor/normal classification and CPTAC validation) — https://github.com/miaomiao6606/Protrec2
- **Python** (Runs the main.py pipeline and performs post-hoc binarization, class assignment, and CPTAC validation computations)
- **pip** (Installs Protrec2 dependencies via requirements.txt before running the recovery pipeline)

## Examples

```
python main.py --expression example/example_expression.csv --output_dir results/ --tissue lung --fdr 0.01 --threshold 5
```

## Evaluation signals

- CPTAC support rate for cancer-specific recovered proteins ≈ 87.18% and for normal-exclusive ≈ 85.45% (accept within tolerance)
- At least 85% of predicted proteins are supported by CPTAC cohorts PDC000234 and PDC000153
- ≈68.5% of cancer-specific recovered proteins are upregulated in tumors; ≈90.0% of normal-exclusive proteins are downregulated
- Per-sample output files follow the PROTREC2_result_{sample_name}.csv schema with Protein and Probability columns and probabilities in 0–1
- Binarization is monotonic across the 0.50–0.95 cutoff range, consistent with the reported threshold sensitivity behavior

## Limitations

- Protrec2 operates on a binary presence/absence basis and does not incorporate protein abundance, modification status, or subcellular localization
- Tissue-specific annotations were derived from population-level data and may not reflect interpatient heterogeneity
- Validation performance is constrained by proteomics dataset coverage; reported recovery rates are lower bounds on true performance
- The protein complex repository is incomplete for novel or condition-specific complexes

## Evidence

- [readme] README output format description for per-sample results: [evidence span withheld — anchor status: external_doc]
- [readme] README performance summary against CPTAC: [evidence span withheld — anchor status: external_doc]
