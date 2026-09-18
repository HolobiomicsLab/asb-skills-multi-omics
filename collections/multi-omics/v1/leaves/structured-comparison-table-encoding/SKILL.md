---
name: structured-comparison-table-encoding
description: Use when an article explicitly compares multiple mechanistic modeling
  tools (e.g., nine tools such as SignalingProfiler 2.0, COSMOS, CausalPath, TPS,
  CARNIVAL, HiPathia, CausalR, NicheNet, KPNN) and you must reproduce the comparison
  table per property axis.
when_to_use_negative:
- The article does not contain an explicit multi-tool comparison section — do not
  infer property values from unrelated passages
- Input is a benchmarking results table or raw omics data rather than published per-tool
  classifications; this skill encodes reported classifications, not computed benchmark
  metrics
- Fewer tools than the article's full tool set are needed and per-property coverage
  would be partial — the validation gate requires all 9 tool rows and no missing property
  cells
tools:
- name: SignalingProfiler 2.0
  role: Primary tool row in the comparison table; the sole tool estimating activity
    of proteins other than kinases/phosphatases from phosphoproteomic data and the
    one lacking metabolomic data integration
  repo: github.com/SaccoPerfettoLab/SignalingProfiler
- name: COSMOS
  role: 'Contrast row: supports integrating metabolomics within multi-omics data to
    generate mechanistic hypotheses, as shown in clear cell renal cell carcinoma patients'
- name: CARNIVAL
  role: Row for the optimization-strategy class; its sign-coherent interaction retention
    defines the algorithm-type axis for SignalingProfiler 2.0's Step 2
  repo: saezlab.github.io/CARNIVAL
- name: CausalPath
  role: Row for the tool combining proteomics in the analysis alongside SignalingProfiler
    2.0
- name: HiPathia
  role: Row for tools that decompose pathways into functional circuits ending on phenotypes
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:9a8d583fe0f2b472e517223933f00a380360d338687a3c2a6428908e16d8d9c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SaccoPerfettoLab/SignalingProfiler
  tools:
  - CARNIVAL
  - COSMOS
  - CausalPath
  - HiPathia
  - SignalingProfiler 2.0
  - VIPER
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# structured-comparison-table-encoding

## Summary

Encodes per-tool property classifications (omics layers, PKN attributes, algorithm type, final network properties) extracted from a paper's tool-comparison section into a structured one-row-per-tool matrix, validated cell-by-cell against the article's statements before export as tool_comparison_table.csv.

## When to use

Use when an article explicitly compares multiple mechanistic modeling tools (e.g., nine tools such as SignalingProfiler 2.0, COSMOS, CausalPath, TPS, CARNIVAL, HiPathia, CausalR, NicheNet, KPNN) and you must reproduce the comparison table per property axis.

## When NOT to use

- The article does not contain an explicit multi-tool comparison section — do not infer property values from unrelated passages
- Input is a benchmarking results table or raw omics data rather than published per-tool classifications; this skill encodes reported classifications, not computed benchmark metrics
- Fewer tools than the article's full tool set are needed and per-property coverage would be partial — the validation gate requires all 9 tool rows and no missing property cells

## Inputs

- Article text with a tool-comparison section (discussion/intro) covering nine mechanistic modeling tools
- Property axes: omics layers used, PKN properties (e.g., Ser/Thr Kinome Atlas Boolean), algorithm type, final network properties

## Outputs

- tool_comparison_table.csv with one row per tool and one column per property axis
- Structured matrix of reported classifications with Boolean flags encoded exactly as reported

## How to apply

