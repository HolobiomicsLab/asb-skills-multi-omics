---
name: count-based-differential-expression-testing
description: Use when RNA-seq reads (or their Salmon quantification output) must be
  tested for differential expression between defined groups, e.g. the simulated disease
  vs normal samples in the eduomics RNA subworkflow.
when_to_use_negative:
- Input is still raw FASTQ that has not been quantified — quantify with Salmon and
  summarize transcript abundances to gene-level counts before running DESeq2
- The task is DNA variant calling rather than RNA differential expression — use the
  eduomics DNA branch (SimuSCoP simulation plus variant calling validation) instead
- Only transcript-level abundances are needed without gene-level differential testing
  — importing and summarizing via tximport for other purposes does not require the
  DESeq2 testing step
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: DESeq2
  role: Performs negative-binomial differential expression testing on the gene-level
    count matrix between simulated disease and normal groups
  repo: https://github.com/lescai-teaching/eduomics
- name: Salmon
  role: Quantifies transcript expression from the simulated FASTQ reads; its abundances
    are imported and summarized to gene-level counts for DESeq2
  repo: https://github.com/lescai-teaching/eduomics
- name: Nextflow
  role: Orchestrates the QUANTIFY_DEANALYSIS_ENRICH_VALIDATE RNA subworkflow that
    wraps the quantification and DESeq2 steps
  repo: https://github.com/lescai-teaching/eduomics
- name: Polyester
  role: Generates the FASTQ files matching the simulated count matrices that serve
    as the input to the quantification and DE testing
- name: clusterProfiler
  role: Consumes the DE gene list from DESeq2 for downstream GO over-representation
    analysis across BP, MF and CC ontologies
  repo: https://github.com/lescai-teaching/eduomics
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:0d27252b98ec1f503c1f0b47caa495dd5ae0d9fbec8d3a5f6c2eb4d5d44a5915
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lescai-teaching/eduomics
  tools:
  - DESeq2
  - Nextflow
  - Polyester
  - Salmon
  - clusterProfiler
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# count-based-differential-expression-testing

## Summary

Runs DESeq2 negative-binomial differential expression testing on gene-level counts quantified by Salmon from Polyester-simulated RNA-seq reads, producing DE gene lists and diagnostics that are later validated for the eduomics educational storyline.

## When to use

Use when RNA-seq reads (or their Salmon quantification output) must be tested for differential expression between defined groups, e.g. the simulated disease vs normal samples in the eduomics RNA subworkflow.

## When NOT to use

- Input is still raw FASTQ that has not been quantified — quantify with Salmon and summarize transcript abundances to gene-level counts before running DESeq2
- The task is DNA variant calling rather than RNA differential expression — use the eduomics DNA branch (SimuSCoP simulation plus variant calling validation) instead
- Only transcript-level abundances are needed without gene-level differential testing — importing and summarizing via tximport for other purposes does not require the DESeq2 testing step

## Inputs

- Polyester-simulated paired-end RNA-seq FASTQ files (disease and normal sample groups)
- Salmon transcript-level quantification outputs, summarized to gene-level counts
- Sample metadata / design table specifying the simulated disease vs normal group contrast

## Outputs

- deseq2_results.tsv — full DESeq2 statistics table for all genes
- deseq2_de_genes.txt — significant differentially expressed gene identifiers
- DESeq2 diagnostic and result plots (PCA, MA, volcano-style) as PDF files
- DE gene list feeding clusterProfiler GO enrichment (enrichment_results.rds)

## How to apply

First quantify the Polyester-simulated FASTQ reads against the reference transcriptome with Salmon's quasi-mapping-based quantification, then import the Salmon transcript-level abundances and summarize them to gene-level counts for DESeq2. Run DESeq2's negative-binomial differential expression testing between the simulated disease and normal sample groups (the count matrices are built with known differential expression patterns so the ground truth is known). Write the full statistics table to deseq2_results.tsv and the significant DE gene identifiers to deseq2_de_genes.txt, and generate the DESeq2 diagnostic and result plots (PCA, MA and volcano-style plots) as PDF files. The DE gene list then feeds clusterProfiler GO over-representation analysis, and the RNA validation checks that the simulated data yield a statistically significant and coherent differential expression pattern before the simulation counts as educational output.

## Related tools

- **DESeq2** (Performs negative-binomial differential expression testing on the gene-level count matrix between simulated disease and normal groups) — https://github.com/lescai-teaching/eduomics
- **Salmon** (Quantifies transcript expression from the simulated FASTQ reads; its abundances are imported and summarized to gene-level counts for DESeq2) — https://github.com/lescai-teaching/eduomics
- **Nextflow** (Orchestrates the QUANTIFY_DEANALYSIS_ENRICH_VALIDATE RNA subworkflow that wraps the quantification and DESeq2 steps) — https://github.com/lescai-teaching/eduomics
- **Polyester** (Generates the FASTQ files matching the simulated count matrices that serve as the input to the quantification and DE testing)
- **clusterProfiler** (Consumes the DE gene list from DESeq2 for downstream GO over-representation analysis across BP, MF and CC ontologies) — https://github.com/lescai-teaching/eduomics

## Examples

```
nextflow run eduomics -profile docker --input samplesheet.csv --genome GATK.GRCh38 --outdir <OUTDIR>
```

## Evaluation signals

- deseq2_results.tsv contains the full statistics table and deseq2_de_genes.txt contains a non-empty list of significant DE gene identifiers
- The recovered DE gene list matches the known ground-truth differential expression pattern injected into the simulated count matrices
- Diagnostic plots (PCA, MA, volcano-style PDFs) show separation of the disease and normal groups consistent with the simulated effect
- The DE genes yield detectable and biologically plausible GO enrichment (dotplot and cnetplot PNGs generated from enrichment_results.rds)
- The pipeline's RNA validation check passes and the pass/fail outcome is recorded in validation_result.txt

## Limitations

- Only simulations passing validation are included in the final output, so a weak or incoherent simulated DE pattern will fail validation_result.txt
- The pipeline focuses on single chromosomes (typically chr22) and human reference data (GATK.GRCh38), which bounds the scale and scope of the DE testing
- Ground truth is only trustworthy because counts are simulated with known differential expression patterns; on real data the DE results cannot be validated against known truth in the same way

## Evidence

- [intro] DESeq2 analysis on Salmon gene-level counts: "Runs Salmon to quantify transcript expression."
- [intro] DESeq2 identifies DE genes: "Performs DESeq2 analysis to identify differentially expressed (DE) genes."
- [readme] Known ground truth in simulated counts: [evidence span withheld — anchor status: not_found]
- [intro] RNA validation checks significance and coherence: "Verifies that the simulated data produce a statistically significant and coherent differential expression pattern, with detectable and biologically plausible functional enrichment."
- [readme] Educational purpose of controlled DE patterns: [evidence span withheld — anchor status: not_found]
