---
name: pearson-correlation-across-benchmark-modalities
description: Use when a benchmark table contains paired per-method performance scores
  computed on two data modalities (e.g., SRT vs scRNA-seq reference datasets) and
  the goal is to test cross-modality agreement of method rankings via Pearson correlation.
when_to_use_negative:
- Scores are not paired per method across the two modalities (e.g., methods evaluated
  only on one modality), since dropping unpaired observations would leave too few
  points for a meaningful Pearson correlation
- The question is about ranking agreement beyond linear association — a Pearson r
  of accuracy scores does not address per-scenario performance differences such as
  Lun, SPARSim, and scDesign3-tree outperforming other methods under specific simulation
  scenarios
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3308
tools:
- name: simpipe
  role: Standard pipeline that estimated parameters from real reference datasets,
    simulated data, and produced the per-method accuracy scores on SRT and scRNA-seq
    reference datasets that feed this correlation
  repo: https://github.com/duohongrui/simpipe
- name: simmethods
  role: Collection of the 49 simulation methods whose accuracy scores on the two modalities
    are being correlated
  repo: https://github.com/duohongrui/simmethods
- name: SRTsim
  role: One of the best-performing simulation methods contributing SRT-based accuracy
    scores to the correlation
- name: scDesign3
  role: Top-accuracy method evaluated on both SRT and scRNA-seq reference datasets,
    contributing paired scores
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:e3dd18739f7c992211f6e38bb1fb16963f7bf2d18e438d7ba7d1a0cdcb698542
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duohongrui/simpipe
  tools:
  - SRTsim
  - ZINB-WaVE
  - scDesign2
  - scDesign3
  - simmethods
  - simpipe
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# pearson-correlation-across-benchmark-modalities

## Summary

Quantifies how strongly per-method accuracy scores benchmarked on SRT (spatially resolved transcriptomics) reference datasets correlate with scores benchmarked on scRNA-seq reference datasets, using a Pearson correlation over paired per-method scores (reported r ≈ 0.92, Fig. 4b).

## When to use

Use when a benchmark table contains paired per-method performance scores computed on two data modalities (e.g., SRT vs scRNA-seq reference datasets) and the goal is to test cross-modality agreement of method rankings via Pearson correlation.

## When NOT to use

- Scores are not paired per method across the two modalities (e.g., methods evaluated only on one modality), since dropping unpaired observations would leave too few points for a meaningful Pearson correlation
- The question is about ranking agreement beyond linear association — a Pearson r of accuracy scores does not address per-scenario performance differences such as Lun, SPARSim, and scDesign3-tree outperforming other methods under specific simulation scenarios

## Inputs

- accuracy_scores.csv: per-method accuracy scores computed separately on SRT and scRNA-seq reference datasets (paired columns, one row per simulation method)
- List of evaluated simulation methods (up to 49 methods from the simmethods collection, e.g., SRTsim, scDesign3, ZINB-WaVE, scDesign2, Lun, SPARSim)

## Outputs

- pearson_r: Pearson correlation coefficient between SRT-based and scRNA-seq-based per-method accuracy scores (expected ≈ 0.92)
- p-value of the Pearson correlation from scipy.stats.pearsonr or R cor.test
- method count: number of methods with complete paired scores contributing to the correlation
- correlation_srt_scrnaseq result record (e.g., correlation_srt_scrnaseq.csv/JSON)

## How to apply

Load the per-method accuracy score table (accuracy_scores.csv, produced by the simpipe benchmark of 49 simulation methods on 152 reference datasets from 24 platforms) with one column of SRT-based accuracy scores and one column of scRNA-seq-based accuracy scores. Pair observations per simulation method (SRTsim, scDesign3, ZINB-WaVE, scDesign2, SPARSim, Lun, etc.) and drop any method with a missing value in either modality so only completely paired methods contribute. Compute the Pearson correlation coefficient and p-value between the two score vectors with scipy.stats.pearsonr (Python) or cor.test (R), and record the number of paired methods alongside pearson_r and the p-value. The article motivates this test because 'some methods tailored to scRNA-seq data have potential compatibility for simulating SRT data', so a high r (reported ≈ 0.92) indicates scRNA-seq benchmark results transfer to SRT benchmarking. Correct application is verified by reproducing pearson_r ≈ 0.92 as reported in Fig. 4b of the Genome Biology benchmark.

## Related tools

- **simpipe** (Standard pipeline that estimated parameters from real reference datasets, simulated data, and produced the per-method accuracy scores on SRT and scRNA-seq reference datasets that feed this correlation) — https://github.com/duohongrui/simpipe
- **simmethods** (Collection of the 49 simulation methods whose accuracy scores on the two modalities are being correlated) — https://github.com/duohongrui/simmethods
- **SRTsim** (One of the best-performing simulation methods contributing SRT-based accuracy scores to the correlation)
- **scDesign3** (Top-accuracy method evaluated on both SRT and scRNA-seq reference datasets, contributing paired scores)

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(df['srt_accuracy'].dropna(), df['scrnaseq_accuracy'].dropna())
```

## Evaluation signals

- Pearson r reproduces the reported value of approximately 0.92 (Fig. 4b)
- Method count matches the number of simulation methods with non-missing scores in both modality columns after pairwise deletion
- Both score vectors are finite and paired 1:1 per method before calling pearsonr (no NaN in either column)
- Result record contains all three required fields: pearson_r, p-value, and method count
- p-value is consistent with r ≈ 0.92 given the number of paired methods (strongly significant)

## Limitations

- Methods with missing accuracy scores in either modality must be dropped, so the correlation may cover fewer than all 49 evaluated simulation methods
- A high Pearson r (≈ 0.92) reflects linear agreement of aggregate accuracy scores, not that every method transfers — the article notes trade-offs between accuracy, scalability, and functionality, and that Phenopath, Lun, Simple, and MFA yield high scalability scores but cannot generate realistic simulated data
- No method performs best on all criteria, so cross-modality correlation of overall accuracy should not be read as per-scenario or per-criterion equivalence

## Evidence

- [abstract] Cross-modality correlation motivation: "some methods tailored to scRNA-seq data have potential compatibility for simulating SRT data"
- [abstract] Top methods by accuracy: "SRTsim, scDesign3, ZINB-WaVE, and scDesign2 have the best accuracy performance across various platforms"
- [abstract] Scope of benchmark underlying the paired scores: "We systematically evaluated 49 simulation methods developed for scRNA-seq and/or SRT data in terms of accuracy, functionality, scalability, and usability using 152 reference datasets"
- [readme] Pipeline provenance of accuracy scores: "Simpipe provides the standard pipeline of estimating parameters from read data, simulating and evaluating simulated data."
