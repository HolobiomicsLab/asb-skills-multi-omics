---
name: phosphoproteomic-activity-inference-method-taxonomy
description: Use when a paper presents or cites multiple mechanistic network-inference
  tools (e.g. SignalingProfiler 2.0, COSMOS, CausalPath, CARNIVAL, HiPathia, NicheNet,
  KPNN) and a side-by-side property comparison table must be filled from text.
when_to_use_negative:
- The paper describes only a single tool's workflow and offers no comparative statements
  about alternative methods.
- The goal is to run the tools rather than curate their qualitative properties — use
  the SignalingProfiler pipeline skill instead.
- Comparison axes require quantitative benchmarking results (e.g. AUROC) that the
  paper's qualitative table does not report.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SignalingProfiler 2.0
  role: Reference tool that is the sole approach estimating activity of proteins other
    than kinases/phosphatases from phosphoproteomic data; compared against all other
    tools
  repo: https://github.com/SaccoPerfettoLab/SignalingProfiler
- name: COSMOS
  role: Compared tool that integrates metabolomics within multi-omics data to generate
    mechanistic hypotheses (clear cell renal cell carcinoma application)
- name: CARNIVAL
  role: Optimization strategy retained as the sign-coherent interaction step compared
    across tools; also a SignalingProfiler dependency
  repo: https://saezlab.github.io/CARNIVAL/
- name: VIPER
  role: Footprint-based algorithm used in SignalingProfiler Step 1, one axis of the
    algorithm-type comparison
- name: ProxPath
  role: Graph-based tool connecting proteins to SIGNOR phenotypes, one axis of the
    algorithm-type comparison
- name: HiPathia
  role: Tool that decomposes pathways into functional circuits ending on phenotypes,
    included as a comparison row
- name: CausalPath
  role: Tool combining proteomics in the analysis alongside SignalingProfiler 2.0,
    included as a comparison row
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:ae45de16d0577fe420f6a24ac99c042becb79d249ec4d67ff04125d23bf28097
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/SaccoPerfettoLab/SignalingProfiler
  tools:
  - CARNIVAL
  - COSMOS
  - CausalPath
  - HiPathia
  - ProxPath
  - SignalingProfiler 2.0
  - VIPER
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# phosphoproteomic-activity-inference-method-taxonomy

## Summary

A curation skill for building a structured per-tool property matrix (omics layers integrated, PKN attributes, algorithm type, final network properties) across mechanistic signaling-modeling tools, as done in the SignalingProfiler 2.0 comparison of nine tools.

## When to use

Use when a paper presents or cites multiple mechanistic network-inference tools (e.g. SignalingProfiler 2.0, COSMOS, CausalPath, CARNIVAL, HiPathia, NicheNet, KPNN) and a side-by-side property comparison table must be filled from text.

## When NOT to use

- The paper describes only a single tool's workflow and offers no comparative statements about alternative methods.
- The goal is to run the tools rather than curate their qualitative properties — use the SignalingProfiler pipeline skill instead.
- Comparison axes require quantitative benchmarking results (e.g. AUROC) that the paper's qualitative table does not report.

## Inputs

- Article full text with a tool-comparison or discussion section
- List of tool names and property axes to populate (e.g. omics layers, PKN attributes, algorithm type)
- Prior knowledge network source descriptors (SIGNOR, PhosphoSitePlus, Ser/Thr Kinome Atlas, CollectTRI, DoRothEA)

## Outputs

- tool_comparison_table.csv: structured matrix with one row per tool (9 tools) and one column per property axis, Boolean flags as reported

## How to apply

Read the tool-comparison section to identify the property axes: omics layers used (transcriptomics, proteomics, phosphoproteomics, metabolomics), PKN properties (e.g. Boolean 'Kinome Atlas integration' flag for Ser/Thr Kinome Atlas kinase-substrate relations), algorithm type (statistical, footprint-based such as VIPER, optimization-based such as CARNIVAL ILP, graph-based such as ProxPath), and final network properties. For each tool row, extract the reported classification verbatim, using the paper's explicit statements as anchors — e.g. SignalingProfiler 2.0 is the sole tool estimating activity of proteins other than kinases/phosphatases from phosphoproteomic data, and it does not include metabolomic data unlike COSMOS, which was applied to clear cell renal cell carcinoma patients. Encode the result as a one-row-per-tool, one-column-per-property matrix (tool_comparison_table.csv). Validate by cross-checking every populated cell against the paper's statements and confirming all 9 tool rows are present with no missing property cells.

