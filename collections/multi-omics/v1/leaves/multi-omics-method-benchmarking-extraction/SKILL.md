---
name: multi-omics-method-benchmarking-extraction
description: Use when an article compares several mechanistic/pathway modeling tools
  across fixed property axes (omics data types, Boolean PKN flags such as Ser/Thr
  Kinome Atlas integration, algorithm class) and you need a machine-readable comparison
  table.
when_to_use_negative:
- The article describes a single tool with no comparative claims about other methods
  — there are no per-tool property cells to extract.
- The goal is to run or benchmark the tools on data rather than extract their reported
  qualitative properties from text.
- A comparison table for these tools already exists and only needs updating for a
  new property axis (extend rather than re-extract).
edam_operation: http://edamontology.org/operation_0306
tools:
- name: SignalingProfiler 2.0
  role: Reference/comparison subject; sole tool estimating activity of non-kinase/phosphatase
    proteins from phosphoproteomic data; lacks metabolomics
  repo: github.com/SaccoPerfettoLab/SignalingProfiler
- name: COSMOS
  role: Comparison subject; supports metabolomics integration for mechanistic hypotheses
    (clear cell renal cell carcinoma example)
- name: CARNIVAL
  role: Comparison subject; optimization strategy retaining sign-coherent interactions
    for context-specific networks
  repo: saezlab.github.io/CARNIVAL
- name: CausalPath
  role: Comparison subject; combines proteomics in the analysis
- name: HiPathia
  role: Comparison subject; decomposes pathways into functional circuits ending on
    phenotypes
- name: VIPER
  role: Comparison subject; footprint-based protein activity estimation used in SignalingProfiler
    Step 1
- name: ProxPath
  role: Comparison subject; graph-based tool estimating regulatory impact of proteins
    on SIGNOR-annotated phenotypes
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41540-024-00417-6
    title: SignalingProfiler 2.0 a network-based approach to bridge multi-omics data
      to phenotypic hallmarks
  source_content_hash: sha256:d8cba635f94331c2ed85fbaf938fe97c916593184881216cb030a67c0962ad6f
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# multi-omics-method-benchmarking-extraction

## Summary

Extracts and encodes per-tool property classifications (omics layers integrated, prior knowledge network attributes, algorithm type, final network properties) from a paper's tool-comparison text into a structured benchmarking matrix, e.g. the nine-tool SignalingProfiler 2.0 vs. COSMOS/CARNIVAL/CausalPath/HiPathia comparison.

## When to use

Use when an article compares several mechanistic/pathway modeling tools across fixed property axes (omics data types, Boolean PKN flags such as Ser/Thr Kinome Atlas integration, algorithm class) and you need a machine-readable comparison table.

## When NOT to use

- The article describes a single tool with no comparative claims about other methods — there are no per-tool property cells to extract.
- The goal is to run or benchmark the tools on data rather than extract their reported qualitative properties from text.
- A comparison table for these tools already exists and only needs updating for a new property axis (extend rather than re-extract).

## Inputs

- Article full text containing tool-comparison statements (intro and discussion sections)
- List of tools to compare (here: SignalingProfiler 2.0, COSMOS, CausalPath, TPS, CARNIVAL, HiPathia, CausalR, NicheNet, KPNN)
- Property axes schema (omics layers, PKN attributes incl. Boolean Ser/Thr Kinome Atlas flag, algorithm type, final network properties)

## Outputs

- tool_comparison_table.csv — structured matrix, one row per tool, one column per property axis, with Boolean flags encoded exactly as reported

## How to apply

