---
name: rna-seq-read-simulation-from-count-matrices
description: Use when you need simulated RNA-seq FASTQ data whose underlying expression
  counts contain a known, controlled differential expression pattern for teaching
  or validating DESeq2/Salmon workflows.
when_to_use_negative:
- You need real, previously uncharacterized RNA-seq data — eduomics simulations are
  designed for education where ground truth must be known, not for discovery studies.
- You want DNA variant-calling simulations — those use the SimuSCoP DNA branch with
  injected ClinVar variants, not Polyester count-matrix simulation.
- Your organism is non-human or you need whole-transome (not chromosome-subset) scope
  — the pipeline assumes human standard reference genomes and focuses on single chromosomes
  to keep compute tractable.
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_3519
- http://edamontology.org/topic_0203
tools:
- name: Polyester
  role: Generates paired-end FASTQ files matching the count matrices with the known
    differential expression pattern
  repo: https://github.com/lescai-teaching/eduomics
- name: Salmon
  role: Quantifies transcript expression from the simulated FASTQ files to confirm
    the encoded counts are recoverable
- name: DESeq2
  role: Identifies differentially expressed genes from the simulated data to validate
    the injected DE pattern
- name: Nextflow
  role: Workflow engine orchestrating the RNA simulation branch (version ≥25.04)
  repo: https://github.com/lescai-teaching/eduomics
- name: Docker
  role: Container system used to run the pipeline reproducibly (-profile docker)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:8349f4b692c325499e5c4937506f3a3204c8d1b1e8173a948c810fd842c49236
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
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_3519
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# rna-seq-read-simulation-from-count-matrices

## Summary

Generate realistic paired-end RNA-seq FASTQ reads from count matrices carrying a known differential expression pattern, using Polyester within the eduomics Nextflow pipeline, so that downstream DE analysis can be taught and validated against known ground truth.

## When to use

Use when you need simulated RNA-seq FASTQ data whose underlying expression counts contain a known, controlled differential expression pattern for teaching or validating DESeq2/Salmon workflows.

## When NOT to use

- You need real, previously uncharacterized RNA-seq data — eduomics simulations are designed for education where ground truth must be known, not for discovery studies.
- You want DNA variant-calling simulations — those use the SimuSCoP DNA branch with injected ClinVar variants, not Polyester count-matrix simulation.
- Your organism is non-human or you need whole-transome (not chromosome-subset) scope — the pipeline assumes human standard reference genomes and focuses on single chromosomes to keep compute tractable.

## Inputs

- Samplesheet rows with type=rna specifying id, chromosome (e.g. chr22), coverage (e.g. 30), reps (e.g. 3), groups (e.g. 2), and simthreshold (e.g. 0.3)
- Subset transcriptome reference and gene annotation for the target chromosome/capture regions
- GO-annotated gene sets used to select genes suitable for differential expression

## Outputs

- Paired-end RNA-seq FASTQ files matching the count matrices with the known DE pattern
- Count matrices carrying the known differential expression pattern
- Salmon quantification of the simulated reads
- deseq2_results.tsv, deseq2_de_genes.txt, and MA/dispersion/count/heatmap/PCA plot PDFs
- validation_result.txt confirming statistically significant and coherent DE with biologically plausible enrichment

## How to apply

Within the nf-core/eduomics RNA branch, first subset the transcriptome reference and annotation to the target chromosome (typically chr22) and identify genes suitable for differential expression based on Gene Ontology annotation. Create realistic count matrices with a known differential expression pattern, choosing the number of replicates and groups via the samplesheet (e.g. reps=3, groups=2) and a simulation threshold (simthreshold, e.g. 0.3). Use Polyester to generate paired-end FASTQ files that match those count matrices, so the simulated reads encode the injected DE signal. After simulation, quantify with Salmon, run DESeq2, and pass results to the validation module — only simulations whose validation_result.txt confirms a statistically significant, coherent DE pattern with biologically plausible GO enrichment are retained in the final rna_simulations output directory.

## Related tools

- **Polyester** (Generates paired-end FASTQ files matching the count matrices with the known differential expression pattern) — https://github.com/lescai-teaching/eduomics
- **Salmon** (Quantifies transcript expression from the simulated FASTQ files to confirm the encoded counts are recoverable)
- **DESeq2** (Identifies differentially expressed genes from the simulated data to validate the injected DE pattern)
- **Nextflow** (Workflow engine orchestrating the RNA simulation branch (version ≥25.04)) — https://github.com/lescai-teaching/eduomics
- **Docker** (Container system used to run the pipeline reproducibly (-profile docker))

## Examples

```
nextflow run nf-core/eduomics -profile docker --input samplesheet.csv --genome GATK.GRCh38 --outdir <OUTDIR>
```

## Evaluation signals

- validation_result.txt reports a statistically significant and coherent differential expression pattern
- DESeq2 recovers DE genes consistent with the known injected pattern (deseq2_de_genes.txt non-empty and matching ground truth)
- GO enrichment on the DE genes yields detectable, biologically plausible enrichment (dotplot and cnetplot PNGs produced)
- Simulated FASTQ files are paired-end and consistent with the count matrices after Salmon quantification
- Only simulations passing validation are retained in the final rna_simulations output directory

## Limitations

- The pipeline focuses on single chromosomes (typically chr22) to reduce computational requirements, so simulations are chromosome-subset rather than transcriptome-wide.
- Simulations are designed for human data using standard reference genomes (GATK.GRCh38 recommended) and paired-end sequencing.
- Only simulations that pass the validation module are included in the final output; failing simulations are discarded.

## Evidence

- [intro] Creates realistic count matrices with known differential expression patterns.: "Creates realistic count matrices with known differential expression patterns."
- [intro] Polyester FASTQ generation: "Uses Polyester to generate FASTQ files matching the count matrices."
- [intro] gene selection via GO: "Identifies genes suitable for differential expression based on Gene Ontology (GO) annotation."
- [intro] validation of simulated data: "Verifies that the simulated data produce a statistically significant and coherent differential expression pattern, with detectable and biologically plausible functional enrichment."
- [intro] filter: only passing simulations retained: "- Only simulations passing validation are included in the final output"
- [readme] samplesheet RNA row parameters: [evidence span withheld — anchor status: not_found]
- [readme] chromosome-subset assumption: [evidence span withheld — anchor status: not_found]
