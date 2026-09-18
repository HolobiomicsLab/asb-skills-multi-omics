---
name: signaling-network-tool-capability-comparison
description: Use when decision support for choosing a mechanistic signaling-network
  tool when the analysis goal requires knowing which omics data types (transcriptomics,
  proteomics, phosphoproteomics, metabolomics) each tool integrates and how its PKN
  and algorithm differ.
when_to_use_negative:
- When the task is to actually run one of the pipelines (e.g., execute SignalingProfiler
  2.0 on user omics data) rather than compare tool capabilities
- When the question concerns quantitative benchmarking performance of tools rather
  than the qualitative per-tool property classifications reported in the paper's table
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3941
tools:
- name: SignalingProfiler 2.0
  role: Subject of the comparison; sole tool estimating activity of proteins other
    than kinases/phosphatases from phosphoproteomic data; does not include metabolomic
    data
  repo: github.com/SaccoPerfettoLab/SignalingProfiler
- name: COSMOS
  role: Compared tool; supports integrating metabolomics within multi-omics data to
    generate mechanistic hypotheses (clear cell renal cell carcinoma application)
- name: CARNIVAL
  role: Compared tool; its optimization strategy retains only sign-coherent interactions
    between proteins to build the context-specific network
  repo: saezlab.github.io/CARNIVAL
- name: CausalPath
  role: Compared tool; combined with SignalingProfiler in combining proteomics in
    the analysis
- name: HiPathia
  role: Compared tool; decomposes pathways into functional circuits ending on phenotypes
- name: VIPER
  role: Footprint-based algorithm used by SignalingProfiler 2.0 to direct transcripts/phosphopeptides;
    relevant to the algorithm-type comparison axis
- name: ProxPath
  role: Graph-based tool used in Step 3 to connect the context-specific network to
    cellular phenotypes; relevant to the algorithm-type comparison axis
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:28d5d8a1e244470f179166dc137e3dd2bbd501edf8f6d838213ec9747667ed92
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
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3941
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# signaling-network-tool-capability-comparison

## Summary

A curation skill for reproducing the qualitative tool-comparison table from the SignalingProfiler 2.0 paper: classifying nine mechanistic modeling tools (SignalingProfiler 2.0, COSMOS, CausalPath, TPS, CARNIVAL, HiPathia, CausalR, NicheNet, KPNN) across property axes such as omics layers used, PKN properties (including Ser/Thr Kinome Atlas integration), algorithm type, and final network properties.

## When to use

Decision support for choosing a mechanistic signaling-network tool when the analysis goal requires knowing which omics data types (transcriptomics, proteomics, phosphoproteomics, metabolomics) each tool integrates and how its PKN and algorithm differ.

## When NOT to use

- When the task is to actually run one of the pipelines (e.g., execute SignalingProfiler 2.0 on user omics data) rather than compare tool capabilities
- When the question concerns quantitative benchmarking performance of tools rather than the qualitative per-tool property classifications reported in the paper's table

## Inputs

- Article text of SignalingProfiler 2.0 (DOI 10.1038/s41540-024-00417-6), especially the introduction and discussion tool-comparison passages
- List of the nine compared tools: SignalingProfiler 2.0, COSMOS, CausalPath, TPS, CARNIVAL, HiPathia, CausalR, NicheNet, KPNN
- Property axes from the paper's table (omics layers, PKN properties such as Ser/Thr Kinome Atlas integration, algorithm type, final network properties)

## Outputs

- tool_comparison_table.csv — a structured matrix with one row per tool (9 rows) and one column per property axis, with Boolean flags and classifications encoded exactly as reported

## How to apply

Read the article's tool-comparison section to identify the property axes (omics layers used, PKN properties, algorithm type, final network properties). For each of the nine tools, extract the reported classification for every axis, including Boolean PKN flags such as whether the PKN contains kinase-substrate relations from the Ser/Thr Kinome Atlas. Encode the results as a structured matrix with one row per tool and one column per property, preserving tool names and Boolean values exactly as reported. Validate each populated cell against the paper's statements — e.g., SignalingProfiler 2.0 is the sole tool estimating activity of proteins other than kinases/phosphatases from phosphoproteomic data, and unlike COSMOS it does not include metabolomic data — and confirm all 9 tool rows and no missing property cells before saving tool_comparison_table.csv.

