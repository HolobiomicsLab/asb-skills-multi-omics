---
name: mapit-seq-pipeline-execution
description: Use when paired-end FASTQ data from MAPIT-seq experiments needs alignment,
  abundant-RNA filtering, RNA editing event calling (A-to-G and C-to-U), or differential
  editing-based RBP target calling.
when_to_use_negative:
- The goal is whole-transcriptome variant calling on matched DNA-RNA pairs rather
  than antibody-directed editing enrichment around RBP targets
- Data lack the hADAR2dd/rAPOBEC1 editing enzymes introduced by the MAPIT-seq protocol
  (e.g., standard RNA-seq without targeted deamination)
- Only alignment-free expression quantification is needed — the full editing-calling
  machinery (GATK, Poisson filters, coverage thresholds) is unnecessary overhead
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0080
tools:
- name: MAPIT-seq pipeline
  role: 'Main workflow: mapping (two-round unique alignment), finetuning, callediting
    (GATK), calltargets (differential editing target calling)'
  repo: https://github.com/WangLabPKU/MAPIT-seq
- name: BWA-MEM
  role: Indexes abundant RNA sequences (rRNA, tRNA, mt-rRNA, mt-tRNA) used for the
    abundant-RNA filtering step
  repo: https://github.com/WangLabPKU/MAPIT-seq
- name: GATK
  role: Engine invoked by the callediting step to detect A-to-G and C-to-U RNA editing
    events ('-e Both')
  repo: https://github.com/WangLabPKU/MAPIT-seq
- name: SAILOR / FLARE
  role: SAILOR-derived editing sites feed FLARE, a Snakemake pipeline that statistically
    flags regions of enriched RNA editing (prepare step files SAILOR input)
  repo: https://github.com/YeoLab/FLARE
- name: REDItools2
  role: Alternative RNA editing event caller from RNA-Seq BAM alignments, provided
    in the article's methods
  repo: https://github.com/BioinfoUNIBA/REDItools2
- name: conda
  role: Installation channel for all pipeline software (conda install snakemake -c
    bioconda -c conda-forge; conda install mpi4py)
merged_aliases:
- mapit-seq-calltargets-execution
merged_alias_records:
- alias: mapit-seq-calltargets-execution
  slug: mapit-seq-calltargets-execution
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:e21834d003a46d73a796f460f0e13df184038a90c5b280575c9f416d75a93043
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/WangLabPKU/MAPIT-seq
  repo_url: https://github.com/WangLabPKU/MAPIT-seq
  tools:
  - BWA-MEM
  - GATK
  - HISAT2
  - MAPIT-seq pipeline
  - REDItools2
  - SAILOR / FLARE
  - conda
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# mapit-seq-pipeline-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Execute the MAPIT-seq computational pipeline (WangLabPKU/MAPIT-seq, v1.7) to detect hADAR2dd A-to-G and rAPOBEC1 C-to-U RNA editing events from paired-end FASTQ data and call RBP-bound target transcripts, simultaneously profiling RBP-RNA interactions and the transcriptome.

## When to use

Use when paired-end FASTQ data from MAPIT-seq experiments needs alignment, abundant-RNA filtering, RNA editing event calling (A-to-G and C-to-U), or differential editing-based RBP target calling.

## When NOT to use

- The goal is whole-transcriptome variant calling on matched DNA-RNA pairs rather than antibody-directed editing enrichment around RBP targets
- Data lack the hADAR2dd/rAPOBEC1 editing enzymes introduced by the MAPIT-seq protocol (e.g., standard RNA-seq without targeted deamination)
- Only alignment-free expression quantification is needed — the full editing-calling machinery (GATK, Poisson filters, coverage thresholds) is unnecessary overhead

## Inputs

- Paired-end FASTQ files (R1.fq.gz, R2.fq.gz) from MAPIT-seq libraries
- GRCh38 reference genome FASTA and GFF3 annotation
- RepeatMasker (rmsk) annotation
- Known SNP datasets split per chromosome (dbSNP, 1000 Genomes, EVS/EVA VCFs)
- Abundant RNA sequences (rRNA, tRNA, mt-rRNA, mt-tRNA) for BWA-MEM index

## Outputs