Read the article's tool-comparison passages (intro and discussion) to identify the comparison axes: omics layers used (transcriptomics, proteomics, phosphoproteomics, metabolomics), PKN properties including the Boolean 'Kinome Atlas integration' flag, algorithm type (statistical, footprint-based, graph/optimization), and final network properties. For each tool row (SignalingProfiler 2.0, COSMOS, CausalPath, TPS, CARNIVAL, HiPathia, CausalR, NicheNet, KPNN) extract the reported value for every property exactly as stated — e.g. SignalingProfiler 2.0 is the sole tool estimating activity of proteins other than kinases/phosphatases from phosphoproteomic data, and unlike COSMOS it does not include metabolomic data. Encode cells verbatim, preserving Boolean flags (e.g. COSMOS supports metabolomics via its clear cell renal cell carcinoma application; SignalingProfiler 2.0 does not). Before saving tool_comparison_table.csv, cross-check every populated cell against the paper's own sentences and confirm the table has all 9 tool rows with no missing property cells.

## Related tools

- **SignalingProfiler 2.0** (Reference/comparison subject; sole tool estimating activity of non-kinase/phosphatase proteins from phosphoproteomic data; lacks metabolomics) — github.com/SaccoPerfettoLab/SignalingProfiler
- **COSMOS** (Comparison subject; supports metabolomics integration for mechanistic hypotheses (clear cell renal cell carcinoma example))
- **CARNIVAL** (Comparison subject; optimization strategy retaining sign-coherent interactions for context-specific networks) — saezlab.github.io/CARNIVAL
- **CausalPath** (Comparison subject; combines proteomics in the analysis)
- **HiPathia** (Comparison subject; decomposes pathways into functional circuits ending on phenotypes)
- **VIPER** (Comparison subject; footprint-based protein activity estimation used in SignalingProfiler Step 1)
- **ProxPath** (Comparison subject; graph-based tool estimating regulatory impact of proteins on SIGNOR-annotated phenotypes)

## Evaluation signals

- Table contains exactly 9 tool rows (SignalingProfiler 2.0, COSMOS, CausalPath, TPS, CARNIVAL, HiPathia, CausalR, NicheNet, KPNN) and no missing property cells.
- Metabolomics-integration flag is negative for SignalingProfiler 2.0 and positive for COSMOS, matching the paper's discussion statements.
- SignalingProfiler 2.0 is the only row marked as estimating activity of proteins other than kinases/phosphatases from phosphoproteomic data.
- Boolean 'Kinome Atlas integration' PKN flag is populated per tool and matches the paper's stated PKN attributes (e.g. Ser/Thr Kinome Atlas inclusion).
- Every cell can be traced back to a verbatim sentence in the article's intro or discussion when cross-checked.

## Limitations

- SignalingProfiler 2.0 does not include metabolomic data, unlike COSMOS — comparison axes must tolerate heterogeneous data-type support across tools.
- Epigenetic, acetylomic, and ubiquitylomic regulation cannot be integrated into the signaling networks (a stated future challenge), so such omics axes will be uniformly negative.
- Underlying prior knowledge coverage is incomplete: regulon and causal interaction resources offer information for <50% (about 9,000 proteins) of the UniProt-SwissProt proteome, so PKN-property cells reflect incomplete background databases (e.g. CollectTRI, Ser/Thr Kinome Atlas).
- Some properties (e.g. Boolean PKN flags like Ser/Thr Kinome Atlas inclusion) are only extractable from supplementary/table text, which may be absent from the main article index.

## Evidence

- [discussion] Compared to tools such as COSMOS23, SignalingProfiler 2.0 does not include metabolomic data: "Compared to tools such as COSMOS23, SignalingProfiler 2.0 does not include metabolomic data"
- [discussion] Sole tool for non-kinase/phosphatase activity from phosphoproteomics: "is the sole tool capable of estimating the activity of proteins, aside from kinases and phosphatases, from the phosphoproteomic data"
- [intro] COSMOS metabolomics integration example in ccRCC patients: "the COSMOS pipeline has been used to generate mechanistic hypotheses from multi-omics data, including metabolomics, in patients with clear cell renal cell carcinoma"
- [intro] Boolean PKN property axis definition (Ser/Thr Kinome Atlas): [evidence span withheld — anchor status: not_found]
- [readme] README describing tool scope for comparison rows: [evidence span withheld — anchor status: not_found]