First read the article's tool-comparison section (e.g., the discussion of SignalingProfiler 2.0 vs. COSMOS, CausalPath, HiPathia, ProxPath, VIPER, CARNIVAL) and enumerate the property axes: omics layers integrated, PKN properties (including Boolean flags such as whether the PKN contains kinase-substrate relations from the Ser/Thr Kinome Atlas), algorithm type, and final network properties. Then, for each of the nine tools, extract the reported classification for every property exactly as stated, preserving tool names and Boolean values verbatim. Encode these as a structured matrix with one row per tool and one column per property. Finally, cross-check each populated cell against the paper's text — e.g., verify that SignalingProfiler 2.0 is recorded as the sole tool estimating activity of proteins other than kinases/phosphatases from phosphoproteomic data, and that it is flagged as not including metabolomic data unlike COSMOS — and confirm the table contains all 9 tool rows with no missing property cells before saving tool_comparison_table.csv.

## Related tools

- **SignalingProfiler 2.0** (Primary tool row in the comparison table; the sole tool estimating activity of proteins other than kinases/phosphatases from phosphoproteomic data and the one lacking metabolomic data integration) — github.com/SaccoPerfettoLab/SignalingProfiler
- **COSMOS** (Contrast row: supports integrating metabolomics within multi-omics data to generate mechanistic hypotheses, as shown in clear cell renal cell carcinoma patients)
- **CARNIVAL** (Row for the optimization-strategy class; its sign-coherent interaction retention defines the algorithm-type axis for SignalingProfiler 2.0's Step 2) — saezlab.github.io/CARNIVAL
- **CausalPath** (Row for the tool combining proteomics in the analysis alongside SignalingProfiler 2.0)
- **HiPathia** (Row for tools that decompose pathways into functional circuits ending on phenotypes)

## Examples

```
import pandas as pd; df = pd.DataFrame(columns=['tool','omics_layers','ser_thr_kinome_atlas_PKN','algorithm_type','metabolomics','network_properties']); df.to_csv('tool_comparison_table.csv', index=False)
```

## Evaluation signals

- Row count check: the table contains exactly 9 tool rows (SignalingProfiler 2.0, COSMOS, CausalPath, TPS, CARNIVAL, HiPathia, CausalR, NicheNet, KPNN)
- Completeness check: no missing property cells in the tool × property matrix
- Cross-check invariant: SignalingProfiler 2.0 is the only row marked as estimating activity of proteins other than kinases/phosphatases from phosphoproteomic data
- Boolean check: the metabolomic-data flag is FALSE for SignalingProfiler 2.0 and TRUE for COSMOS, per the paper's direct comparison
- Boolean check: the Ser/Thr Kinome Atlas PKN flag is encoded exactly as reported in the paper's table

## Limitations

- Classifications must be transcribed exactly as reported; the paper notes SignalingProfiler 2.0 does not include metabolomic data and cannot integrate epigenetic, acetylomic, and ubiquitylomic data, so absence of a capability must be encoded as a negative flag rather than left blank
- Prior knowledge coverage is incomplete — regulon and causal interaction resources cover <50% (about 9,000 proteins) of the UniProt-SwissProt proteome — which affects how PKN-coverage property cells should be interpreted
- If the article's comparison section is silent on a property for a tool, the cell cannot be honestly populated; the workflow demands no missing cells, so such gaps must be resolved against the paper's statements or flagged

## Evidence

- [intro] COSMOS supports metabolomics integration: "the COSMOS pipeline has been used to generate mechanistic hypotheses from multi-omics data, including metabolomics, in patients with clear cell renal cell carcinoma"
- [discussion] SignalingProfiler 2.0 sole non-kinase/phosphatase activity estimator: "is the sole tool capable of estimating the activity of proteins, aside from kinases and phosphatases, from the phosphoproteomic data"
- [discussion] Metabolomic exclusion of SignalingProfiler 2.0: "Compared to tools such as COSMOS23, SignalingProfiler 2.0 does not include metabolomic data"
- [intro] HiPathia pathway decomposition classification: "other tools, such as HiPathia21, decompose pathways into functional circuits ending on phenotypes"
- [intro] CARNIVAL optimization strategy axis: "the CARNIVAL optimization strategy19 that retains only the sign-coherent interactions between proteins (context-specific network)"
- [discussion] CausalPath proteomics combination classification: "is the sole approach together with CausalPath42 combining proteomics in the analysis"
