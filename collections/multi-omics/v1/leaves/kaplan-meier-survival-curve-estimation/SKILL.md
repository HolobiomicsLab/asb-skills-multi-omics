---
name: kaplan-meier-survival-curve-estimation
description: Use when patient samples have been partitioned into groups (e.g., network-based
  clusters) and merged with clinical metadata (survival time, censoring status), and
  you must test whether one group shows significantly worse survival (p < 0.05).
when_to_use_negative:
- Input is a single homogeneous cohort without group labels or stratification (no
  cluster assignments to compare)
- Clinical metadata lacks censoring status or survival time, so Kaplan–Meier estimation
  is not well-defined
- The goal is inferring the context-specific networks or clusters themselves rather
  than testing survival differences of existing groups
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: pyPARAGON
  role: Upstream network-based stratification of the 105 breast cancer tumors that
    produces the patient clusters whose survival is compared with this skill
  repo: https://github.com/netlab-ku/pyPARAGON
- name: lifelines
  role: Python library providing KaplanMeierFitter, the multivariate_logrank test,
    and survival-curve plotting used to reproduce Fig. 4C / Fig. S5
- name: Python
  role: Execution environment (pandas for merging cluster assignments with clinical
    survival metadata; matplotlib for figure output)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bib/bbae399
    title: 'Unveiling hidden connections in omics data <i>via</i> pyPARAGON: an integrative
      hybrid approach for disease network construction'
  source_content_hash: sha256:2e934b26b78aab089784057895464fa464b99a7a9dffaa31ad5415083ce287d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/netlab-ku/pyPARAGON
  tools:
  - Python
  - lifelines
  - pyPARAGON
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# kaplan-meier-survival-curve-estimation

## Summary

Estimate per-group Kaplan–Meier overall survival curves and compare groups with the log-rank test, as applied in pyPARAGON's breast cancer use case to test whether network-based patient clusters from 105 CPTAC tumors differ in overall survival (Fig. 4C, Fig. S5).

## When to use

Use when patient samples have been partitioned into groups (e.g., network-based clusters) and merged with clinical metadata (survival time, censoring status), and you must test whether one group shows significantly worse survival (p < 0.05).

## When NOT to use

- Input is a single homogeneous cohort without group labels or stratification (no cluster assignments to compare)
- Clinical metadata lacks censoring status or survival time, so Kaplan–Meier estimation is not well-defined
- The goal is inferring the context-specific networks or clusters themselves rather than testing survival differences of existing groups

## Inputs

- Patient cluster assignment table for the 105 CPTAC breast cancer tumors (from pyPARAGON network-based stratification)
- Clinical overall-survival metadata per patient (survival time, censoring/event status)

## Outputs

- Per-cluster Kaplan–Meier survival curve estimates with confidence intervals (e.g., cluster-4 vs cluster-1)
- Log-rank (multivariate_logrank) p-value for the survival difference between clusters
- Annotated Kaplan–Meier figure (reproduction of Fig. 4C / Fig. S5)

## How to apply

Merge the patient cluster assignments from the prior stratification sub-task (e.g., the four clusters obtained from pyPARAGON-based network stratification of the 105 CPTAC breast cancer tumors) with clinical overall-survival metadata (survival time, censoring flag) using pandas. Fit a KaplanMeierFitter per cluster in lifelines, using survival time as the duration and the censoring flag as the event indicator, so censored patients are handled correctly. Compare the groups of interest (here cluster-4 versus cluster-1) with the multivariate_logrank test in lifelines to obtain a p-value for the survival difference. Plot the Kaplan–Meier curves with confidence intervals using lifelines' matplotlib plotting utilities, annotating the log-rank p-value on the figure, mirroring Fig. 4C / Fig. S5 of the paper.

## Related tools

- **pyPARAGON** (Upstream network-based stratification of the 105 breast cancer tumors that produces the patient clusters whose survival is compared with this skill) — https://github.com/netlab-ku/pyPARAGON
- **lifelines** (Python library providing KaplanMeierFitter, the multivariate_logrank test, and survival-curve plotting used to reproduce Fig. 4C / Fig. S5)
- **Python** (Execution environment (pandas for merging cluster assignments with clinical survival metadata; matplotlib for figure output))

## Examples

```
from lifelines import KaplanMeierFitter, statistics; kmf4 = KaplanMeierFitter().fit(t4, e4, label='cluster-4'); kmf1 = KaplanMeierFitter().fit(t1, e1, label='cluster-1'); p = statistics.multivariate_logrank_test([*t4,*t1],[4]*len(t4)+[1]*len(t1),[*e4,*e1]).p_value
```

## Evaluation signals

- The reproduced Kaplan–Meier figure shows cluster-4 with lower survival probability than cluster-1 across follow-up time
- The multivariate_logrank p-value for cluster-4 vs cluster-1 is below the significance threshold reported in the paper (p < 0.05)
- Censored patients are treated as non-events: the event indicator passed to KaplanMeierFitter is the censoring flag, and curve step-downs occur only at observed death times
- All 105 CPTAC breast cancer tumors are retained after the pandas merge (no silent sample loss from mismatched identifiers)
- Curves are plotted with confidence intervals and the annotated log-rank p-value, matching the presentation of Fig. 4C / Fig. S5

## Limitations

- Kaplan–Meier with log-rank testing depends on the quality of the upstream stratification; pyPARAGON's inferred networks can include generic biological processes (e.g., transcription, replication), which may affect cluster definitions
- With only 105 tumors split across four clusters, individual group sizes are small, so curves and p-values are sensitive to censoring patterns and group membership changes
- Log-rank comparison of a single pair (cluster-4 vs cluster-1) is a targeted hypothesis test; it does not by itself establish omnibus differences among all four clusters without additional correction

## Evidence

- [abstract] we demonstrate its utility in network-based stratification of patient tumors by integrating phosphoproteomic data from 105 breast cancer tumors with the interactome: "we demonstrate its utility in network-based stratification of patient tumors by integrating phosphoproteomic data from 105 breast cancer tumors with the interactome"
- [results] Kaplan–Meier analysis with the log-rank test of overall survival showed that patients in cluster-4 have a significantly worse survival probability than patients in cluster-1: [evidence span withheld — anchor status: not_found]
- [readme] This integration of annotations and inferred subnetworks provides patient stratification, survival analysis, and personalized medicine.: "This integration of annotations and inferred subnetworks provides patient stratification, survival analysis, and personalized medicine."
