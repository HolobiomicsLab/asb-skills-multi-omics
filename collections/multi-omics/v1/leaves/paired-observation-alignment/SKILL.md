---
name: paired-observation-alignment
description: Use when two score vectors cover the same methods/entities measured under
  different conditions or modalities and you must drop methods missing in either modality
  before computing a paired correlation.
when_to_use_negative:
- The two score sets come from different methods/entities, so observations cannot
  be paired on a shared method identifier — an unpaired ecological correlation would
  be the wrong analysis.
- The analysis goal is per-dataset or per-scenario performance ranking (e.g. which
  of SRTsim, scDesign3, ZINB-WaVE, scDesign2 is best on one platform) rather than
  cross-modality agreement.
- Scores are ordinal or non-normal rank summaries where Spearman correlation, not
  Pearson, is appropriate.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: scipy.stats.pearsonr
  role: Computes the Pearson correlation coefficient and p-value between the paired
    SRT and scRNA-seq accuracy score vectors
- name: simmethods
  role: Collection of the 49 simulation methods whose per-method accuracy scores form
    the paired observations being aligned
  repo: https://github.com/duohongrui/simmethods
- name: simpipe
  role: Standard pipeline used to estimate parameters, simulate data, and evaluate
    simulated data from which the accuracy scores originate
  repo: https://github.com/duohongrui/simpipe
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1186/s13059-024-03290-y
    title: Systematic evaluation with practical guidelines for single-cell and spatially
      resolved transcriptomics data simulation under multiple scenarios
  source_content_hash: sha256:335a3a84dfdd0efb5775e903803563a4e4b588b1fb0537728916126118bdd921
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/duohongrui/simmethods
  tools:
  - SRTsim
  - ZINB-WaVE
  - scDesign2
  - scDesign3
  - scipy.stats.pearsonr
  - simmethods
  - simpipe
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Paired observation alignment

## Summary

Aligns per-method accuracy scores computed independently on two reference modalities (SRT vs scRNA-seq datasets) into paired observations so a cross-modality correlation (Pearson r) is computed on matched methods only.

## When to use

Use when two score vectors cover the same methods/entities measured under different conditions or modalities and you must drop methods missing in either modality before computing a paired correlation.

## When NOT to use

- The two score sets come from different methods/entities, so observations cannot be paired on a shared method identifier — an unpaired ecological correlation would be the wrong analysis.
- The analysis goal is per-dataset or per-scenario performance ranking (e.g. which of SRTsim, scDesign3, ZINB-WaVE, scDesign2 is best on one platform) rather than cross-modality agreement.
- Scores are ordinal or non-normal rank summaries where Spearman correlation, not Pearson, is appropriate.

## Inputs

- accuracy_scores.csv with per-method accuracy score columns computed on SRT reference datasets
- accuracy_scores.csv with per-method accuracy score columns computed on scRNA-seq reference datasets
- Method identifier column shared by both modality score sets

## Outputs

- Pearson r between SRT-based and scRNA-seq-based per-method accuracy scores
- p-value of the correlation
- Count of methods contributing paired observations
- correlation_srt_scrnaseq result record (expected pearson_r ≈ 0.92)

## How to apply

Load the per-method accuracy table (e.g. task_001's accuracy_scores.csv) that contains accuracy scores computed separately on SRT and scRNA-seq reference datasets. Extract the paired per-technique score columns (SRT-based scores vs scRNA-seq-based scores) and drop methods with missing values in either modality so observations are strictly paired. Compute the Pearson correlation coefficient and p-value on the two aligned score vectors with scipy.stats.pearsonr (or R cor.test), and record the number of methods contributing to the correlation since a small paired n changes interpretability. Verify the result against the reported benchmark value (Pearson's r = 0.92 in Fig. 4b of the Genome Biology simulation benchmark). Rationale: pairing on method identity avoids spurious correlation driven by unbalanced method sets across the 49 evaluated simulators.

## Related tools

- **scipy.stats.pearsonr** (Computes the Pearson correlation coefficient and p-value between the paired SRT and scRNA-seq accuracy score vectors)
- **simmethods** (Collection of the 49 simulation methods whose per-method accuracy scores form the paired observations being aligned) — https://github.com/duohongrui/simmethods
- **simpipe** (Standard pipeline used to estimate parameters, simulate data, and evaluate simulated data from which the accuracy scores originate) — https://github.com/duohongrui/simpipe

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(df['srt_score'].dropna(), df.loc[df['srt_score'].notna(), 'scrnaseq_score'])
```

## Evaluation signals

- After dropping NAs, both score vectors have identical length and their method IDs match one-to-one (paired observations).
- Recorded method count equals the number of rows remaining after the paired NA drop and is consistent across the two vectors.
- Computed pearson_r reproduces the reported value of approximately 0.92 (Fig. 4b).
- The p-value from pearsonr is finite and consistent with the reported significance of the correlation.
- Correlation computed with scipy.stats.pearsonr in Python matches R cor.test on the same aligned data.

## Limitations

- Correlation is over methods, not datasets; a high r (0.92) reflects cross-modality agreement of method accuracy rankings but does not certify any single simulator.
- Methods that run on only one modality are excluded, so the paired set may be smaller than the 49 evaluated methods.
- The article notes some methods tailored to scRNA-seq data have only potential compatibility for simulating SRT data, so per-method SRT scores may be missing for scRNA-seq-specific simulators.

## Evidence

- [other] Extract the paired per-technique score columns (SRT-based scores vs scRNA-seq-based scores) and drop methods with missing values in either modality so observations are paired.: [evidence span withheld — anchor status: not_found]
- [results] Per-method accuracy scores on SRT and scRNA-seq reference datasets are highly correlated, with Pearson's r = 0.92 (Fig. 4b).: [evidence span withheld — anchor status: not_found]
- [abstract] some methods tailored to scRNA-seq data have potential compatibility for simulating SRT data: "some methods tailored to scRNA-seq data have potential compatibility for simulating SRT data"
- [readme] Simmethods collects and documents 49 popular and common simulation methods for single-cell and spatial transcriptome data.: [evidence span withheld — anchor status: external_doc]
