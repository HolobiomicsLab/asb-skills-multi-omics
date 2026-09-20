---
name: rna-seq-transcript-abundance-quantification
description: Use when starting from RNA-seq FASTQ files (e.g. Polyester-simulated
  paired-end reads) and transcript-level abundance estimates or a gene-level count
  matrix are needed for differential expression analysis.
when_to_use_negative:
- Input is already a gene-level count matrix (e.g. the eduomics simulated count matrices)
  — skip quantification and proceed directly to DESeq2
- The goal is variant detection from DNA reads — use the eduomics DNA branch (SimuSCoP
  simulation with variant calling validation), not transcript quantification
edam_operation: http://edamontology.org/operation_3258
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3522
tools:
- name: Salmon
  role: Performs quasi-mapping-based transcript expression quantification from the
    simulated FASTQ reads
  repo: https://github.com/lescai-teaching/eduomics
- name: Polyester
  role: Upstream simulator that generates the FASTQ files matching the known count
    matrices that Salmon quantifies
  repo: https://github.com/lescai-teaching/eduomics
- name: DESeq2
  role: Downstream consumer of the gene-level summarized counts for negative-binomial
    differential expression testing
- name: Nextflow
  role: Orchestrates the quantification step within the eduomics RNA subworkflow
  repo: https://github.com/lescai-teaching/eduomics
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:3f60685d0fa2f3cf6be3beb3bccfb460f55789072072b5970d9934ef863262a3
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
  edam_operation: http://edamontology.org/operation_3258
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3522
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# RNA-seq transcript abundance quantification

## Summary

Quantify transcript abundance from RNA-seq FASTQ reads using Salmon's quasi-mapping-based quantification, then summarize transcript-level abundances to gene-level counts suitable for DESeq2 differential expression testing. In eduomics this step bridges Polyester-simulated reads and the downstream DE/enrichment analysis.

## When to use

Use when starting from RNA-seq FASTQ files (e.g. Polyester-simulated paired-end reads) and transcript-level abundance estimates or a gene-level count matrix are needed for differential expression analysis.

## When NOT to use

- Input is already a gene-level count matrix (e.g. the eduomics simulated count matrices) — skip quantification and proceed directly to DESeq2
- The goal is variant detection from DNA reads — use the eduomics DNA branch (SimuSCoP simulation with variant calling validation), not transcript quantification

## Inputs

- Paired-end FASTQ files (e.g. Polyester-simulated RNA-seq reads)
- Reference transcriptome (transcript sequences/index) consistent with the simulation reference bundle

## Outputs

- Per-sample Salmon transcript-level abundance files (quant.sf with NumReads/TPM)
- Gene-level count matrix summarized from Salmon abundances for DESeq2 input
- Salmon run metadata/logs (mapping and quantification diagnostics)

## How to apply

Run Salmon in quasi-mapping mode against the reference transcriptome to quantify transcript expression from the FASTQ files, producing per-sample transcript-level abundance tables (quant.sf-style output with NumReads/TPM). Then import the Salmon transcript-level abundances and summarize them to gene-level counts, since DESeq2's negative-binomial testing operates on gene-level counts rather than transcript-level estimates. In the eduomics RNA branch this step sits between Polyester read simulation ('Uses Polyester to generate FASTQ files matching the count matrices') and DESeq2 differential testing, so the output count matrix must preserve the simulated disease/normal group structure. Keep the quantification chromosome-restricted when the underlying simulation is chromosome-specific (eduomics typically chr22) so abundances match the reference bundle delivered with the dataset.

## Related tools

- **Salmon** (Performs quasi-mapping-based transcript expression quantification from the simulated FASTQ reads) — https://github.com/lescai-teaching/eduomics
- **Polyester** (Upstream simulator that generates the FASTQ files matching the known count matrices that Salmon quantifies) — https://github.com/lescai-teaching/eduomics
- **DESeq2** (Downstream consumer of the gene-level summarized counts for negative-binomial differential expression testing)
- **Nextflow** (Orchestrates the quantification step within the eduomics RNA subworkflow) — https://github.com/lescai-teaching/eduomics

## Examples

```
nextflow run eduomics -profile docker --input samplesheet.csv --genome GATK.GRCh38 --outdir <OUTDIR>
```

## Evaluation signals

- Salmon quantification output files (quant.sf-style tables with NumReads/TPM) exist for every simulated sample in the samplesheet
- Gene-level summarization yields an integer count matrix whose sample columns match the simulated disease/normal group structure
- Downstream DESeq2 runs without errors and writes deseq2_results.tsv plus deseq2_de_genes.txt
- The quantified abundances reproduce the known differential expression pattern injected during count-matrix simulation, as confirmed by the RNA validation checks recorded in validation_result.txt
- Only simulations passing validation are included in the final pipeline output

## Limitations

- Eduomics restricts simulations to a single chromosome (typically chr22), so the transcriptome reference used for quantification must be restricted to the same target regions or abundances will not match the ground truth
- Salmon quantification quality depends on the sequencing profile; simulated reads only reflect the coverage and replicate settings (e.g. coverage=30, reps=3, groups=2) given in the samplesheet

## Evidence

- [intro] Runs Salmon to quantify transcript expression.: "Runs Salmon to quantify transcript expression."
- [readme] RNA branch flow including quantification between simulation and DE analysis: [evidence span withheld — anchor status: not_found]
- [readme] Simulation ground truth that quantification must reproduce: [evidence span withheld — anchor status: not_found]
- [intro] DESeq2 consumes the gene-level counts: "Performs DESeq2 analysis to identify differentially expressed (DE) genes."
