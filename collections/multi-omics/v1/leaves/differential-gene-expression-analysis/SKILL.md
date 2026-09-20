---
name: differential-gene-expression-analysis
description: Use when Salmon-quantified transcript counts from simulated RNA-seq reads
  (Polyester-generated FASTQ with a known injected DE pattern) must be analyzed with
  DESeq2 and GO enrichment validated for teaching workflows.
when_to_use_negative:
- The input is real (non-simulated) RNA-seq data where the ground-truth DE pattern
  is unknown — this skill is designed for controlled simulation datasets used for
  teaching
- The task is the DNA variant-simulation branch (SimuSCoP reads, ClinVar pathogenic
  variants, variant calling validation) rather than transcriptomic differential expression
- The analysis requires whole-genome, multi-chromosome differential expression — eduomics
  assumes chromosome-specific analysis (typically chr22) to reduce computational requirements
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Salmon
  role: Quantifies transcript expression from the Polyester-simulated FASTQ files,
    producing the counts that feed DESeq2
- name: DESeq2
  role: Performs the differential expression analysis to identify differentially expressed
    (DE) genes from the quantified counts
- name: Polyester
  role: Generates FASTQ files matching the count matrices carrying the known DE pattern
    being validated
- name: FastQC
  role: Quality control of the simulated FASTQ reads before quantification
- name: MultiQC
  role: Aggregates QC results for the RNA simulation run
provenance:
  source_task_ids:
  - task_002
  - task_007
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:55dd251cb04d137bcef786dd9bd90e1db5981a4325e79956efd75605d3022fb7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DESeq2
  - Docker
  - FastQC
  - MultiQC
  - Polyester
  - Salmon
  - clusterProfiler
  - igraph
  - org.Hs.eg.db
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# differential-gene-expression-analysis

## Summary

Identifies differentially expressed genes from Salmon-quantified transcript expression of simulated RNA-seq reads using DESeq2, followed by Gene Ontology enrichment with clusterProfiler, within the eduomics RNA simulation branch whose ground-truth DE pattern is known by construction.

## When to use

Use when Salmon-quantified transcript counts from simulated RNA-seq reads (Polyester-generated FASTQ with a known injected DE pattern) must be analyzed with DESeq2 and GO enrichment validated for teaching workflows.

## When NOT to use

- The input is real (non-simulated) RNA-seq data where the ground-truth DE pattern is unknown — this skill is designed for controlled simulation datasets used for teaching
- The task is the DNA variant-simulation branch (SimuSCoP reads, ClinVar pathogenic variants, variant calling validation) rather than transcriptomic differential expression
- The analysis requires whole-genome, multi-chromosome differential expression — eduomics assumes chromosome-specific analysis (typically chr22) to reduce computational requirements

## Inputs

- Polyester-simulated paired-end RNA-seq FASTQ files matching a count matrix with a known DE pattern
- Salmon transcript quantification outputs
- GATK.GRCh38 reference genome and transcriptome annotation subset to target regions
- GO gene-to-term annotations (org.Hs.eg.db) with simthreshold = 0.3 Jaccard cutoff
- samplesheet.csv row of type rna (e.g. rna_simulation_1,rna,chr22,30,,3,2,0.3)

## Outputs

- deseq2_results.tsv (per-gene DESeq2 statistics)
- deseq2_de_genes.txt (differentially expressed gene list)
- MA, dispersion, count, heatmap, and PCA plot PDFs
- enrichment_results.rds with dotplot and cnetplot PNGs
- validation_result.txt from the validation module
- rna_simulations output directory retaining only validated simulations

## How to apply

Run Salmon on the Polyester-simulated FASTQ files to quantify transcript expression, with FastQC and MultiQC guarding read quality. Feed the count matrix carrying the known differential expression pattern into DESeq2 to identify differentially expressed genes, saving deseq2_results.tsv and deseq2_de_genes.txt plus MA, dispersion, count, heatmap, and PCA plot PDFs. Conduct GO enrichment analysis on the DE genes, saving enrichment_results.rds with dotplot and cnetplot PNGs. Run the validation module to confirm the analysis recovers a statistically significant and coherent DE pattern with detectable, biologically plausible functional enrichment, writing validation_result.txt; only simulations passing validation are retained in the final rna_simulations output directory. Gene selection upstream is constrained by a GO-similarity network built with a Jaccard threshold (simthreshold, default 0.3) restricted to protein-coding genes on the target chromosome (typically chr22).

## Related tools

- **Salmon** (Quantifies transcript expression from the Polyester-simulated FASTQ files, producing the counts that feed DESeq2)
- **DESeq2** (Performs the differential expression analysis to identify differentially expressed (DE) genes from the quantified counts)
- **Polyester** (Generates FASTQ files matching the count matrices carrying the known DE pattern being validated)
- **FastQC** (Quality control of the simulated FASTQ reads before quantification)
- **MultiQC** (Aggregates QC results for the RNA simulation run)

## Examples

```
nextflow run eduomics -profile docker --input samplesheet.csv --genome GATK.GRCh38 --outdir <OUTDIR>
```

## Evaluation signals

- deseq2_results.tsv and deseq2_de_genes.txt exist and the DE gene list matches the known injected DE pattern
- MA, dispersion, count, heatmap, and PCA plot PDFs are all produced alongside the DESeq2 tables
- enrichment_results.rds exists with dotplot and cnetplot PNGs showing detectable, biologically plausible GO enrichment
- validation_result.txt reports a statistically significant and coherent differential expression pattern
- the simulation is retained in the final rna_simulations directory only if it passes the validation module

## Limitations

- Gene selection is restricted to protein-coding genes on a single chromosome (typically chr22) via a GO Jaccard similarity network pruned at simthreshold = 0.3, so the DE universe is narrow by design
- Only simulations passing the validation module are included in the final output; failures are dropped from rna_simulations
- The analysis validates simulations against known ground truth — it does not establish biological discovery on uncontrolled datasets

## Evidence

- [intro] Performs DESeq2 analysis to identify differentially expressed (DE) genes.: "Performs DESeq2 analysis to identify differentially expressed (DE) genes."
- [intro] Runs Salmon to quantify transcript expression.: "Runs Salmon to quantify transcript expression."
- [intro] Conducts GO enrichment analysis on DE genes.: "Conducts GO enrichment analysis on DE genes."
- [intro] Verifies that the simulated data produce a statistically significant and coherent differential expression pattern, with detectable and biologically plausible functional enrichment.: "Verifies that the simulated data produce a statistically significant and coherent differential expression pattern, with detectable and biologically plausible functional enrichment."
- [readme] Creates realistic count matrices with known differential expression patterns and generates corresponding RNA-seq reads: [evidence span withheld — anchor status: not_found]
