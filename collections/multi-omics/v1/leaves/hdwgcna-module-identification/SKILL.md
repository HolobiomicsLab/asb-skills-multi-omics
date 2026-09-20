---
name: hdwgcna-module-identification
description: Use when benchmarking co-expression module detection on normalised single-cell
  or spatial transcriptomic data (e.g. Seurat-processed Visium kidney GSE261545) and
  the hdWGCNA arm of an SGCRNA-vs-hdWGCNA comparison is needed.
when_to_use_negative:
- 'When the analysis goal requires many fine-grained, cell type-specific modules:
  on the Visium kidney dataset hdWGCNA identified only two modules (grey and turquoise),
  so it is the wrong choice if resolution comparable to SGCRNA''s 114 modules is needed.'
- When the input is raw, unnormalised counts — the comparison workflow requires the
  same Seurat-normalised expression matrix that SGCRNA's CGM preprocessing consumes.
- When sample/column order between the expression matrix and the annotation table
  cannot be verified, since the module-parameter correlation step requires matching
  order.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: hdWGCNA
  role: The WGCNA variant optimised for single-cell and spatial transcriptomic analyses;
    run on the same normalised expression matrix as SGCRNA to produce the baseline
    module set (grey and turquoise on GSE261545).
- name: SGCRNA
  role: Comparator method whose module counts (114 vs hdWGCNA's 2) and spatial module
    patterns define the evaluation target of this skill.
  repo: https://github.com/C37H41N2O6/SGCRNA
- name: Seurat
  role: Provides the standard spatial-transcriptomics normalisation workflow applied
    to GSE261545 before module detection.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.04.27.650628
    title: 'SGCRNA: Spectral Clustering-Guided Co-Expression Network Analysis Without
      Scale-Free Constraints for Multi-Omic Data'
  source_content_hash: sha256:7b7ab0b0e4f3a115bc39a6257866c61ea72e9066530edaea2078c047c400dd54
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/C37H41N2O6/SGCRNAs.jl
  repo_url: https://github.com/C37H41N2O6/SGCRNA
  tools:
  - SGCRNA
  - Seurat
  - hdWGCNA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# hdwgcna-module-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identification of co-expression gene modules with hdWGCNA, the WGCNA variant optimised for single-cell and spatial transcriptomic analyses, used in the SGCRNA paper as the baseline against which SGCRNA's spectral-clustering modules were benchmarked on Visium kidney data (GSE261545).

## When to use

Use when benchmarking co-expression module detection on normalised single-cell or spatial transcriptomic data (e.g. Seurat-processed Visium kidney GSE261545) and the hdWGCNA arm of an SGCRNA-vs-hdWGCNA comparison is needed.

## When NOT to use

- When the analysis goal requires many fine-grained, cell type-specific modules: on the Visium kidney dataset hdWGCNA identified only two modules (grey and turquoise), so it is the wrong choice if resolution comparable to SGCRNA's 114 modules is needed.
- When the input is raw, unnormalised counts — the comparison workflow requires the same Seurat-normalised expression matrix that SGCRNA's CGM preprocessing consumes.
- When sample/column order between the expression matrix and the annotation table cannot be verified, since the module-parameter correlation step requires matching order.

## Inputs

- Normalised spatial or single-cell expression matrix (e.g. Seurat-normalised Visium kidney counts from GSE261545, same matrix passed to SGCRNA's CGM)
- Tissue-spot / phenotype annotation table aligned by column order with the expression matrix (analogous to the SraRunTable.csv clinical data used in the SGCRNA workflow)

## Outputs

- Co-expression module assignments (on GSE261545: two hdWGCNA modules, grey and turquoise)
- Module-count comparison table SGCRNA vs hdWGCNA (module_comparison_sgcrna_hdwgcna.csv)
- Per-module spatial enrichment / module-parameter correlation results (e.g. CorPhenMod-style heatmap output)

## How to apply

Normalise the Visium spatial gene expression dataset with the standard Seurat spatial-transcriptomics workflow, then run hdWGCNA on exactly the same normalised expression matrix that is fed to SGCRNA's CGM (mode=:FTEST) and SpectralClustering (tNodeNum=100, depthMax=5) so the module counts are directly comparable. Tabulate the number of modules returned — on GSE261545 hdWGCNA returns only two (grey and turquoise) versus SGCRNA's 114 — as the headline comparison metric. Correlate each module with tissue-spot annotations using a CorPhenMod-style module-parameter correlation step, verifying that sample/column order matches between the expression matrix and the annotation table before correlating, since a mismatch produces spurious spatial assignments. Record per-module spatial patterns (reported expectation: hdWGCNA grey=tumour, turquoise=healthy proximal tubule) and save the module-count comparison plus per-module spatial enrichment assignments to module_comparison_sgcrna_hdwgcna.csv.

## Related tools

- **hdWGCNA** (The WGCNA variant optimised for single-cell and spatial transcriptomic analyses; run on the same normalised expression matrix as SGCRNA to produce the baseline module set (grey and turquoise on GSE261545).)
- **SGCRNA** (Comparator method whose module counts (114 vs hdWGCNA's 2) and spatial module patterns define the evaluation target of this skill.) — https://github.com/C37H41N2O6/SGCRNA
- **Seurat** (Provides the standard spatial-transcriptomics normalisation workflow applied to GSE261545 before module detection.)

## Evaluation signals

- hdWGCNA returns exactly two modules (grey and turquoise) on the normalised Visium kidney GSE261545 matrix, reproducing the reported count against SGCRNA's 114.
- Module-level spatial enrichment assigns hdWGCNA grey to tumour regions and turquoise to healthy proximal tubules.
- Approximately 71% of the comparator SGCRNA modules are non-cell-type-specific, with the remainder specific to tumour, healthy proximal tubules, glomeruli, LH-CD, and artery regions — the expected contrast in module specificity.
- Sample/column order between the expression matrix and the tissue-spot annotation table is confirmed identical before running the module-parameter correlation.
- Output file module_comparison_sgcrna_hdwgcna.csv contains both the module-count comparison and per-module spatial enrichment assignments.

## Limitations

- Module resolution is coarse: on the Visium kidney dataset hdWGCNA identified only two modules (grey and turquoise), limiting detection of cell type-specific co-expression structure.
- As a member of the WGCNA family, hdWGCNA inherits the conventional WGCNA limitations the paper lists: the scale-free topology assumption, complex manual parameter tuning, and disregard of regression line slopes.
- The module-parameter correlation step (CorPhenMod-style) is order-sensitive: expression matrix columns and annotation-table rows must match, otherwise spatial assignments are invalid.

## Evidence

- [other] order-sensitive module correlation caveat: [evidence span withheld — anchor status: not_found]
