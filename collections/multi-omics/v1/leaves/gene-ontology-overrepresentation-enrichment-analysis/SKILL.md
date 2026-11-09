---
name: gene-ontology-overrepresentation-enrichment-analysis
description: Use when you have a list of significant DE gene identifiers (e.g. deseq2_de_genes.txt)
  from simulated RNA-seq data and need GO Biological Process, Molecular Function and
  Cellular Component over-representation results.
when_to_use_negative:
- The input is a raw Salmon transcript-level abundance table rather than a filtered
  list of significant DE genes — summarize to gene-level counts and run DESeq2 first
- The goal is DNA variant calling validation on SimuSCoP-simulated reads; GO enrichment
  belongs only to the RNA differential expression branch
- The gene list comes from real (non-simulated) data where no ground-truth DE pattern
  exists for validation against
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0089
tools:
- name: clusterProfiler
  role: Performs over-representation analysis of the DE gene list across Biological
    Process, Molecular Function and Cellular Component GO ontologies; renders dotplot
    and cnetplot PNGs
  repo: github.com/lescai-teaching/eduomics
- name: DESeq2
  role: Upstream negative-binomial differential expression testing that produces the
    significant DE gene list used as ORA input
  repo: github.com/lescai-teaching/eduomics
- name: Salmon
  role: Upstream quasi-mapping quantification of Polyester-simulated FASTQ reads whose
    transcript abundances feed the gene-level count matrix
  repo: github.com/lescai-teaching/eduomics
- name: Nextflow
  role: Orchestrates the QUANTIFY_DEANALYSIS_ENRICH_VALIDATE subworkflow that embeds
    the enrichment step
  repo: github.com/lescai-teaching/eduomics
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:5cf0f66891dccf0ea0007288b8aa9fbbfa860421232eeb3a9f963e9b772756b3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - Nextflow
  - Salmon
  - clusterProfiler
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# gene-ontology-overrepresentation-enrichment-analysis

## Summary

Runs Gene Ontology over-representation analysis (clusterProfiler) on differentially expressed genes from the eduomics RNA simulation workflow to test whether the known DE pattern yields detectable and biologically coherent functional enrichment.

## When to use

Use when you have a list of significant DE gene identifiers (e.g. deseq2_de_genes.txt) from simulated RNA-seq data and need GO Biological Process, Molecular Function and Cellular Component over-representation results.

## When NOT to use

- The input is a raw Salmon transcript-level abundance table rather than a filtered list of significant DE genes — summarize to gene-level counts and run DESeq2 first
- The goal is DNA variant calling validation on SimuSCoP-simulated reads; GO enrichment belongs only to the RNA differential expression branch
- The gene list comes from real (non-simulated) data where no ground-truth DE pattern exists for validation against

## Inputs

- Significant DE gene identifier list (deseq2_de_genes.txt) from DESeq2 testing of Salmon-quantified Polyester-simulated reads
- Full DESeq2 statistics table (deseq2_results.tsv)
- Gene Ontology annotation for the human reference transcriptome (GATK.GRCh38)

## Outputs

- enrichment_results.rds (clusterProfiler enrichment object for BP, MF and CC ontologies)
- dotplot and cnetplot visualizations as PNG files
- validation_result.txt recording pass/fail of the RNA enrichment and DE coherence checks

## How to apply

In the eduomics QUANTIFY_DEANALYSIS_ENRICH_VALIDATE subworkflow, first obtain significant DE gene identifiers by running DESeq2's negative-binomial testing between simulated disease and normal groups (writing deseq2_results.tsv and deseq2_de_genes.txt). Feed that DE gene list into clusterProfiler's over-representation analysis across all three GO ontologies (BP, MF, CC), saving the enrichment object as enrichment_results.rds. Render dotplot and cnetplot visualizations as PNG files for inspection. The rationale is educational: the simulation injects a known DE pattern, so GO enrichment should recover biologically plausible terms for the GO-annotated genes selected during simulation. Finally apply the RNA validation thresholds, confirming the functional enrichment is detectable and coherent, and record pass/fail in validation_result.txt.

## Related tools

- **clusterProfiler** (Performs over-representation analysis of the DE gene list across Biological Process, Molecular Function and Cellular Component GO ontologies; renders dotplot and cnetplot PNGs) — github.com/lescai-teaching/eduomics
- **DESeq2** (Upstream negative-binomial differential expression testing that produces the significant DE gene list used as ORA input) — github.com/lescai-teaching/eduomics
- **Salmon** (Upstream quasi-mapping quantification of Polyester-simulated FASTQ reads whose transcript abundances feed the gene-level count matrix) — github.com/lescai-teaching/eduomics
- **Nextflow** (Orchestrates the QUANTIFY_DEANALYSIS_ENRICH_VALIDATE subworkflow that embeds the enrichment step) — github.com/lescai-teaching/eduomics

## Examples

```
nextflow run eduomics -profile docker --input samplesheet.csv --genome GATK.GRCh38
```

## Evaluation signals

- validation_result.txt records a pass: the functional enrichment is detectable and biologically plausible alongside a statistically significant, coherent DE pattern
- enrichment_results.rds contains results for all three GO ontologies (BP, MF, CC) on the DE gene list
- Dotplot and cnetplot PNG files are rendered and show GO terms consistent with the simulated genes' known GO annotations
- The ORA input matches deseq2_de_genes.txt exactly — the significant gene identifiers written by the DESeq2 step
- Only simulations passing the RNA validation thresholds are included in the final pipeline output

## Limitations

- Validation depends on the simulation: if the injected DE pattern is weak, GO enrichment may not be detectable and the simulation fails the validation gate
- The pipeline focuses on single chromosomes (typically chr22) to reduce computation, restricting the GO term space available for enrichment
- The enrichment step is only meaningful for genes with GO annotation; genes selected for simulation must be GO-annotated for the check to be coherent

## Evidence

- [intro] GO enrichment on DE genes: "Conducts GO enrichment analysis on DE genes."
- [intro] RNA validation rationale: "Verifies that the simulated data produce a statistically significant and coherent differential expression pattern, with detectable and biologically plausible functional enrichment."
- [intro] GO-annotated gene selection: "Identifies genes suitable for differential expression based on Gene Ontology (GO) annotation."
- [readme] Educational ground truth: [evidence span withheld — anchor status: not_found]
