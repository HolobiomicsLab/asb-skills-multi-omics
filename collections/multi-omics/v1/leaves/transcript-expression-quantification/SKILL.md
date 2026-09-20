---
name: transcript-expression-quantification
description: Use when you have paired-end RNA-seq FASTQ files (e.g. Polyester-simulated
  reads with a known count-matrix ground truth) and need transcript-level abundance
  quantification prior to differential expression analysis.
when_to_use_negative:
- Input is DNA sequencing data — the DNA branch uses SimuSCoP for variant injection
  and variant-calling validation, not Salmon quantification
- You already have a gene-level count table — quantification is unnecessary; proceed
  directly to DESeq2
- The pipeline focuses on single chromosomes (typically chr22) with capture-based/targeted
  assumptions; whole-genome/whole-transome-scale studies need different scaling
edam_operation: http://edamontology.org/operation_3768
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Salmon
  role: Quantifies transcript expression from the Polyester-simulated FASTQ reads
    before DESeq2 analysis
  repo: https://github.com/lescai-teaching/eduomics
- name: Polyester
  role: Generates paired-end FASTQ files matching the count matrices that Salmon then
    quantifies
  repo: https://github.com/lescai-teaching/eduomics
- name: DESeq2
  role: Consumes Salmon quantification to identify differentially expressed (DE) genes
    for validation
  repo: https://github.com/lescai-teaching/eduomics
- name: FastQC
  role: Quality control of the FASTQ reads before/alongside quantification
  repo: https://github.com/lescai-teaching/eduomics
- name: MultiQC
  role: Aggregates QC reports (FastQC) across samples
  repo: https://github.com/lescai-teaching/eduomics
- name: Nextflow
  role: Workflow engine orchestrating the RNA quantification branch (version ≥25.04.4)
  repo: https://github.com/lescai-teaching/eduomics
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:ed2dad6a3bbc18e2bccbb90fd791c1a24b53eba9386df57f0239162f3646c2ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lescai-teaching/eduomics
  tools:
  - DESeq2
  - Docker
  - FastQC
  - MultiQC
  - Nextflow
  - Polyester
  - Salmon
  edam_operation: http://edamontology.org/operation_3768
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# transcript-expression-quantification

## Summary

Quantify transcript expression from simulated RNA-seq FASTQ reads with Salmon within the eduomics pipeline, producing abundance estimates that feed DESeq2 differential expression analysis and validation of a known DE ground truth.

## When to use

Use when you have paired-end RNA-seq FASTQ files (e.g. Polyester-simulated reads with a known count-matrix ground truth) and need transcript-level abundance quantification prior to differential expression analysis.

## When NOT to use

- Input is DNA sequencing data — the DNA branch uses SimuSCoP for variant injection and variant-calling validation, not Salmon quantification
- You already have a gene-level count table — quantification is unnecessary; proceed directly to DESeq2
- The pipeline focuses on single chromosomes (typically chr22) with capture-based/targeted assumptions; whole-genome/whole-transome-scale studies need different scaling

## Inputs

- Paired-end RNA-seq FASTQ files simulated by Polyester from count matrices with known DE patterns
- Transcriptome reference (subset to target chromosome/capture regions, e.g. GATK.GRCh38-based)
- Samplesheet (samplesheet.csv) with RNA simulation parameters: id, type=rna, chromosome, reps, groups, simthreshold

## Outputs

- Salmon transcript abundance quantification (used as DESeq2 input)
- deseq2_results.tsv and deseq2_de_genes.txt
- MA, dispersion, count, heatmap, and PCA plot PDFs
- enrichment_results.rds with dotplot and cnetplot PNGs
- validation_result.txt from the validation module

## How to apply

Within the eduomics RNA branch, count matrices with a known differential expression pattern are first generated, then Polyester produces paired-end FASTQ reads matching those count matrices. Run Salmon to quantify transcript expression from the simulated FASTQ files, with FastQC and MultiQC providing quality control of the reads. Salmon's quantification output is passed to DESeq2 to identify differentially expressed genes, and a downstream validation module then verifies that the simulated data yield a statistically significant and coherent DE pattern with biologically plausible GO enrichment. Judging correctness: Salmon quantification must be consistent with the injected ground truth such that DESeq2 recovers the expected DE genes and GO enrichment is detectable; only simulations passing validation are retained in the final rna_simulations output directory.

## Related tools

- **Salmon** (Quantifies transcript expression from the Polyester-simulated FASTQ reads before DESeq2 analysis) — https://github.com/lescai-teaching/eduomics
- **Polyester** (Generates paired-end FASTQ files matching the count matrices that Salmon then quantifies) — https://github.com/lescai-teaching/eduomics
- **DESeq2** (Consumes Salmon quantification to identify differentially expressed (DE) genes for validation) — https://github.com/lescai-teaching/eduomics
- **FastQC** (Quality control of the FASTQ reads before/alongside quantification) — https://github.com/lescai-teaching/eduomics
- **MultiQC** (Aggregates QC reports (FastQC) across samples) — https://github.com/lescai-teaching/eduomics
- **Nextflow** (Workflow engine orchestrating the RNA quantification branch (version ≥25.04.4)) — https://github.com/lescai-teaching/eduomics

## Examples

```
nextflow run eduomics -profile docker --input samplesheet.csv --genome GATK.GRCh38 --outdir <OUTDIR>
```

## Evaluation signals

- Salmon quantification of the simulated FASTQ is coherent with the injected count-matrix ground truth (expected DE genes are recoverable)
- DESeq2 on Salmon output produces a statistically significant and coherent differential expression pattern (deseq2_de_genes.txt non-empty and matching known truth)
- GO enrichment on DE genes is detectable and biologically plausible (dotplot/cnetplot generated from enrichment_results.rds)
- validation_result.txt written by the validation module confirms the simulation passed
- Only simulations passing validation appear in the final rna_simulations output directory; FastQC/MultiQC reports show acceptable read quality

## Limitations

- Simulations focus on single chromosomes (typically chr22) to reduce computational cost, so quantification realism is chromosome-specific
- The pipeline assumes paired-end reads and human standard reference genomes (e.g. GATK.GRCh38)
- Only simulations passing the validation module are included in the final output; failed simulations are excluded
- Salmon output is validated only in an educational context where ground truth is known, not as a general benchmark of quantification accuracy

## Evidence

- [intro] Salmon quantifies Polyester-simulated reads in the RNA branch: "Runs Salmon to quantify transcript expression."
- [intro] Simulated FASTQ matches count matrices: "Uses Polyester to generate FASTQ files matching the count matrices."
- [readme] Known ground truth in count matrices: [evidence span withheld — anchor status: not_found]
- [intro] Validation of DE and enrichment before final output: "Verifies that the simulated data produce a statistically significant and coherent differential expression pattern, with detectable and biologically plausible functional enrichment."