- editing_counts_per_sample.tsv with separate A-to-G and C-to-U editing count columns
- Per-sample RNA editing event tables (*_Edit_*_DATA.tsv under 6-Edit_calling)
- RBP target transcript lists with adjusted P values (e.g., prc2_target_adjusted_pvalues.csv)
- High-confidence RBP-binding edit clusters and de novo RBP-binding motifs

## How to apply

Install all pipeline software via conda and clone the WangLabPKU/MAPIT-seq repository. Prepare the GRCh38 reference FASTA, GFF3 annotation, RepeatMasker (rmsk) annotation, and known SNP sets (dbSNP, 1000 Genomes, EVS/EVA) split per chromosome with 'chr'-prefixed names. Run 'Mapit mapping' to filter abundant RNAs (rRNA, tRNA, mt-rRNA, mt-tRNA indexed by BWA-MEM) and perform two-round unique genome alignment, then 'Mapit finetuning' to fine-tune alignments. Run 'Mapit callediting' with GATK using '-e Both' to detect both ADAR (A-to-G) and APOBEC (C-to-U) editing, then 'Mapit calltargets' to identify RBP targets by differential editing in 50-nt bins (binSize 50) requiring minimum editing-site coverage of 10 and Poisson filtering (default maximum p-value 0.1). For downstream annotation, prepare files for SAILOR and use FLARE (a Snakemake pipeline) or REDItools2 to flag areas of RNA-editing enrichment; target calls apply editing fold enrichment > 2, MAPIT score > 0.5, and adjusted P < 0.05.

## Related tools

- **MAPIT-seq pipeline** (Main workflow: mapping (two-round unique alignment), finetuning, callediting (GATK), calltargets (differential editing target calling)) — https://github.com/WangLabPKU/MAPIT-seq
- **BWA-MEM** (Indexes abundant RNA sequences (rRNA, tRNA, mt-rRNA, mt-tRNA) used for the abundant-RNA filtering step) — https://github.com/WangLabPKU/MAPIT-seq
- **GATK** (Engine invoked by the callediting step to detect A-to-G and C-to-U RNA editing events ('-e Both')) — https://github.com/WangLabPKU/MAPIT-seq
- **SAILOR / FLARE** (SAILOR-derived editing sites feed FLARE, a Snakemake pipeline that statistically flags regions of enriched RNA editing (prepare step files SAILOR input)) — https://github.com/YeoLab/FLARE
- **REDItools2** (Alternative RNA editing event caller from RNA-Seq BAM alignments, provided in the article's methods) — https://github.com/BioinfoUNIBA/REDItools2
- **conda** (Installation channel for all pipeline software (conda install snakemake -c bioconda -c conda-forge; conda install mpi4py))

## Examples

```
Mapit mapping -v GRCh38 --fq R1.fq.gz --fq2 R2.fq.gz --rna-strandness FR -n <sample> -r <rep> -o Mapit_result -t <threads>; Mapit finetuning -v GRCh38 -n <sample> -r <rep> -o Mapit_result; Mapit callediting -v GRCh38 --sampleList <samples> --prefix RBP -e Both -t <threads>
```

## Evaluation signals

- editing_counts_per_sample.tsv exists, contains one row per sample in --sampleList, and has separate A-to-G and C-to-U count columns consistent with records under the 6-Edit_calling output directory
- Reproduced XIST adjusted P values match reported values within tolerance: 0.0463 (EED), 1.72e-03 (EZH2), 9.94e-06 (SUZ12)
- PRC2 component target sets (EED, EZH2, SUZ12) remain near-empty at thresholds fold enrichment > 2, MAPIT score > 0.5, adjusted P < 0.05
- Chromosome nomenclature in reference and SNP files is consistent ('chr'-prefixed) and per-chromosome SNP splits load without header errors
- Editing quantification reports events per million bases within 800-nt windows flanking YTHDF2 PAR-CLIP peaks

## Limitations

- FLARE requires Linux CentOS 7 or newer, snakemake, and Singularity for several pipeline steps
- REDItools2.0 was tested with Python 2.7.12 and has no guarantee of compatibility with other Python versions; installing it at user level can uninstall/downgrade existing pysam packages
- Mechanism underlying MAPIT-seq's higher SNR versus INSCRIBE remains unexplained (contributing factors, e.g. secondary antibody use, are speculative); scMAPIT-seq application to tissue samples is deferred to future work

## Evidence

- [methods] conda install route: [evidence span withheld — anchor status: not_found]
- [readme] FLARE description: [evidence span withheld — anchor status: not_found]