## Related tools

- **SignalingProfiler 2.0** (Subject of the comparison; sole tool estimating activity of proteins other than kinases/phosphatases from phosphoproteomic data; does not include metabolomic data) — github.com/SaccoPerfettoLab/SignalingProfiler
- **COSMOS** (Compared tool; supports integrating metabolomics within multi-omics data to generate mechanistic hypotheses (clear cell renal cell carcinoma application))
- **CARNIVAL** (Compared tool; its optimization strategy retains only sign-coherent interactions between proteins to build the context-specific network) — saezlab.github.io/CARNIVAL
- **CausalPath** (Compared tool; combined with SignalingProfiler in combining proteomics in the analysis)
- **HiPathia** (Compared tool; decomposes pathways into functional circuits ending on phenotypes)
- **VIPER** (Footprint-based algorithm used by SignalingProfiler 2.0 to direct transcripts/phosphopeptides; relevant to the algorithm-type comparison axis)
- **ProxPath** (Graph-based tool used in Step 3 to connect the context-specific network to cellular phenotypes; relevant to the algorithm-type comparison axis)

## Examples

```
R> devtools::install_github('https://github.com/SaccoPerfettoLab/SignalingProfiler/')
```

## Evaluation signals

- tool_comparison_table.csv contains exactly 9 tool rows: SignalingProfiler 2.0, COSMOS, CausalPath, TPS, CARNIVAL, HiPathia, CausalR, NicheNet, KPNN
- No missing property cells: every tool row has a value for each property axis (omics layers, PKN properties, algorithm type, final network properties)
- Boolean PKN flags match the paper, e.g. the Ser/Thr Kinome Atlas kinase-substrate integration flag is populated exactly as reported
- Cross-check: the SignalingProfiler 2.0 row shows it is the sole tool estimating activity of non-kinase/non-phosphatase proteins from phosphoproteomic data
- Cross-check: COSMOS row flagged as including metabolomic data while SignalingProfiler 2.0 row is flagged as not including metabolomic data

## Limitations

- SignalingProfiler 2.0 does not include metabolomic data (a limitation relative to COSMOS)
- The comparison table is qualitative and derived from the paper's reported classifications; properties not stated in the text cannot be filled
- Prior knowledge coverage is incomplete: regulon and causal interaction resources cover <50% (~9,000 proteins) of the UniProt-SwissProt proteome, which affects PKN-related comparison axes

## Evidence

- [intro] COSMOS supports metabolomics integration: "the COSMOS pipeline has been used to generate mechanistic hypotheses from multi-omics data, including metabolomics, in patients with clear cell renal cell carcinoma"
- [discussion] SignalingProfiler 2.0 lacks metabolomic data: "Compared to tools such as COSMOS23, SignalingProfiler 2.0 does not include metabolomic data"
- [discussion] Sole tool for non-kinase/phosphatase activity from phosphoproteomics: "is the sole tool capable of estimating the activity of proteins, aside from kinases and phosphatases, from the phosphoproteomic data"
- [intro] HiPathia circuit decomposition: "other tools, such as HiPathia21, decompose pathways into functional circuits ending on phenotypes"
- [intro] CARNIVAL retains sign-coherent interactions: "the CARNIVAL optimization strategy19 that retains only the sign-coherent interactions between proteins (context-specific network)"
- [intro] PKN includes Ser/Thr Kinome Atlas relations: "Kinome Atlas integration*Boolean, indicating if PKN contains kinase-substrate relations from theSer/Thr Kinome Atlas28"
- [discussion] SP2.0 combines proteomics with CausalPath: "is the sole approach together with CausalPath42 combining proteomics in the analysis"
- [readme] README: installation command: "devtools::install_github('https://github.com/SaccoPerfettoLab/SignalingProfiler/')"
