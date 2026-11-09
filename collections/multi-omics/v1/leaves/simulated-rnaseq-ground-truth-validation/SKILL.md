---
name: simulated-rnaseq-ground-truth-validation
description: Use when simulated RNA-seq FASTQ reads (Polyester-generated with a known
  DE ground truth) must be checked that quantification, DESeq2 testing, and GO enrichment
  reproduce the injected signal before release as a teaching dataset.
when_to_use_negative:
- The data are real (non-simulated) RNA-seq reads with unknown ground truth — validation
  against an injected DE signal is not meaningful
- Input is already a gene-level count matrix from a prior quantification step rather
  than simulated FASTQ reads requiring Salmon quantification
- The task is DNA variant-calling validation (checking injected pathogenic variants
  in SimuSCoP reads) rather than RNA differential expression validation
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0203
tools:
- name: Salmon
  role: Quantifies the Polyester-simulated FASTQ reads via quasi-mapping, producing
    transcript-level abundances for gene-level summarization
  repo: https://github.com/lescai-teaching/eduomics
- name: DESeq2
  role: Performs negative-binomial differential expression testing between simulated
    disease and normal groups; its statistics table feeds the validation significance
    check
  repo: https://github.com/lescai-teaching/eduomics
- name: clusterProfiler
  role: Runs GO over-representation analysis (BP/MF/CC) on the DE gene list; enrichment
    detectability and biological coherence are validated from its results
- name: Nextflow
  role: Orchestrates the QUANTIFY_DEANALYSIS_ENRICH_VALIDATE subworkflow including
    the validation step writing validation_result.txt
  repo: https://github.com/lescai-teaching/eduomics
- name: Polyester
  role: Generates the simulated FASTQ files matching the count matrices with known
    DE patterns that this validation evaluates
  repo: https://github.com/lescai-teaching/eduomics
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:2bad75302033c8ab94b9a7204ed2bfb4de7fef9ba3fcec1bcf034ca11077d1fd
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
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# simulated-rnaseq-ground-truth-validation

## Summary

Validates Polyester-simulated RNA-seq data by confirming that the Salmon→DESeq2→clusterProfiler RNA subworkflow recovers a statistically significant differential expression pattern with detectable, biologically plausible GO enrichment, writing the pass/fail outcome to validation_result.txt.

## When to use

Use when simulated RNA-seq FASTQ reads (Polyester-generated with a known DE ground truth) must be checked that quantification, DESeq2 testing, and GO enrichment reproduce the injected signal before release as a teaching dataset.

## When NOT to use

- The data are real (non-simulated) RNA-seq reads with unknown ground truth — validation against an injected DE signal is not meaningful
- Input is already a gene-level count matrix from a prior quantification step rather than simulated FASTQ reads requiring Salmon quantification
- The task is DNA variant-calling validation (checking injected pathogenic variants in SimuSCoP reads) rather than RNA differential expression validation

## Inputs

- Polyester-simulated paired-end FASTQ files for disease and normal sample groups
- Reference transcriptome/index for Salmon quasi-mapping
- Gene-level count matrix summarized from Salmon transcript abundances
- GO annotation (Biological Process, Molecular Function, Cellular Component) used for gene selection

## Outputs

- deseq2_results.tsv (full DESeq2 statistics table)
- deseq2_de_genes.txt (significant DE gene identifiers)
- DESeq2 diagnostic plots as PDFs (PCA, MA, volcano-style)
- enrichment_results.rds (clusterProfiler ORA enrichment object)
- dotplot and cnetplot visualizations as PNG files
- validation_result.txt (pass/fail outcome of RNA validation checks)

## How to apply

First quantify the Polyester-simulated paired-end FASTQ reads against the reference transcriptome with Salmon quasi-mapping, then import and summarize transcript abundances to gene-level counts for DESeq2. Run DESeq2's negative-binomial differential testing between the simulated disease and normal groups, writing deseq2_results.tsv and deseq2_de_genes.txt plus diagnostic PDFs (PCA, MA, volcano-style). Run clusterProfiler over-representation analysis (ORA) on the DE gene list across BP, MF and CC ontologies, saving enrichment_results.rds and dotplot/cnetplot PNGs. Finally apply the RNA validation thresholds: the DE pattern must be statistically significant, and the functional enrichment must be detectable and biologically coherent (i.e., consistent with the GO-annotated genes selected for simulation); write the pass/fail outcome to validation_result.txt. Only simulations passing validation are included in the final educational output.

## Related tools

- **Salmon** (Quantifies the Polyester-simulated FASTQ reads via quasi-mapping, producing transcript-level abundances for gene-level summarization) — https://github.com/lescai-teaching/eduomics
- **DESeq2** (Performs negative-binomial differential expression testing between simulated disease and normal groups; its statistics table feeds the validation significance check) — https://github.com/lescai-teaching/eduomics
- **clusterProfiler** (Runs GO over-representation analysis (BP/MF/CC) on the DE gene list; enrichment detectability and biological coherence are validated from its results)
- **Nextflow** (Orchestrates the QUANTIFY_DEANALYSIS_ENRICH_VALIDATE subworkflow including the validation step writing validation_result.txt) — https://github.com/lescai-teaching/eduomics
- **Polyester** (Generates the simulated FASTQ files matching the count matrices with known DE patterns that this validation evaluates) — https://github.com/lescai-teaching/eduomics

## Examples

```
nextflow run nf-core/eduomics -profile docker --input samplesheet.csv --genome GATK.GRCh38 --outdir <OUTDIR>
```

## Evaluation signals

- validation_result.txt records a pass outcome: the simulated data yield a statistically significant and coherent differential expression pattern
- deseq2_de_genes.txt contains the expected injected DE genes, matching the known ground-truth DE pattern from the simulated count matrices
- enrichment_results.rds shows detectable GO over-representation in the ontologies used for gene selection, with dotplot/cnetplot PNGs rendered successfully
- The DE directionality is consistent between the disease and normal simulated groups, i.e. enrichment is biologically plausible rather than random
- Simulations failing these checks are excluded, since only simulations passing validation are included in the final output

## Limitations

- The pipeline focuses on single chromosomes (typically chr22) to reduce computational requirements, so validation reflects chromosome-limited simulation scope
- Validation depends on the simulation parameters (coverage, reps, groups, simthreshold in the samplesheet); poorly parameterized simulations may fail the significance/coherence checks
- Validation is designed for paired-end human RNA-seq simulations against standard reference genomes (e.g. GATK.GRCh38) and does not generalize to other data modalities

## Evidence

- [intro] RNA validation checks recorded in validation_result.txt: [evidence span withheld — anchor status: not_found]
- [intro] Validation question for simulated RNA-seq data: [evidence span withheld — anchor status: not_found]
- [intro] Only passing simulations are output: "Only simulations passing validation are included in the final output"
- [methods] RNA subworkflow steps from Salmon to DESeq2: "Runs Salmon to quantify transcript expression."
- [readme] README pipeline logic for RNA mode: [evidence span withheld — anchor status: not_found]
