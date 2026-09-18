---
name: cross-method-module-annotation-verification
description: Use when you have gene-module assignment tables from two co-expression
  methods (e.g., SGCRNA spectral clusters and WGCNA brown4/grey modules) plus a paper's
  reported GO term associations, and you need to independently rerun GO enrichment
  per module and record agreement.
when_to_use_negative:
- You only have a single method's modules and no reported annotations to verify against
  — plain enrichment analysis, not cross-method verification, is needed.
- Module assignments have not yet been computed; run CGM + SpectralClustering (SGCRNA)
  and WGCNA first.
- The claimed annotations are non-GO (e.g., KEGG or trait correlations); the CorPhenMod
  module-phenotype correlation workflow is the appropriate skill instead.
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SGCRNA
  role: Provides the CGM preprocessing, SpectralClustering, SetNetwork/DrawNetwork,
    and CorPhenMod Julia functions that generate the module assignments whose GO annotations
    are being verified.
  repo: https://github.com/C37H41N2O6/SGCRNA
- name: WGCNA
  role: Baseline co-expression method whose brown4 and grey modules are extracted
    for side-by-side GO enrichment comparison with SGCRNA modules.
- name: Julia
  role: Host environment (Julia 1.11 or later) in which SGCRNAs functions run and
    in which RCall is used to execute GO enrichment scripts.
  repo: https://github.com/C37H41N2O6/SGCRNA
- name: RCall
  role: Julia package used to execute R scripts that perform the Gene Ontology enrichment
    analysis of each module's gene list, since SGCRNA itself does not expose GO enrichment
    functions.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1101/2025.04.27.650628
    title: 'SGCRNA: Spectral Clustering-Guided Co-Expression Network Analysis Without
      Scale-Free Constraints for Multi-Omic Data'
  source_content_hash: sha256:2e21ea787feb9a1e1155336c0a113ca4492082fa003b2ecb111006cf67182c72
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
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.25
n_claims: 8
---

# cross-method-module-annotation-verification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A verification skill that independently reproduces Gene Ontology (GO) enrichment for co-expression modules computed by two methods (SGCRNA spectral clustering and WGCNA) on the GSE114007 osteoarthritis transcriptomic dataset, then checks reproduced terms against the paper's reported associations (SGCRNA module 45 = ER, module 70 = ECM, WGCNA brown4 = Golgi, grey = broad ontologies).

## When to use

Use when you have gene-module assignment tables from two co-expression methods (e.g., SGCRNA spectral clusters and WGCNA brown4/grey modules) plus a paper's reported GO term associations, and you need to independently rerun GO enrichment per module and record agreement.

## When NOT to use

- You only have a single method's modules and no reported annotations to verify against — plain enrichment analysis, not cross-method verification, is needed.
- Module assignments have not yet been computed; run CGM + SpectralClustering (SGCRNA) and WGCNA first.
- The claimed annotations are non-GO (e.g., KEGG or trait correlations); the CorPhenMod module-phenotype correlation workflow is the appropriate skill instead.

## Inputs

- SGCRNA module gene assignment table derived from GSE114007 (normalizedCounts_coding.tsv + SraRunTable.csv)
- WGCNA module assignment table (brown4, grey modules) from the comparison analysis
- Per-module gene symbol lists (e.g., SGCRNA modules 45 and 70, WGCNA brown4 and grey)
- Reported GO associations used as the verification target (module 45 = ER, module 70 = ECM, brown4 = Golgi, grey = broad)

## Outputs

- GO enrichment results table of significant ontology terms and enriched genes per module
- go_enrichment_verification.tsv combining per-module GO results and the agreement verdict

## How to apply

First compute or load module assignments: preprocess the GSE114007 RNA expression matrix (normalizedCounts_coding.tsv) with SGCRNAs.CGM (optionally mode=:FTEST to highlight and remove outlying value ranges) and cluster with SpectralClustering, and obtain the WGCNA module assignments (brown4, grey) from the comparison analysis. Extract the gene lists for SGCRNA modules 45 and 70 and WGCNA modules brown4 and grey from the assignment tables. Since SGCRNA itself does not provide GO enrichment, use RCall to execute R scripts within Julia that run Gene Ontology enrichment analysis on each of the four gene lists. Compile the significant GO terms and enriched genes into a results table — for module 70 expect ECM-associated genes such as MMP11, MMP13, ADAMTS7, ADAMTS14, LAMB1, LAMB3, FNDC1. Finally, compare each module's significant ontology terms against the reported associations (module 45 = endoplasmic reticulum, module 70 = ECM and ECM-degrading enzymes implicated in OA progression, brown4 = Golgi, grey = broad ontologies) and write the combined results and verification verdict to go_enrichment_verification.tsv.

## Related tools

- **SGCRNA** (Provides the CGM preprocessing, SpectralClustering, SetNetwork/DrawNetwork, and CorPhenMod Julia functions that generate the module assignments whose GO annotations are being verified.) — https://github.com/C37H41N2O6/SGCRNA
- **WGCNA** (Baseline co-expression method whose brown4 and grey modules are extracted for side-by-side GO enrichment comparison with SGCRNA modules.)
- **Julia** (Host environment (Julia 1.11 or later) in which SGCRNAs functions run and in which RCall is used to execute GO enrichment scripts.) — https://github.com/C37H41N2O6/SGCRNA
- **RCall** (Julia package used to execute R scripts that perform the Gene Ontology enrichment analysis of each module's gene list, since SGCRNA itself does not expose GO enrichment functions.)

## Examples

```
CorData, GradData = CGM(Data.Symbol, Matrix(Data[:,5:end]), mode=:FTEST); clust, pos, edge_data = SpectralClustering(CorData, GradData);
```

## Evaluation signals

- Reproduced GO terms for SGCRNA module 45 are dominated by endoplasmic reticulum (ER) associations, matching the reported finding.
- Reproduced GO terms for SGCRNA module 70 include extracellular matrix (ECM) categories and enriched ECM/ECM-degrading genes such as MMP11, MMP13, ADAMTS7, ADAMTS14, LAMB1, LAMB3, and FNDC1.
- WGCNA brown4 enriches Golgi-associated terms and grey shows broad ontologies, consistent with the reported comparison associations.
- go_enrichment_verification.tsv exists and contains, for each of the four modules, significant GO terms plus an explicit agreement verdict against the reported associations.
- Module assignments reproduce from the documented pipeline: CGM (optionally mode=:FTEST) followed by SpectralClustering on the GSE114007 normalized counts.

## Limitations

- SGCRNAs provides clustering, network visualisation, and module-phenotype correlation (CorPhenMod) but no GO enrichment functions, so GO analysis depends on RCall/R rather than being self-contained in the Julia package.
- CGM's optional mode argument (:NONE default; :LESS, :SIGMA, :FTEST) changes which values are removed, which can shift module composition and hence GO enrichment results.
- The package repository has no changelog ('No changelog found'), so version-to-version behaviour of SGCRNAs functions is not traceable.

## Evidence

- [readme] README shows loading the GSE114007 expression matrix and SraRunTable.csv used to derive module assignments: [evidence span withheld — anchor status: not_found]
