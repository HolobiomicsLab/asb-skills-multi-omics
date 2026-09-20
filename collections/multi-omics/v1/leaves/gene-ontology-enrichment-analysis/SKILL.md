---
name: gene-ontology-enrichment-analysis
description: Use when you have gene-to-module assignments from co-expression clustering
  (e.g., SGCRNA modules from Julia's SpectralClustering, or WGCNA modules such as
  brown4/grey) and need to test which GO terms are over-represented in each module
  to validate or compare module biological identity.
when_to_use_negative:
- Input is a raw expression matrix (e.g., normalizedCounts_coding.tsv) that has not
  yet been clustered into modules via CGM/SpectralClustering or WGCNA — run the clustering
  skill first to obtain gene-to-module assignments.
- The analysis goal is module detection or module-trait correlation (CorPhenMod) rather
  than testing over-representation of ontology terms within already-defined gene sets.
edam_operation: http://edamontology.org/operation_3631
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0203
tools:
- name: SGCRNA
  role: Source of the module gene assignments (modules 45 and 70) obtained by spectral
    clustering of the GSE114007 co-expression network; the gene lists fed into GO
    enrichment
  repo: https://github.com/C37H41N2O6/SGCRNA
- name: WGCNA
  role: Provides the comparison module assignments (brown4, grey) whose GO term profiles
    are contrasted against the SGCRNA modules
- name: Julia
  role: Host environment in which SGCRNA runs and from which R-based GO enrichment
    is executed
- name: RCall
  role: Bridge used to execute R scripts (for GO enrichment analysis) within the Julia
    session
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.04.27.650628
    title: 'SGCRNA: Spectral Clustering-Guided Co-Expression Network Analysis Without
      Scale-Free Constraints for Multi-Omic Data'
  source_content_hash: sha256:3c862edd757948b8b8c1b4166c60abce8e86be0d04c3eb2a0ce07a64b26c0605
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
  edam_operation: http://edamontology.org/operation_3631
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# gene-ontology-enrichment-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Gene Ontology enrichment analysis of co-expression module gene lists, used to verify that SGCRNA and WGCNA modules derived from the GSE114007 osteoarthritis dataset recover biologically meaningful ontologies (ER for SGCRNA module 45, ECM for module 70, Golgi for WGCNA brown4).

## When to use

Use when you have gene-to-module assignments from co-expression clustering (e.g., SGCRNA modules from Julia's SpectralClustering, or WGCNA modules such as brown4/grey) and need to test which GO terms are over-represented in each module to validate or compare module biological identity.

## When NOT to use

- Input is a raw expression matrix (e.g., normalizedCounts_coding.tsv) that has not yet been clustered into modules via CGM/SpectralClustering or WGCNA — run the clustering skill first to obtain gene-to-module assignments.
- The analysis goal is module detection or module-trait correlation (CorPhenMod) rather than testing over-representation of ontology terms within already-defined gene sets.

## Inputs

- SGCRNA module gene assignments for GSE114007 (derived from normalizedCounts_coding.tsv and SraRunTable.csv)
- WGCNA module assignments (brown4, grey) from the comparison analysis
- Gene lists for SGCRNA modules 45 and 70 extracted from the assignment tables

## Outputs

- Results table of significant GO terms per module and their enriched genes (e.g., MMP11, MMP13, ADAMTS7, ADAMTS14, LAMB1, LAMB3, FNDC1 for module 70)
- go_enrichment_verification.tsv containing combined results and verification verdict comparing observed terms with reported module associations

## How to apply

Load the SGCRNA module gene assignments produced for GSE114007 (from the normalizedCounts_coding.tsv expression matrix clustered via CGM and SpectralClustering) and the WGCNA module assignments (brown4, grey) from the comparison analysis. Extract the gene lists for SGCRNA modules 45 and 70 and WGCNA modules brown4 and grey from the assignment tables. Because SGCRNA is a Julia package, execute R scripts within Julia via RCall to run GO enrichment on each of the four gene lists. Compile the significant GO terms and enriched genes — for module 70 these include ECM-associated genes MMP11, MMP13, ADAMTS7, ADAMTS14, LAMB1, LAMB3, FNDC1 — into a results table. Judge correctness by comparing each module's significant ontology terms against the reported associations: module 45 = endoplasmic reticulum, module 70 = extracellular matrix (including ECM-degrading enzymes implicated in OA progression), brown4 = Golgi, grey = broad ontologies; record agreement in go_enrichment_verification.tsv.

## Related tools

- **SGCRNA** (Source of the module gene assignments (modules 45 and 70) obtained by spectral clustering of the GSE114007 co-expression network; the gene lists fed into GO enrichment) — https://github.com/C37H41N2O6/SGCRNA
- **WGCNA** (Provides the comparison module assignments (brown4, grey) whose GO term profiles are contrasted against the SGCRNA modules)
- **Julia** (Host environment in which SGCRNA runs and from which R-based GO enrichment is executed)
- **RCall** (Bridge used to execute R scripts (for GO enrichment analysis) within the Julia session)

## Evaluation signals

- SGCRNA module 45's significant GO terms should be dominated by endoplasmic reticulum (ER) associations, matching the reported finding.
- SGCRNA module 70's significant GO terms should include extracellular matrix (ECM) categories, with ECM-degrading enzymes MMP11, MMP13, ADAMTS7 and ADAMTS14 plus ECM structural genes LAMB1, LAMB3 and FNDC1 among the enriched genes.
- WGCNA brown4 should recover Golgi-associated terms and grey should show broad ontologies, consistent with the reported comparison associations.
- The verification verdict recorded in go_enrichment_variance.tsv-style output (go_enrichment_verification.tsv) should document agreement or disagreement between observed terms and each module's reported association.
- Each of the four module gene lists (45, 70, brown4, grey) should yield a non-empty enrichment result table with identifiable significant terms.

## Limitations

- GO enrichment validity depends on the module assignments coming from the same clustering run (GSE114007 via SGCRNA's CGM/SpectralClustering or the WGCNA comparison); applying it to arbitrary gene sets is outside the verified scope.
- The enrichment is run via RCall from Julia, so R package availability and gene identifier compatibility (gene symbols from normalizedCounts_coding.tsv) are prerequisites for reproducibility.

## Evidence

- [methods] Dataset used: [evidence span withheld — anchor status: not_found]
