---
name: snp-database-chromosome-splitting
description: Use when setting up the MAPIT-seq reference bundle (GRCh38 FASTA, GFF3,
  RepeatMasker rmsk) and editing-call inputs require known-SNP masks per chromosome,
  or when FLARE/SAILOR needs chromosome-level dbSNP bedfiles.
when_to_use_negative:
- Editing events are already called and the task is only quantifying A-to-G/C-to-U
  events per million bases or calling RBP targets via differential editing — no SNP
  re-annotation is needed then
- The SNP resources are already split per chromosome and use the same 'chr'-prefixed
  nomenclature as the reference genome, so re-splitting would be redundant
- Running a non-MAPIT-seq editing caller (e.g. standalone REDItools2 on DNA-Seq/RNA-Seq
  BAMs) that does not consume these per-chromosome SNP masks
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: MAPIT-seq pipeline
  role: 'Consumer of the chr-split SNP annotations: ''Mapit callediting'' (GATK RNA
    editing detection) and ''prepare'' (SAILOR file preparation) expect the known
    SNP datasets split per chromosome with ''chr''-prefixed names'
  repo: https://github.com/WangLabPKU/MAPIT-seq
- name: SAILOR/FLARE
  role: Downstream editing-site pipeline that uses known SNPs as per-chromosome/combined
    bed3 files to mask common variants during edit-site flagging
  repo: https://github.com/YeoLab/FLARE
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41592-025-02774-4
    title: Co-profiling of in situ RNA-protein interactions and transcriptome in single
      cells and tissues
  source_content_hash: sha256:96119f3c4b4ea9d1097d6e15ac6d43c1ea03654dcf38ae6bfa2327e581fa101e
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
  - HISAT2
  - MAPIT-seq pipeline
  - SAILOR/FLARE
  - conda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# snp-database-chromosome-splitting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preparation step of the MAPIT-seq pipeline in which known SNP resources (dbSNP, 1000 Genomes, EVS/EVA) are downloaded and split per chromosome with 'chr'-prefixed naming so that per-chromosome annotation files can be consumed by downstream editing-calling stages (GATK-based 'callediting' and SAILOR/FLARE-style site filtering).

## When to use

Use when setting up the MAPIT-seq reference bundle (GRCh38 FASTA, GFF3, RepeatMasker rmsk) and editing-call inputs require known-SNP masks per chromosome, or when FLARE/SAILOR needs chromosome-level dbSNP bedfiles.

## When NOT to use

- Editing events are already called and the task is only quantifying A-to-G/C-to-U events per million bases or calling RBP targets via differential editing — no SNP re-annotation is needed then
- The SNP resources are already split per chromosome and use the same 'chr'-prefixed nomenclature as the reference genome, so re-splitting would be redundant
- Running a non-MAPIT-seq editing caller (e.g. standalone REDItools2 on DNA-Seq/RNA-Seq BAMs) that does not consume these per-chromosome SNP masks

## Inputs

- dbSNP per-chromosome files (NCBI ftp.ncbi.nih.gov/snp/organisms/)
- 1000 Genomes Project biallelic SNV VCFs (GRCh38 release)
- EVS/EVA SNP archives (EVS_split_chr.zip from Zenodo record 17089899)
- GRCh38 reference FASTA and GFF3 annotation (for chromosome-nomenclature matching)
- RepeatMasker rmsk annotation (hg38) as part of the same reference bundle

## Outputs

- Per-chromosome SNP annotation files with 'chr'-prefixed names (split by chromosome)
- Combined known-SNP bed3 file (chrom/start/end, headers removed) for SAILOR/FLARE
- Chromosome-consistent reference bundle consumed by 'Mapit callediting' (GATK) and 'prepare' (SAILOR inputs)

## How to apply

Download each known SNP resource as chromosome-partitioned archives — e.g. the MAPIT-seq-provided EVS_split_chr.zip from Zenodo record 17089899, the 1000 Genomes biallelic-SNV release, and per-chromosome dbSNP files from NCBI's ftp.ncbi.nih.gov/snp/organisms/. Ensure every file uses 'chr'-prefixed chromosome names matching the GRCh38 reference FASTA and GFF3 annotation supplied to 'Mapit mapping'/'Mapit callediting'. For SAILOR/FLARE, download individual chromosome bedfiles, strip header lines, retain only chrom/start/end columns, and concatenate (e.g. 'tail -n+2 $b | cut -f1,2,3 | sort') into a combined bed3 file; alternatively keep them per-chromosome so the editing pipeline can look up the right mask. Verify nomenclature consistency between the SNP files and the reference before running GATK-based editing calls, since a mismatch causes silent lookup failures rather than explicit errors.

## Related tools

- **MAPIT-seq pipeline** (Consumer of the chr-split SNP annotations: 'Mapit callediting' (GATK RNA editing detection) and 'prepare' (SAILOR file preparation) expect the known SNP datasets split per chromosome with 'chr'-prefixed names) — https://github.com/WangLabPKU/MAPIT-seq
- **SAILOR/FLARE** (Downstream editing-site pipeline that uses known SNPs as per-chromosome/combined bed3 files to mask common variants during edit-site flagging) — https://github.com/YeoLab/FLARE

## Examples

```
wget -O EVS_split_chr.zip "https://zenodo.org/records/17089899/files/EVS_split_chr.zip?download=1"; for b in $(ls *.bed); do tail -n+2 $b | cut -f1,2,3 | sort >> mm10_dbsnp_combined.bed3; done
```

## Evaluation signals

- Every chromosome partition file exists and uses 'chr'-prefixed names identical to the GRCh38 FASTA/GFF3 chromosome identifiers
- For SAILOR/FLARE, the combined bed file contains only chrom/start/end columns and no header lines (tail -n+2 + cut -f1,2,3 applied to each source bed)
- Downstream 'callediting' output under the 6-Edit_calling directory is produced without chromosome-lookup errors, and editing_counts_per_sample.tsv has separate A-to-G and C-to-U columns for every sample in --sampleList
- The EVS archive downloaded from Zenodo (record 17089899) decompresses into per-chromosome files that cover the full genome set used by the reference annotation

## Limitations

- Chromosome nomenclature mismatch (e.g. 'chr1' vs '1') between SNP files and the reference silently breaks SNP masking in downstream GATK/SAILOR steps
- Known SNP datasets must be downloaded from external mirrors (NCBI dbSNP ftp, 1000 Genomes EBI, EVS at evs.gs.washington.edu, EVA at ftp.ebi.ac.uk); availability and version drift are outside pipeline control

## Evidence

- [readme] FLARE README: per-chromosome dbSNP bedfiles: [evidence span withheld — anchor status: not_found]
- [readme] FLARE README: combining chromosome bedfiles: [evidence span withheld — anchor status: not_found]
- [readme] FLARE README: nomenclature consistency: [evidence span withheld — anchor status: not_found]
