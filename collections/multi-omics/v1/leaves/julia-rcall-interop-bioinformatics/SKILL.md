---
name: julia-rcall-interop-bioinformatics
description: Use when a Julia co-expression pipeline (SGCRNA) has produced gene module
  assignments and you need R functionality — e.g. GO enrichment, WGCNA module comparison
  — executed inside the same Julia session via RCall.
when_to_use_negative:
- Input is raw counts or unclustered expression data — run CGM preprocessing and SpectralClustering
  first to obtain module gene assignments
- The analysis can stay entirely in Julia's SGCRNA ecosystem (e.g. CGM, SpectralClustering,
  SetNetwork, DrawNetwork, CorPhenMod) and does not need R libraries such as WGCNA
  or GO tools
- Single-gene or single-module inspection with no need to annotate modules against
  reference ontologies
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SGCRNA
  role: Julia package whose CGM and SpectralClustering functions produce the co-expression
    modules whose gene lists are then passed to R for enrichment analysis
  repo: https://github.com/C37H41N2O6/SGCRNA
- name: Julia
  role: Host language (1.11 or later) for the SGCRNA pipeline and RCall interop
- name: RCall
  role: Used from Julia to execute R scripts performing Gene Ontology enrichment of
    module gene lists
- name: WGCNA
  role: Reference co-expression method whose module assignments (brown4, grey) are
    extracted and compared against SGCRNA modules via the R-side enrichment
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.04.27.650628
    title: 'SGCRNA: Spectral Clustering-Guided Co-Expression Network Analysis Without
      Scale-Free Constraints for Multi-Omic Data'
  source_content_hash: sha256:d9e973a0f26a220ed43e2eaf322e8e052dd503cf4f234ebef31217202d485866
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
  - RCall
  - SGCRNA
  - WGCNA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# julia-rcall-interop-bioinformatics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A skill for bridging Julia-based co-expression analysis (SGCRNA) with R bioinformatics libraries via RCall, so that module gene lists from spectral clustering can be subjected to R-side analyses such as Gene Ontology enrichment and comparison against WGCNA modules.

## When to use

Use when a Julia co-expression pipeline (SGCRNA) has produced gene module assignments and you need R functionality — e.g. GO enrichment, WGCNA module comparison — executed inside the same Julia session via RCall.

## When NOT to use

- Input is raw counts or unclustered expression data — run CGM preprocessing and SpectralClustering first to obtain module gene assignments
- The analysis can stay entirely in Julia's SGCRNA ecosystem (e.g. CGM, SpectralClustering, SetNetwork, DrawNetwork, CorPhenMod) and does not need R libraries such as WGCNA or GO tools
- Single-gene or single-module inspection with no need to annotate modules against reference ontologies

## Inputs

- Normalized RNA expression matrix (TSV, e.g. normalizedCounts_coding.tsv with a Symbol column and sample columns)
- Clinical/phenotype data table (e.g. SraRunTable.csv)
- Module gene-assignment tables for SGCRNA clusters (e.g. modules 45, 70) and WGCNA modules (brown4, grey) from dataset GSE114007

## Outputs

- GO enrichment results per module (significant GO terms and enriched genes, e.g. ECM-associated genes for module 70)
- Comparison/verification table mapping significant ontology terms to reported module associations (e.g. go_enrichment_verification.tsv)

## How to apply

Load the Julia-processed expression data (e.g. normalizedCounts_coding.tsv read with CSV.jl) and the module assignments produced by CGM preprocessing and SpectralClustering. Preprocess with the SGCRNA CGM function using the mode argument (:NONE, :LESS, :SIGMA, or :FTEST — :FTEST generates a plot highlighting the removed value range) and cluster with SpectralClustering to obtain clust/edge_data. Extract gene lists for the modules of interest (e.g. SGCRNA modules 45 and 70, WGCNA modules brown4 and grey from GSE114007). Use RCall from Julia to execute R scripts that run Gene Ontology enrichment on each gene list, compiling significant terms and enriched genes (e.g. ECM-associated MMP11, MMP13, ADAMTS7, ADAMTS14, LAMB1, LAMB3, FNDC1 in module 70) into a results table. Compare significant ontology terms against reported module associations (module 45 = ER, module 70 = ECM, brown4 = Golgi, grey = broad ontologies) and save combined results to a file such as go_enrichment_verification.tsv.

## Related tools

- **SGCRNA** (Julia package whose CGM and SpectralClustering functions produce the co-expression modules whose gene lists are then passed to R for enrichment analysis) — https://github.com/C37H41N2O6/SGCRNA
- **Julia** (Host language (1.11 or later) for the SGCRNA pipeline and RCall interop)
- **RCall** (Used from Julia to execute R scripts performing Gene Ontology enrichment of module gene lists)
- **WGCNA** (Reference co-expression method whose module assignments (brown4, grey) are extracted and compared against SGCRNA modules via the R-side enrichment)

## Examples

```
using SGCRNAs; CorData, GradData = CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST); clust, pos, edge_data = SpectralClustering(CorData, GradData)
```

## Evaluation signals

- The CGM preprocessing mode plot (when mode=:LESS, :SIGMA, or :FTEST is set) shows the range of values removed highlighted in red, confirming filtering was applied
- SpectralClustering returns clust, pos, edge_data; maximum(clust[d]) defines the number of modules k used in SetNetwork, and the gene lists extracted per module must partition the clustered genes
- GO enrichment output contains significant terms consistent with reported associations: module 45 = endoplasmic reticulum, module 70 = extracellular matrix (including ECM-degrading enzymes), brown4 = Golgi, grey = broad ontologies
- Enriched genes for module 70 include ECM-associated entries such as MMP11, MMP13, ADAMTS7, ADAMTS14, LAMB1, LAMB3, FNDC1
- The final verification table (e.g. go_enrichment_verification.tsv) exists with one row per module pairing agreement/disagreement against the reported GO associations

## Limitations

- RCall interop requires R and the relevant R packages to be available in the same environment as Julia 1.11+; the skill adds an R dependency to an otherwise Julia-only SGCRNA workflow
- SGCRNA's optional CGM filtering modes (:LESS, :SIGMA, :FTEST) alter which genes enter clustering, and default mode=:NONE applies no filtering, so module membership depends on this choice
- Cross-module comparison depends on module numbering (e.g. modules 45 and 70) being stable between runs of SpectralClustering

## Evidence

- [methods] dataset choice: [evidence span withheld — anchor status: not_found]
