---
name: module-spatial-enrichment-mapping
description: Use when gene co-expression modules have already been clustered (SGCRNA
  SpectralClustering or hdWGCNA) and you need to attribute each module to tissue regions
  or traits, e.g. spatial spots in a Visium dataset.
when_to_use_negative:
- No modules exist yet — run CGM/SpectralClustering (SGCRNA) or hdWGCNA first; CorPhenMod
  requires a new_clust module assignment
- Annotation table rows are not ordered to match expression-matrix columns — mismatched
  sample/column order invalidates the module-parameter correlations
- Input is raw unnormalised counts without a standard expression-normalisation workflow
  (e.g. Seurat spatial pipeline) applied first
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SGCRNA
  role: Provides CorPhenMod for module-phenomenon correlation and CGM/SpectralClustering
    for the upstream modules
  repo: https://github.com/C37H41N2O6/SGCRNA
- name: Julia
  role: Runtime environment in which the SGCRNAs package functions (CGM, SpectralClustering,
    CorPhenMod) are executed
- name: hdWGCNA
  role: Alternative WGCNA variant for single-cell/spatial data whose modules can be
    compared against SGCRNA modules in the enrichment table
- name: WGCNA
  role: Baseline co-expression framework whose module-trait association workflow CorPhenMod
    extends
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.04.27.650628
    title: 'SGCRNA: Spectral Clustering-Guided Co-Expression Network Analysis Without
      Scale-Free Constraints for Multi-Omic Data'
  source_content_hash: sha256:60c60fc19c521169c3bb694c89d32115bfcbc78a99f5156943738e52fe13c7d3
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
  - Julia
  - SGCRNA
  - WGCNA
  - hdWGCNA
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# module-spatial-enrichment-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Correlates identified co-expression modules (e.g. from SGCRNA or WGCNA) with sample-level annotations or tissue-spot labels using the SGCRNAs CorPhenMod function, assigning each module a spatial or phenotype pattern such as tumour, proximal tubule, glomeruli, LH-CD, or artery.

## When to use

Use when gene co-expression modules have already been clustered (SGCRNA SpectralClustering or hdWGCNA) and you need to attribute each module to tissue regions or traits, e.g. spatial spots in a Visium dataset.

## When NOT to use

- No modules exist yet — run CGM/SpectralClustering (SGCRNA) or hdWGCNA first; CorPhenMod requires a new_clust module assignment
- Annotation table rows are not ordered to match expression-matrix columns — mismatched sample/column order invalidates the module-parameter correlations
- Input is raw unnormalised counts without a standard expression-normalisation workflow (e.g. Seurat spatial pipeline) applied first

## Inputs

- Normalised gene expression matrix (genes x samples/spots), e.g. a Seurat-normalised Visium matrix or TSV such as normalizedCounts_coding.tsv
- Module assignment vector/new_clust from SpectralClustering (SGCRNAs)
- Sample or tissue-spot annotation table, e.g. SraRunTable.csv or Visium tissue-spot labels
- edge_data from SetNetwork/DrawNetwork providing gene names in module order

## Outputs

- Module-phenotype correlation figure, e.g. Result/Fig/CorPhenMod.png
- Per-module spatial enrichment assignments (e.g. tumour, proximal tubule, glomeruli, LH-CD, artery)
- Module comparison table such as module_comparison_sgcrna_hdwgcna.csv

## How to apply

After clustering (e.g. SGCRNA CGM with mode=:FTEST followed by SpectralClustering), align the expression matrix column order with the annotation table by innerjoining on gene symbols in edge_data order and sorting the phenotype table (sort!(Phen)), ensuring sample/column order matches between expression matrix and annotations. Call SGCRNAs.CorPhenMod with the expression columns, the selected annotation columns (e.g. Phen[:,[2,3,5]]), the new_clust module assignments, and an output PNG path; the function computes module-parameter correlations (CorPhenMod-style module-level enrichment). For spatial data, tabulate per-module enrichment against tissue-spot labels (tumour, healthy proximal tubules, glomeruli, LH-CD, artery) and record the dominant spatial pattern per module, saving results such as module_comparison_sgcrna_hdwgcna.csv.

## Related tools

- **SGCRNA** (Provides CorPhenMod for module-phenomenon correlation and CGM/SpectralClustering for the upstream modules) — https://github.com/C37H41N2O6/SGCRNA
- **Julia** (Runtime environment in which the SGCRNAs package functions (CGM, SpectralClustering, CorPhenMod) are executed)
- **hdWGCNA** (Alternative WGCNA variant for single-cell/spatial data whose modules can be compared against SGCRNA modules in the enrichment table)
- **WGCNA** (Baseline co-expression framework whose module-trait association workflow CorPhenMod extends)

## Examples

```
Data = innerjoin(Data, DataFrame(Symbol=names(edge_data)), on=:Symbol, order=:right); CorPhenMod(Data[:,5:end], Phen[:,[2,3,5]], new_clust, "Result/Fig/CorPhenMod.png");
```

## Evaluation signals

- The output PNG (e.g. CorPhenMod.png) is generated and the expression matrix columns match the annotation rows after the innerjoin and sort! steps
- Every module receives exactly one dominant enrichment label; in the reference Visium kidney run ~71% of SGCRNA modules were non-cell-type-specific, with the remainder specific to tumour, proximal tubules, glomeruli, LH-CD, and artery
- hdWGCNA comparison reproduces the expected pattern (grey=tumour, turquoise=proximal tubule; SGCRNA 114 modules vs hdWGCNA two modules) in module_comparison_sgcrna_hdwgcna.csv
- Correlation magnitude/direction per module is consistent with known tissue-spot annotations rather than noise (order-sensitive check on column/row alignment)

## Limitations

- Module-phenomenon correlation is order-sensitive: sample/column order must match between the expression matrix and the annotation table, otherwise correlations are meaningless
- Upstream limitations of conventional WGCNA (scale-free assumption, parameter tuning, neglect of regression slopes) propagate to module definitions if hdWGCNA/WGCNA rather than SGCRNA is used
- Most modules may be non-specific — in the Visium kidney analysis ~71% of SGCRNA modules showed no cell-type specificity, so absence of spatial enrichment is a common outcome

## Evidence

- [readme] CorPhenMod(Data[:,5:end], Phen[:,[2,3,5]], new_clust, "Result/Fig/CorPhenMod.png");: [evidence span withheld — anchor status: not_found]
- [other] SGCRNA partitioned the data into 114 modules whereas hdWGCNA identified only two: [evidence span withheld — anchor status: not_found]
