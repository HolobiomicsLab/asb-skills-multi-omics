---
name: clinical-survival-metadata-integration
description: Use when network-based patient stratification has produced cluster labels
  (e.g. pyPARAGON clusters of 105 breast cancer tumors) and the goal is to test whether
  clusters differ in overall survival.
when_to_use_negative:
- Input is a per-patient context-specific network or community/module annotation from
  pyPARAGON but no survival time or censoring metadata is available — no survival
  analysis is possible
- The analysis target is network inference, PageRank-flux edge scoring, or graphlet-guided
  network construction rather than post-stratification clinical evaluation
- The outcome is a non-time-to-event endpoint (e.g. drug sensitivity scores from GDSC/CCLE)
  where log-rank/Kaplan–Meier does not apply
tools:
- name: pyPARAGON
  role: Upstream tool whose community analysis module produces the patient stratification
    (cluster assignments) that are merged with survival metadata; the README states
    the annotations/subnetwork integration supports survival analysis
  repo: https://github.com/netlab-ku/pyPARAGON
- name: Python
  role: Execution environment (venv 'pyPARAGONenv') and scripting language for pandas
    merging, lifelines KaplanMeierFitter fitting, log-rank testing, and matplotlib
    plotting
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:6dbd87a9de1a8bd7a16d9ea98a96ca2583fd99a3a874fee476df34e49f768981
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/netlab-ku/pyPARAGON
  tools:
  - Python
  - pyPARAGON
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# clinical-survival-metadata-integration

## Summary

Merges pyPARAGON-derived patient cluster assignments for the 105 CPTAC breast-cancer tumors with clinical overall-survival metadata (survival time, censoring flag), then fits per-cluster Kaplan–Meier curves and tests cluster-pair differences with the log-rank test.

## When to use

Use when network-based patient stratification has produced cluster labels (e.g. pyPARAGON clusters of 105 breast cancer tumors) and the goal is to test whether clusters differ in overall survival.

## When NOT to use

- Input is a per-patient context-specific network or community/module annotation from pyPARAGON but no survival time or censoring metadata is available — no survival analysis is possible
- The analysis target is network inference, PageRank-flux edge scoring, or graphlet-guided network construction rather than post-stratification clinical evaluation
- The outcome is a non-time-to-event endpoint (e.g. drug sensitivity scores from GDSC/CCLE) where log-rank/Kaplan–Meier does not apply

## Inputs

- Patient cluster assignment table from pyPARAGON stratification (sample/patient ID -> cluster label for the 105 CPTAC breast-cancer tumors)
- Clinical overall-survival metadata for the 105 CPTAC breast-cancer tumors (survival time, censoring/event status)

## Outputs

- Per-cluster Kaplan–Meier survival curves with confidence intervals (matplotlib figure, e.g. Fig. 4C / Fig. S5 reproduction)
- Log-rank p-values for pairwise cluster survival comparisons (e.g. cluster-4 vs cluster-1)
- Merged survival-per-cluster data table (pandas DataFrame)

## How to apply

Load the patient cluster assignments from the prior network-based stratification sub-task (pyPARAGON overrepresented biological process similarity) and merge them with CPTAC breast-cancer clinical overall-survival metadata (survival time, censoring status) for the 105 tumors using pandas, keyed on sample/patient identifiers. Fit a KaplanMeierFitter from lifelines per patient cluster, passing survival time as the duration and the censoring flag as the event indicator. Compare specific cluster pairs (e.g. cluster-4 vs cluster-1) using lifelines' log-rank test to obtain a p-value, then plot the Kaplan–Meier curves with confidence intervals in matplotlib and annotate the log-rank p-value on the figure. Judge success by whether the reproduced figure shows one cluster with lower survival probability than another and the log-rank p-value falls below the significance threshold reported in the paper (p < 0.05, Fig. 4C / Fig. S5).

## Related tools

- **pyPARAGON** (Upstream tool whose community analysis module produces the patient stratification (cluster assignments) that are merged with survival metadata; the README states the annotations/subnetwork integration supports survival analysis) — https://github.com/netlab-ku/pyPARAGON
- **Python** (Execution environment (venv 'pyPARAGONenv') and scripting language for pandas merging, lifelines KaplanMeierFitter fitting, log-rank testing, and matplotlib plotting)

## Examples

```
from lifelines import KaplanMeierFitter; kmf = KaplanMeierFitter(); kmf.fit(durations, event_observed); kmf.plot_survival_function(ci_show=True)
```

## Evaluation signals

- The reproduced Kaplan–Meier figure must show cluster-4 with lower survival probability than cluster-1 (matching Fig. 4C / Fig. S5)
- The log-rank p-value for the cluster-4 vs cluster-1 comparison must be below the paper's significance threshold (p < 0.05)
- Merged survival table must cover the 105 CPTAC breast-cancer tumors with non-null survival time and censoring status for every patient assigned to a cluster
- Each fitted KaplanMeierFitter should use survival time as duration and the censoring flag as the event indicator (uncensored = event/death)
- Plot should include per-cluster confidence intervals and an annotated log-rank p-value

## Limitations

- Survival analysis is only possible after a downstream stratification step; pyPARAGON itself identifies significant modules via hypergeometric tests and the survival link comes from integrating annotations with inferred subnetworks, not from the network inference module directly
- pyPARAGON's inferred networks frequently contain generic biological processes (e.g. transcription, replication) that can complicate which communities/modules are used to define clusters for survival comparison
- The article/README do not state a pyPARAGON software version or provide clinical metadata file formats; survival metadata must be sourced separately (CPTAC breast cancer)

## Evidence

- [readme] This integration of annotations and inferred subnetworks provides patient stratification, survival analysis, and personalized medicine.: "This integration of annotations and inferred subnetworks provides patient stratification, survival analysis, and personalized medicine."
- [discussion] Patients were clustered based on the network-based similarity between the overrepresented biological processes identified with functional modules.: "Patients were clustered based on the network-based similarity between the overrepresented biological processes identified with functional modules."
- [intro] using a breast cancer dataset comprising 105 tumors and associated phosphoproteomic data: "We demonstrated the utility of pyPARAGON in patient stratification using a breast cancer dataset comprising 105 tumors and associated phosphoproteomic data"