## Related tools

- **SignalingProfiler 2.0** (Reference tool that is the sole approach estimating activity of proteins other than kinases/phosphatases from phosphoproteomic data; compared against all other tools) — https://github.com/SaccoPerfettoLab/SignalingProfiler
- **COSMOS** (Compared tool that integrates metabolomics within multi-omics data to generate mechanistic hypotheses (clear cell renal cell carcinoma application))
- **CARNIVAL** (Optimization strategy retained as the sign-coherent interaction step compared across tools; also a SignalingProfiler dependency) — https://saezlab.github.io/CARNIVAL/
- **VIPER** (Footprint-based algorithm used in SignalingProfiler Step 1, one axis of the algorithm-type comparison)
- **ProxPath** (Graph-based tool connecting proteins to SIGNOR phenotypes, one axis of the algorithm-type comparison)
- **HiPathia** (Tool that decomposes pathways into functional circuits ending on phenotypes, included as a comparison row)
- **CausalPath** (Tool combining proteomics in the analysis alongside SignalingProfiler 2.0, included as a comparison row)

## Examples

```
devtools::install_github('https://github.com/SaccoPerfettoLab/SignalingProfiler/')
```

## Evaluation signals

- tool_comparison_table.csv contains exactly 9 tool rows (SignalingProfiler 2.0, COSMOS, CausalPath, TPS, CARNIVAL, HiPathia, CausalR, NicheNet, KPNN).
- No missing property cells: every tool-by-property cell is populated or explicitly marked as reported by the paper.
- Boolean PKN flags (e.g. Ser/Thr Kinome Atlas integration) are encoded exactly as stated in the article text.
- Cell values for metabolomics integration distinguish COSMOS (yes) from SignalingProfiler 2.0 (no).
- Uniqueness claim holds: only SignalingProfiler 2.0 is marked as estimating non-kinase/non-phosphatase protein activity from phosphoproteomic data.

## Limitations

- SignalingProfiler 2.0 does not include metabolomic data (limitation relative to COSMOS), nor epigenetic, acetylomic, or ubiquitylomic data.
- Regulon and causal-interaction resources cover <50% (about 9,000 proteins) of the UniProt-SwissProt proteome, constraining PKN-based comparison axes.
- Qualitative classifications are only as complete as the paper's statements; tools or properties not discussed in the text cannot be filled without invention.

## Evidence

- [discussion] Compared to tools such as COSMOS23, SignalingProfiler 2.0 does not include metabolomic data: "Compared to tools such as COSMOS23, SignalingProfiler 2.0 does not include metabolomic data"
- [discussion] is the sole tool capable of estimating the activity of proteins, aside from kinases and phosphatases, from the phosphoproteomic data: "is the sole tool capable of estimating the activity of proteins, aside from kinases and phosphatases, from the phosphoproteomic data"
- [intro] the COSMOS pipeline has been used to generate mechanistic hypotheses from multi-omics data, including metabolomics, in patients with clear cell renal cell carcinoma: "the COSMOS pipeline has been used to generate mechanistic hypotheses from multi-omics data, including metabolomics, in patients with clear cell renal cell carcinoma"
- [intro] Kinome Atlas integration*Boolean, indicating if PKN contains kinase-substrate relations from theSer/Thr Kinome Atlas28: "Kinome Atlas integration*Boolean, indicating if PKN contains kinase-substrate relations from theSer/Thr Kinome Atlas28"
- [readme] SignalingProfiler is a R based pipeline that integrates transcriptomics, proteomics and phosphoproteomics data among a case and a control: [evidence span withheld — anchor status: not_found]
