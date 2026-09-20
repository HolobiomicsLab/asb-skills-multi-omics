---
name: nf-core-pipeline-execution-and-validation
description: Use when a task requires launching an nf-core/eduomics Nextflow workflow
  with a samplesheet and reference config, or validating that simulated -omics outputs
  pass the pipeline's internal validation module before reuse.
when_to_use_negative:
- The goal is analyzing real, previously generated -omics data rather than simulating
  datasets with known ground truth — eduomics is designed for educational simulation,
  not reanalysis
- The task is variant calling or DE analysis on student-supplied data (use the appropriate
  nf-core pipelines instead of this simulation framework)
- Nextflow and a container engine cannot be installed or the target genome is non-human,
  since the pipeline assumes standard human reference genomes
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0203
tools:
- name: Nextflow
  role: Workflow engine to run the nf-core/eduomics pipeline across containerized
    processes (version ≥25.04)
  repo: https://github.com/nf-core/eduomics
- name: Docker
  role: Container profile making installation trivial and results reproducible (one
    container per process)
- name: Polyester
  role: Generates RNA-seq FASTQ files matching the simulated count matrices
  repo: https://github.com/lescai-teaching/eduomics
- name: Salmon
  role: Quantifies transcript expression from the simulated FASTQ files
  repo: https://github.com/lescai-teaching/eduomics
- name: DESeq2
  role: Identifies differentially expressed genes from Salmon quantification
  repo: https://github.com/lescai-teaching/eduomics
- name: FastQC
  role: Quality control of the simulated FASTQ reads
  repo: https://github.com/lescai-teaching/eduomics
- name: MultiQC
  role: Aggregates FastQC QC reports across samples
  repo: https://github.com/lescai-teaching/eduomics
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:42cc19911426dd17babd16ba1604ab9d3a2298ab4a71a2862bf0cc38a896302c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nf-core/eduomics
  tools:
  - DESeq2
  - Docker
  - FastQC
  - MultiQC
  - Nextflow
  - Polyester
  - Salmon
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# nf-core-pipeline-execution-and-validation

## Summary

Execute an nf-core Nextflow pipeline (eduomics) under a containerized profile and verify that its simulation modules produce statistically valid, ground-truth-backed outputs. This skill covers running the workflow and interpreting its built-in validation gates.

## When to use

Use when a task requires launching an nf-core/eduomics Nextflow workflow with a samplesheet and reference config, or validating that simulated -omics outputs pass the pipeline's internal validation module before reuse.

## When NOT to use

- The goal is analyzing real, previously generated -omics data rather than simulating datasets with known ground truth — eduomics is designed for educational simulation, not reanalysis
- The task is variant calling or DE analysis on student-supplied data (use the appropriate nf-core pipelines instead of this simulation framework)
- Nextflow and a container engine cannot be installed or the target genome is non-human, since the pipeline assumes standard human reference genomes

## Inputs

- samplesheet.csv with columns id,type,chromosome,coverage,capture,reps,groups,simthreshold
- Nextflow profile selection (docker/singularity/conda/institute)
- Reference genome configuration (recommended GATK.GRCh38)
- For DNA rows: capture regions BED; for RNA rows: reps and groups parameters

## Outputs

- Organized simulation output directories (dna_simulations / rna_simulations)
- RNA: deseq2_results.tsv, deseq2_de_genes.txt, MA/dispersion/count/heatmap/PCA plot PDFs, enrichment_results.rds, dotplot and cnetplot PNGs, validation_result.txt
- DNA: simulated FASTQ with injected variants, variant VCFs (gnomAD, Mills, 1000 Genomes, dbSNP), validation results, AI educational scenario
- Assembled reference bundles for downstream analysis

## How to apply

Launch the pipeline with `nextflow run nf-core/eduomics -profile docker --input samplesheet.csv --genome GATK.GRCh38 --outdir <OUTDIR>`, using Nextflow ≥25.04 (DSL2) and a container engine (Docker, Singularity, or Conda). Structure the samplesheet with per-simulation parameters (id, type=dna|rna, chromosome such as chr22, coverage, capture BED for DNA, reps/groups/simthreshold for RNA). For RNA simulations, the workflow subsets the transcriptome reference, builds count matrices with a known DE pattern, simulates reads with Polyester, quantifies with Salmon, runs DESeq2, and performs GO enrichment, writing deseq2_results.tsv, deseq2_de_genes.txt, plot PDFs, and enrichment_results.rds with dotplot/cnetplot PNGs. Run the validation module, which checks for statistically significant and coherent differential expression with biologically plausible functional enrichment and writes validation_result.txt. Only simulations passing validation are retained in the final rna_simulations output directory, so treat validation_result.txt as the acceptance gate for downstream reuse.

## Related tools

- **Nextflow** (Workflow engine to run the nf-core/eduomics pipeline across containerized processes (version ≥25.04)) — https://github.com/nf-core/eduomics
- **Docker** (Container profile making installation trivial and results reproducible (one container per process))
- **Polyester** (Generates RNA-seq FASTQ files matching the simulated count matrices) — https://github.com/lescai-teaching/eduomics
- **Salmon** (Quantifies transcript expression from the simulated FASTQ files) — https://github.com/lescai-teaching/eduomics
- **DESeq2** (Identifies differentially expressed genes from Salmon quantification) — https://github.com/lescai-teaching/eduomics
- **FastQC** (Quality control of the simulated FASTQ reads) — https://github.com/lescai-teaching/eduomics
- **MultiQC** (Aggregates FastQC QC reports across samples) — https://github.com/lescai-teaching/eduomics

## Examples

```
nextflow run nf-core/eduomics -profile docker --input samplesheet.csv --genome GATK.GRCh38 --outdir results
```

## Evaluation signals

- validation_result.txt confirms the simulated data produce a statistically significant and coherent differential expression pattern
- GO enrichment on DE genes yields detectable, biologically plausible functional enrichment (dotplot/cnetplot PNGs non-empty)
- deseq2_de_genes.txt recovers the ground-truth DE genes injected into the count matrices
- Only simulations passing validation appear in the final rna_simulations output directory
- MultiQC report shows simulated FASTQs pass FastQC QC checks

## Limitations

- Chromosome-specific analysis (typically chr22) is assumed to reduce computational load, so results are not genome-wide
- DNA simulations assume exome or targeted capture-based sequencing; whole-genome simulation is not the default scope
- Custom config files via -c cannot set parameters; parameters must be provided via CLI or -params-file
- Pipeline is designed for human data with standard reference genomes and paired-end reads

## Evidence

- [readme] Launch with docker profile and GATK.GRCh38: "nextflow run eduomics \
   -profile <docker/singularity/.../institute> \
   --input samplesheet.csv \
   --genome GATK.GRCh38"
- [intro] Validation gate definition: "Verifies that the simulated data produce a statistically significant and coherent differential expression pattern, with detectable and biologically plausible functional enrichment."
- [intro] Only validated simulations retained: "Only simulations passing validation are included in the final output"
- [readme] Educational ground-truth philosophy: [evidence span withheld — anchor status: not_found]
- [readme] Sample sheet schema: "id,type,chromosome,coverage,capture,reps,groups,simthreshold"
