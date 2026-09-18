---
name: simulation-method-accuracy-score-wrangling
description: Use when a benchmark table contains paired per-method accuracy scores
  for two transcriptomics modalities (SRT vs scRNA-seq) and you need to test whether
  method rankings or scores transfer between them.
when_to_use_negative:
- 'The score table lacks paired observations: methods were scored on only one modality,
  so no within-method SRT-vs-scRNA-seq pairing exists after dropping missing values.'
- The goal is to compare simulation methods on a single modality (e.g., ranking accuracy
  on scRNA-seq only) rather than testing cross-modality score agreement.
- The scores are non-numeric or ordinal-only rankings where a Pearson product-moment
  correlation on raw scores is inappropriate.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_3170
tools:
- name: simmethods
  role: Collection of 49 simulation methods for single-cell and spatial transcriptome
    data whose per-method accuracy scores feed the cross-modality correlation
  repo: https://github.com/duohongrui/simmethods
- name: simpipe
  role: Standard pipeline for estimating parameters from real datasets, simulating
    single-cell RNA-seq datasets, and evaluating simulated data from general and functional
    perspectives; source of accuracy_scores.csv
  repo: https://github.com/duohongrui/simpipe
- name: SRTsim
  role: One of the best-accuracy simulation methods contributing paired scores on
    SRT reference datasets
- name: scDesign3
  role: Top-accuracy simulator scored on both SRT and scRNA-seq reference datasets
    in the correlation
- name: ZINB-WaVE
  role: Top-accuracy simulator contributing paired per-method scores across modalities
- name: scDesign2
  role: Top-accuracy simulator contributing paired per-method scores across modalities
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:c5ccc70cecd6b1d0aef7fdf5a493f1a00ae8cde83af35143cc3f48aa0e52a80f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duohongrui/simmethods
  tools:
  - SRTsim
  - ZINB-WaVE
  - scDesign2
  - scDesign3
  - simmethods
  - simpipe
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# simulation-method-accuracy-score-wrangling

## Summary

Wrangles per-method accuracy scores computed separately on SRT and scRNA-seq reference datasets and quantifies their cross-modality agreement via Pearson correlation, showing that methods accurate on scRNA-seq data tend to be accurate on SRT data (r ≈ 0.92, Fig. 4b).

## When to use

Use when a benchmark table contains paired per-method accuracy scores for two transcriptomics modalities (SRT vs scRNA-seq) and you need to test whether method rankings or scores transfer between them.

## When NOT to use

- The score table lacks paired observations: methods were scored on only one modality, so no within-method SRT-vs-scRNA-seq pairing exists after dropping missing values.
- The goal is to compare simulation methods on a single modality (e.g., ranking accuracy on scRNA-seq only) rather than testing cross-modality score agreement.
- The scores are non-numeric or ordinal-only rankings where a Pearson product-moment correlation on raw scores is inappropriate.

## Inputs

- accuracy_scores.csv with per-method accuracy scores computed separately on SRT reference datasets and scRNA-seq reference datasets (one row per simulation method)
- list of simulation methods evaluated (e.g., SRTsim, scDesign3, ZINB-WaVE, scDesign2, Lun, SPARSim, scDesign3-tree)

## Outputs

- correlation_srt_scrnaseq record containing pearson_r, p-value, and the number of methods contributing to the correlation
- paired scatter of SRT vs scRNA-seq per-method accuracy scores (cf. Fig. 4b)

## How to apply

Load the accuracy_scores.csv produced by the benchmarking pipeline (task_001), which stores one row per simulation method with score columns computed on SRT reference datasets and on scRNA-seq reference datasets. Extract the two paired score columns and drop methods with missing values in either modality so only methods runnable on both data types contribute paired observations. Compute the Pearson correlation coefficient and p-value between the two score vectors with scipy.stats.pearsonr (or R cor.test), and record the number of contributing methods (out of the 49 simulation methods evaluated on 152 reference datasets from 24 platforms). Verify the result reproduces the reported Pearson's r of approximately 0.92 (Fig. 4b); a high r supports the paper's conclusion that methods tailored to scRNA-seq data have potential compatibility for simulating SRT data.

## Related tools

- **simmethods** (Collection of 49 simulation methods for single-cell and spatial transcriptome data whose per-method accuracy scores feed the cross-modality correlation) — https://github.com/duohongrui/simmethods
- **simpipe** (Standard pipeline for estimating parameters from real datasets, simulating single-cell RNA-seq datasets, and evaluating simulated data from general and functional perspectives; source of accuracy_scores.csv) — https://github.com/duohongrui/simpipe
- **SRTsim** (One of the best-accuracy simulation methods contributing paired scores on SRT reference datasets)
- **scDesign3** (Top-accuracy simulator scored on both SRT and scRNA-seq reference datasets in the correlation)
- **ZINB-WaVE** (Top-accuracy simulator contributing paired per-method scores across modalities)
- **scDesign2** (Top-accuracy simulator contributing paired per-method scores across modalities)

## Examples

```
from scipy.stats import pearsonr; df = pd.read_csv('accuracy_scores.csv').dropna(subset=['srt_score','scrnaseq_score']); pearson_r, p = pearsonr(df['srt_score'], df['scrnaseq_score'])
```

## Evaluation signals

- Recomputed Pearson r on the paired score columns reproduces the reported value of approximately 0.92.
- The method count written to correlation_srt_scrnaseq equals the number of rows with non-missing scores in both the SRT and scRNA-seq columns.
- Both score vectors are paired per-method (same method ID order) after dropping methods with missing values in either modality.
- The p-value reported alongside pearson_r corresponds to scipy.stats.pearsonr (or R cor.test) output on the same paired vectors.
- Result is consistent with the paper's finding that SRTsim, scDesign3, ZINB-WaVE, and scDesign2 have the best accuracy performance across various platforms.

## Limitations

- Only methods runnable on both SRT and scRNA-seq data contribute paired observations; methods with missing scores in one modality are dropped and cannot inform the correlation.
- A high correlation does not mean every method transfers equally — the paper stresses users should consider trade-offs between method accuracy and scalability (or functionality) when making decisions.
- No method performs best on all criteria, so accuracy-score correlation alone should not drive method selection.

## Evidence

- [results] Per-method accuracy scores on SRT and scRNA-seq reference datasets are highly correlated, with Pearson's r = 0.92 (Fig. 4b).: [evidence span withheld — anchor status: not_found]
- [abstract] some methods tailored to scRNA-seq data have potential compatibility for simulating SRT data: "some methods tailored to scRNA-seq data have potential compatibility for simulating SRT data"
- [abstract] We systematically evaluated 49 simulation methods developed for scRNA-seq and/or SRT data in terms of accuracy, functionality, scalability, and usability using 152 reference datasets derived from 24: "We systematically evaluated 49 simulation methods developed for scRNA-seq and/or SRT data in terms of accuracy, functionality, scalability, and usability using 152 reference datasets derived from 24"
- [abstract] SRTsim, scDesign3, ZINB-WaVE, and scDesign2 have the best accuracy performance across various platforms: "SRTsim, scDesign3, ZINB-WaVE, and scDesign2 have the best accuracy performance across various platforms"
- [readme] The goal of simpipe is to establish a standard pipeline to estimating parameters from real datasets, simulating single-cell RNA-seq datasets and evaluating the simulated data from general and functional perspectives.: "The goal of simpipe is to establish a standard pipeline to estimating parameters from real datasets, simulating single-cell RNA-seq datasets and evaluating the simulated data from general and"
