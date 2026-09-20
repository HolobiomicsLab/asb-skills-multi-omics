---
name: gene-level-count-summarization-from-transcript-estimates
description: Use when Salmon quasi-mapping quant outputs (quant.sf files) from simulated
  FASTQ reads must be converted to gene-level counts before running DESeq2 between
  simulated disease and normal groups.
when_to_use_negative:
- The input is already a gene-level count matrix (e.g. the eduomics ground-truth count
  matrix itself), so transcript-to-gene summarization is unnecessary
- The quantification was performed with an aligner that directly outputs gene-level
  counts rather than Salmon transcript-level estimates
- The goal is transcript-level differential transcript expression rather than gene-level
  DE testing in DESeq2
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Salmon
  role: Produces the transcript-level abundance estimates that are summarized to gene-level
    counts
- name: DESeq2
  role: Consumes the gene-level count matrix for negative-binomial differential expression
    testing
- name: Nextflow
  role: Orchestrates the QUANTIFY_DEANALYSIS_ENRICH_VALIDATE subworkflow containing
    this step
  repo: https://github.com/lescai-teaching/eduomics
- name: Polyester
  role: Generated the simulated FASTQ reads whose quantified abundances are summarized
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:a04bb59299463f9234f1190d51c8e2aa0888c801c8d0aa2d83cd426514453f70
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
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Gene-level count summarization from transcript estimates

## Summary

Summarizes Salmon transcript-level abundance estimates from Polyester-simulated RNA-seq reads into gene-level counts suitable for DESeq2 negative-binomial differential expression testing within the eduomics RNA subworkflow.

## When to use

Use when Salmon quasi-mapping quant outputs (quant.sf files) from simulated FASTQ reads must be converted to gene-level counts before running DESeq2 between simulated disease and normal groups.

## When NOT to use

- The input is already a gene-level count matrix (e.g. the eduomics ground-truth count matrix itself), so transcript-to-gene summarization is unnecessary
- The quantification was performed with an aligner that directly outputs gene-level counts rather than Salmon transcript-level estimates
- The goal is transcript-level differential transcript expression rather than gene-level DE testing in DESeq2

## Inputs

- Salmon transcript-level quantification outputs (quant.sf) from Polyester-simulated paired-end FASTQ reads
- Transcript-to-gene mapping for the (subset) reference transcriptome
- Sample metadata distinguishing simulated disease vs normal groups

## Outputs

- Gene-level count matrix formatted for DESeq2 input
- Downstream DESeq2 statistics table (deseq2_results.tsv)
- Significant DE gene identifier list (deseq2_de_genes.txt)

## How to apply

After Salmon quantifies the Polyester-simulated FASTQ reads against the reference transcriptome using quasi-mapping, import the transcript-level abundance files and summarize them to gene-level counts so that DESeq2's negative-binomial differential expression testing can be run between the simulated disease and normal sample groups. The summarization step requires mapping transcript identifiers to gene identifiers, and the resulting gene-level count matrix becomes the direct input to DESeq2, whose full statistics are written to deseq2_results.tsv and significant DE gene identifiers to deseq2_de_genes.txt. This bridging step is essential because Salmon operates at transcript resolution while DESeq2 requires gene-level counts; in the eduomics context the ground-truth count matrices with known differential expression patterns are what the simulated reads were generated from, so the summarized counts should recover that known signal.

## Related tools

- **Salmon** (Produces the transcript-level abundance estimates that are summarized to gene-level counts)
- **DESeq2** (Consumes the gene-level count matrix for negative-binomial differential expression testing)
- **Nextflow** (Orchestrates the QUANTIFY_DEANALYSIS_ENRICH_VALIDATE subworkflow containing this step) — https://github.com/lescai-teaching/eduomics
- **Polyester** (Generated the simulated FASTQ reads whose quantified abundances are summarized)

## Evaluation signals

- Summarized gene-level counts must be consistent with the known differential expression patterns injected into the ground-truth count matrices
- DESeq2 run on the summarized counts must produce a statistically significant and coherent differential expression pattern as checked by the pipeline's RNA validation recorded in validation_result.txt
- The significant DE gene list (deseq2_de_genes.txt) must correspond to the genes known to be differentially expressed in the simulation
- Subsequent GO enrichment via clusterProfiler must be detectable and biologically plausible for the summarized-count-derived DE gene list

## Limitations

- The pipeline focuses on single chromosomes (typically chr22) to reduce computational requirements, so the reference transcriptome being summarized is chromosome-subsetted rather than whole-genome
- Only simulations passing the pipeline's RNA validation thresholds are included in final outputs, so failures in summarization-dependent validation exclude the dataset

## Evidence

- [other] task workflow quote: [evidence span withheld — anchor status: not_found]
- [intro] article intro salmon: "Runs Salmon to quantify transcript expression."
- [intro] article intro deseq2: "Performs DESeq2 analysis to identify differentially expressed (DE) genes."
- [readme] readme rna logic: [evidence span withheld — anchor status: not_found]
