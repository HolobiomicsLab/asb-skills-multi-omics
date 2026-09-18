---
name: prior-knowledge-network-property-annotation
description: Use when building a feature-by-feature comparison of pathway/mechanistic
  modeling tools (e.g., SignalingProfiler 2.0, COSMOS, CARNIVAL, HiPathia, CausalPath)
  and you must populate per-tool cells for omics data types and PKN properties.
when_to_use_negative:
- The goal is to construct or optimize a PKN itself (e.g., create_PKN from SIGNOR/PsP
  resources) rather than annotate and compare existing tools' PKN properties
- Only a single tool is under consideration and no cross-tool property matrix is needed
- The source text does not report classifications for the property axes, so cells
  would have to be inferred rather than extracted as reported
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: SignalingProfiler 2.0
  role: 'Reference tool being annotated: integrates transcriptomics, proteomics, and
    phosphoproteomics with a PKN sourced from SIGNOR and PhosphoSitePlus; the sole
    tool estimating activity of proteins other than kinases/phosphatases from phosphoproteomic
    data, and it does not include metabolomic data'
  repo: https://github.com/SaccoPerfettoLab/SignalingProfiler/
- name: COSMOS
  role: 'Contrast row: supports integrating metabolomics within multi-omics data to
    generate mechanistic hypotheses, e.g., in clear cell renal cell carcinoma patients'
- name: CARNIVAL
  role: Downstream optimization dependency of SignalingProfiler 2.0 whose ILP strategy
    retains sign-coherent interactions in the context-specific network
  repo: https://saezlab.github.io/CARNIVAL/
- name: CausalPath
  role: 'Contrast row: the other approach combining proteomics in the analysis alongside
    SignalingProfiler 2.0'
- name: HiPathia
  role: 'Contrast row: decomposes pathways into functional circuits ending on phenotypes'
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:b33ab706c1d360e1c68165f225ef71e10a886a4f8c65acdd09c9bfba5e8247c1
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
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# prior-knowledge-network-property-annotation

## Summary

Annotate each mechanistic modeling tool's prior knowledge network (PKN) and integration capabilities along explicit property axes (omics layers accepted, PKN database provenance, Boolean kinase-substrate attributes, algorithm type) to build a structured per-tool comparison matrix, as done in the SignalingProfiler 2.0 paper's nine-tool comparison.

## When to use

Use when building a feature-by-feature comparison of pathway/mechanistic modeling tools (e.g., SignalingProfiler 2.0, COSMOS, CARNIVAL, HiPathia, CausalPath) and you must populate per-tool cells for omics data types and PKN properties.

## When NOT to use

- The goal is to construct or optimize a PKN itself (e.g., create_PKN from SIGNOR/PsP resources) rather than annotate and compare existing tools' PKN properties
- Only a single tool is under consideration and no cross-tool property matrix is needed
- The source text does not report classifications for the property axes, so cells would have to be inferred rather than extracted as reported

## Inputs

- Article text with a tool-comparison section covering nine mechanistic modeling tools
- Reported PKN source databases per tool (e.g., SIGNOR, PhosphoSitePlus, Ser/Thr Kinome Atlas, CollectTRI, Dorothea, Omnipath)
- Reported omics integration capabilities per tool (transcriptomics, proteomics, phosphoproteomics, metabolomics)

## Outputs

- Structured tool-comparison matrix (e.g., tool_comparison_table.csv) with one row per tool and one column per property
- Boolean and categorical PKN property annotations per tool

## How to apply

Define the property axes first: omics layers used (transcriptomics, proteomics, phosphoproteomics, metabolomics), PKN attributes (Boolean flags such as whether the PKN contains kinase-substrate relations from the Ser/Thr Kinome Atlas), algorithm type (statistical, footprint-based such as VIPER, graph-based such as ProxPath, optimization-based such as CARNIVAL ILP), and final network properties. Then, for every tool row (e.g., SignalingProfiler 2.0, COSMOS, CausalPath, TPS, CARNIVAL, HiPathia, CausalR, NicheNet, KPNN), extract the classification from the paper's tool-comparison statements, grounding each Boolean and categorical value in explicit textual evidence. Encode the result as a structured matrix (one row per tool, one column per property) with values exactly as reported, then cross-check each cell against the paper's statements before exporting (e.g., tool_comparison_table.csv).

## Related tools

- **SignalingProfiler 2.0** (Reference tool being annotated: integrates transcriptomics, proteomics, and phosphoproteomics with a PKN sourced from SIGNOR and PhosphoSitePlus; the sole tool estimating activity of proteins other than kinases/phosphatases from phosphoproteomic data, and it does not include metabolomic data) — https://github.com/SaccoPerfettoLab/SignalingProfiler/
- **COSMOS** (Contrast row: supports integrating metabolomics within multi-omics data to generate mechanistic hypotheses, e.g., in clear cell renal cell carcinoma patients)
- **CARNIVAL** (Downstream optimization dependency of SignalingProfiler 2.0 whose ILP strategy retains sign-coherent interactions in the context-specific network) — https://saezlab.github.io/CARNIVAL/
- **CausalPath** (Contrast row: the other approach combining proteomics in the analysis alongside SignalingProfiler 2.0)
- **HiPathia** (Contrast row: decomposes pathways into functional circuits ending on phenotypes)

## Examples

```
create_PKN(database = c('SIGNOR', 'PsP'), direct = TRUE, organism = 'human')
```

## Evaluation signals

- The matrix contains exactly all nine tool rows (SignalingProfiler 2.0, COSMOS, CausalPath, TPS, CARNIVAL, HiPathia, CausalR, NicheNet, KPNN) with no missing property cells
- Every populated cell is cross-checked against the paper's statements; e.g., SignalingProfiler 2.0 is the sole tool estimating activity of proteins other than kinases/phosphatases from phosphoproteomic data
- The metabolomics column distinguishes COSMOS (supports metabolomics) from SignalingProfiler 2.0 (does not include metabolomic data)
- Boolean PKN attributes (e.g., Ser/Thr Kinome Atlas kinase-substrate relations) are encoded exactly as reported, as 'Boolean, indicating if PKN contains kinase-substrate relations from the Ser/Thr Kinome Atlas'
- Property values are written verbatim as reported rather than paraphrased or inferred

## Limitations

- The comparison reflects only properties reported in the source article; unreported cells cannot be filled without inference
- PKN coverage in underlying resources is incomplete: regulon and causal interaction databases offer information for <50% (about 9,000 proteins) of the UniProt-SwissProt proteome
- PsP Regulatory_sites and Kinase_Substrates files must be manually downloaded if PhosphoSitePlus information is included in the PKN

## Evidence

- [intro] Kinome Atlas integration Boolean flag for PKN kinase-substrate provenance: "Kinome Atlas integration*Boolean, indicating if PKN contains kinase-substrate relations from theSer/Thr Kinome Atlas28"
- [discussion] SignalingProfiler 2.0 lacks metabolomic data unlike COSMOS: "Compared to tools such as COSMOS23, SignalingProfiler 2.0 does not include metabolomic data"
- [discussion] SignalingProfiler 2.0 uniqueness on non-kinase/phosphatase activity from phosphoproteomics: "is the sole tool capable of estimating the activity of proteins, aside from kinases and phosphatases, from the phosphoproteomic data"
- [methods] PKN construction from SIGNOR and PhosphoSitePlus as the annotated resource layer: [evidence span withheld — anchor status: not_found]
- [readme] README defines the network nodes the PKN connects: [evidence span withheld — anchor status: not_found]
