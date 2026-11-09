---
name: enrichment-result-visualization-and-interpretation
description: Use when a differential expression step (e.g. DESeq2 on Salmon gene-level
  counts) has produced a DE gene list and you need GO over-representation results
  with dotplot/cnetplot figures and a coherence check.
when_to_use_negative:
- Input is raw FASTQ or transcript-level abundances that have not yet been summarized
  to gene-level counts and tested with DESeq2 — quantification and differential analysis
  must come first
- The goal is variant calling or DNA simulation validation, which follows the separate
  SimuSCoP-based DNA branch of eduomics rather than the RNA enrichment branch
- The gene list is too small or not derived from a differential expression contrast,
  so over-representation analysis would lack statistical power and biological meaning
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0203
tools:
- name: clusterProfiler
  role: Performs GO over-representation analysis of the DE gene list across Biological
    Process, Molecular Function and Cellular Component ontologies; provides dotplot
    and cnetplot visualization functions
- name: DESeq2
  role: Upstream producer of the significant DE gene identifiers that feed the enrichment
    analysis
- name: Salmon
  role: Upstream quantification of Polyester-simulated reads whose gene-level counts
    feed DESeq2 and hence the enrichment step
- name: Nextflow
  role: Orchestrates the QUANTIFY_DEANALYSIS_ENRICH_VALIDATE subworkflow that wraps
    the enrichment and validation steps
  repo: https://github.com/lescai-teaching/eduomics
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:f960367bdd70d0f7878c23661829e7564f1ba8a79de6604fdcddb7107a85a8bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lescai-teaching/eduomics
  tools:
  - DESeq2
  - Nextflow
  - Salmon
  - clusterProfiler
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# enrichment-result-visualization-and-interpretation

## Summary

Run Gene Ontology over-representation analysis on a list of differentially expressed genes with clusterProfiler across BP, MF and CC ontologies, save the enrichment object, render dotplot and cnetplot visualizations, and judge whether the enrichment is detectable and biologically coherent.

## When to use

Use when a differential expression step (e.g. DESeq2 on Salmon gene-level counts) has produced a DE gene list and you need GO over-representation results with dotplot/cnetplot figures and a coherence check.

## When NOT to use

- Input is raw FASTQ or transcript-level abundances that have not yet been summarized to gene-level counts and tested with DESeq2 — quantification and differential analysis must come first
- The goal is variant calling or DNA simulation validation, which follows the separate SimuSCoP-based DNA branch of eduomics rather than the RNA enrichment branch
- The gene list is too small or not derived from a differential expression contrast, so over-representation analysis would lack statistical power and biological meaning

## Inputs

- Significant differentially expressed gene identifier list (deseq2_de_genes.txt) from DESeq2 negative-binomial testing on Salmon-derived gene-level counts
- Gene Ontology annotation for the reference organism (human, GATK.GRCh38 background in eduomics)
- Full DESeq2 statistics table (deseq2_results.tsv) for context on DE gene ranking

## Outputs

- clusterProfiler enrichment object serialized as enrichment_results.rds
- Dotplot visualization of enriched GO terms (PNG)
- cnetplot visualization linking DE genes to enriched GO terms (PNG)
- Pass/fail enrichment coherence outcome written to validation_result.txt

## How to apply

Take the significant DE gene identifiers produced by DESeq2's negative-binomial testing (written to deseq2_de_genes.txt in the eduomics RNA subworkflow) as the query set for clusterProfiler over-representation analysis. Run the ORA separately across the Biological Process, Molecular Function and Cellular Component GO ontologies, and serialize the enrichment object as enrichment_results.rds so downstream steps can reload it. Render a dotplot (showing gene ratio and enrichment significance per GO term) and a cnetplot (showing DE genes mapped to enriched terms) as PNG files for interpretation. Finally apply the RNA validation thresholds: the functional enrichment must be detectable and biologically plausible, and the outcome is recorded in validation_result.txt; in the eduomics educational setting the ground-truth injected DE pattern means enriched terms should correspond to the functions of genes deliberately made differentially expressed.

## Related tools

- **clusterProfiler** (Performs GO over-representation analysis of the DE gene list across Biological Process, Molecular Function and Cellular Component ontologies; provides dotplot and cnetplot visualization functions)
- **DESeq2** (Upstream producer of the significant DE gene identifiers that feed the enrichment analysis)
- **Salmon** (Upstream quantification of Polyester-simulated reads whose gene-level counts feed DESeq2 and hence the enrichment step)
- **Nextflow** (Orchestrates the QUANTIFY_DEANALYSIS_ENRICH_VALIDATE subworkflow that wraps the enrichment and validation steps) — https://github.com/lescai-teaching/eduomics

## Examples

```
Rscript -e 'library(clusterProfiler); ego <- enrichGO(gene=readLines("deseq2_de_genes.txt"), OrgDb="org.Hs.eg.db", ont="BP", readable=TRUE); saveRDS(ego,"enrichment_results.rds"); png("dotplot.png"); print(dotplot(ego)); dev.off(); png("cnetplot.png"); print(cnetplot(ego)); dev.off()'
```

## Evaluation signals

- enrichment_results.rds exists and contains ORA results for all three GO ontologies (BP, MF, CC)
- dotplot and cnetplot PNG files are generated and the enriched terms map to the ground-truth functions of genes deliberately made differentially expressed
- validation_result.txt records a pass outcome, indicating the enrichment is detectable and biologically plausible under the pipeline's RNA validation checks
- The DE gene list input (deseq2_de_genes.txt) is non-empty and consistent with the significant rows of deseq2_results.tsv

## Limitations

- Only simulations passing validation are included in the final output, so an enrichment that fails the biologically-plausible check invalidates the whole simulated dataset
- The pipeline focuses on single chromosomes (typically chr22) to reduce computational requirements, which limits the universe of genes available for GO enrichment
- Enrichment quality depends on the injected DE pattern being biologically plausible; random or incoherent count matrices will not yield interpretable GO terms

## Evidence

- [other] clusterProfiler ORA on DE gene list across ontologies: [evidence span withheld — anchor status: not_found]
- [intro] GO enrichment on DE genes: "Conducts GO enrichment analysis on DE genes."
- [intro] validation of RNA simulation results: "Verifies that the simulated data produce a statistically significant and coherent differential expression pattern, with detectable and biologically plausible functional enrichment."
- [readme] README RNA branch description: [evidence span withheld — anchor status: not_found]
- [intro] only passing simulations in output: "Only simulations passing validation are included in the final output"
