---
name: gwas-snp-pvalue-reanalysis
description: Use when re-analyzing per-SNP p-values from a GWAS (SNP_p_values.txt
  style input) to cross-validate dependent p-value combination methods against GWAR
  MIN2/MAX reference results.
when_to_use_negative:
- Input p-values are independent (no correlation matrix available) — metacp's dependent-test
  comparison to GWAR MIN2/MAX presumes dependent tests with a supplied correlation
  matrix
- Input statistics are z-scores rather than p-values, since this workflow invokes
  metacp with the 'p-values' argument
- A true meta-analysis across multiple rows for the same SNP/gene is needed rather
  than within-row p-value combination (here the Yes/No meta-analysis flag is set to
  reproduce the single-table comparison)
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3517
tools:
- name: metacp
  role: Python command-line tool run via metacp.py to produce CCT, MinP, MCM and CMC
    combined p-values from the 67-SNP p-value file and correlation matrix
  repo: https://github.com/pbagos/metacp
- name: Python
  role: Runtime for metacp.py and for the -log10 transformation and R2 correlation
    computation against GWAR MIN2/MAX results
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:57222f4d4fbe3c45f3a99ee1cf5d7748c98a64eeb3cc4fae49bd17eb5c66f1b4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/pbagos/metacp
  repo_url: https://github.com/pbagos/metacp
  tools:
  - Python
  - metacp
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# gwas-snp-pvalue-reanalysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the metacp-vs-GWAR agreement analysis on the 67 SNPs from the type-2 diabetes GWAS by running metacp's dependent-test methods (CCT, MinP, MCM, CMC) and comparing them to GWAR's MIN2 and MAX combined p-values via R2 on -log10 p-values.

## When to use

Use when re-analyzing per-SNP p-values from a GWAS (SNP_p_values.txt style input) to cross-validate dependent p-value combination methods against GWAR MIN2/MAX reference results.

## When NOT to use

- Input p-values are independent (no correlation matrix available) — metacp's dependent-test comparison to GWAR MIN2/MAX presumes dependent tests with a supplied correlation matrix
- Input statistics are z-scores rather than p-values, since this workflow invokes metacp with the 'p-values' argument
- A true meta-analysis across multiple rows for the same SNP/gene is needed rather than within-row p-value combination (here the Yes/No meta-analysis flag is set to reproduce the single-table comparison)

## Inputs

- SNP_p_values.txt: p-value file for the 67 SNPs from the type-2 diabetes GWAS, one row per SNP
- correlation_matrix.txt: correlation matrix for the dependent tests
- GWAR reference combined p-values for the same 67 SNPs under MIN2 and MAX conditions

## Outputs

- output.txt: metacp combined p-values for CCT, MinP, MCM and CMC per SNP
- gwar_agreement_table.csv: per-condition agreement table covering MIN2, MAX, CCT, MinP, MCM and CMC with R2 values on -log10 p-values

## How to apply

Load the input p-value file for the 67 SNPs (one row per SNP) and run metacp with the correlation matrix for dependent tests, requesting cct, minp, mcm and cmc methods; MCM combines as 2*min(pCCT, pMinP, 0.5) and CMC applies CCT to pCCT and pMinP. Load the GWAR reference combined p-values for the same SNPs under MIN2 and MAX conditions. Transform all combined p-values (metacp CCT/MinP/MCM/CMC and GWAR MIN2/MAX) to -log10 scale and compute R2 between each metacp condition and each GWAR reference condition. Assemble a per-condition agreement table covering MIN2, MAX, CCT, MinP, MCM and CMC and write it to gwar_agreement_table.csv. The rationale is that metacp computes combined p-values without the complex and time-consuming numerical integrations GWAR requires, so near-identical rankings (R2 of 0.9818 to 1.000) demonstrate consistency at far lower cost.

## Related tools

- **metacp** (Python command-line tool run via metacp.py to produce CCT, MinP, MCM and CMC combined p-values from the 67-SNP p-value file and correlation matrix) — https://github.com/pbagos/metacp
- **Python** (Runtime for metacp.py and for the -log10 transformation and R2 correlation computation against GWAR MIN2/MAX results)

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values Yes correlation_matrix.txt cct minp mcm cmc
```

## Evaluation signals

- R2 values computed on -log10 transformed p-values between each metacp condition (CCT, MinP, MCM, CMC) and each GWAR reference (MIN2, MAX) fall in the range 0.9818 to 1.000
- gwar_agreement_table.csv contains exactly the six conditions MIN2, MAX, CCT, MinP, MCM and CMC
- output.txt contains one combined p-value per SNP per requested method for all 67 SNPs
- metacp output p-values lie in [0, 1] and MCM equals 2*min(pCCT, pMinP, 0.5) per SNP
- Combined p-value rankings agree with GWAR MIN2/MAX despite metacp skipping numerical integrations

## Limitations

- metacp's various implemented methods are not all suitable for the same analysis, and for dependent tests the optimal choice is highly problem dependent
- In some multi-omics studies the correlation structure of the test statistics to be combined is hard to decipher, which prevents supplying the required correlation matrix
- GWAR requires complex and time-consuming numerical integrations, which is precisely what metacp avoids; the agreement is method-specific to CCT/MinP/MCM/CMC vs MIN2/MAX

## Evidence
