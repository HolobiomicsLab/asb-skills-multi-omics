---
name: correlation-matrix-specified-meta-analysis
description: Use when the input is a text file of n rows x k columns of p-values or
  z-scores from tests that are not independent (e.g. correlated SNPs in a GWAS) and
  a correlation matrix can be provided or derived.
when_to_use_negative:
- When the tests to combine are genuinely independent — plain Fisher, Stouffer, or
  Edgington combinations without a correlation matrix suffice
- When the correlation structure of the test statistics is hard to decipher (e.g.
  some multi-omics studies), since the optimal dependent-test choice is highly problem
  dependent
- When the input statistics are raw summary statistics rather than p-values or z-scores
  — metacp requires the 3rd argument to specify p-values or z-scores
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: metacp
  role: Runs the p-value combination, accepting a correlation matrix file for dependent
    p-values and writing combined p-values to the output file
  repo: github.com/pbagos/metacp
- name: Python
  role: Execution environment for the metacp.py command-line script
  repo: github.com/pbagos/metacp
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:e9d0ab3a4b62833ac330f33ee5d62df7aed9caff5e10ebeb77ef1a4d7358ac03
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/pbagos/metacp
  tools:
  - Python
  - metacp
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# correlation-matrix-specified-meta-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Combine dependent p-values with metacp by supplying an explicit correlation matrix (5th CLI argument) so that correlation-aware methods (cct, minp, mcm, cmc, corstouffer, brown-type adjustments) account for dependence among tests instead of assuming independence.

## When to use

Use when the input is a text file of n rows x k columns of p-values or z-scores from tests that are not independent (e.g. correlated SNPs in a GWAS) and a correlation matrix can be provided or derived.

## When NOT to use

- When the tests to combine are genuinely independent — plain Fisher, Stouffer, or Edgington combinations without a correlation matrix suffice
- When the correlation structure of the test statistics is hard to decipher (e.g. some multi-omics studies), since the optimal dependent-test choice is highly problem dependent
- When the input statistics are raw summary statistics rather than p-values or z-scores — metacp requires the 3rd argument to specify p-values or z-scores

## Inputs

- Input text file of n rows and k columns plus headers containing p-values or z-scores (e.g. SNP_p_values.txt)
- Correlation matrix text file for the dependent tests (e.g. correlation_matrix.txt)
- Optional weight matrix text file for weighted methods (e.g. weight_matrix.txt)

## Outputs

- Output text file (e.g. output.txt) with combined p-values, one per row, for each requested method

## How to apply

Prepare the input file (e.g. SNP_p_values.txt) with one row per SNP/gene and columns of p-values, then run metacp passing 'Yes' or 'No' for meta-analysis, a correlation_matrix.txt file as the 5th argument, an optional weight_matrix.txt as the 6th, and the dependent-test methods to run (cct, minp, mcm, cmc, corstouffer, etc.). If the correlation matrix argument is omitted, metacp calculates a default correlation matrix from the observations according to the specifics of the requested method — supply one explicitly when the dependence structure is known. MCM computes 2*min(pCCT, pMinP, 0.5) and CMC applies CCT to pCCT and pMinP; combined p-values are written to the user-specified output file. Rationale: for the 67-SNP type-2 diabetes GWAS re-analysis, supplying the correlation matrix and running cct/minp/mcm/cmc gave -log10 p-values correlating with GWAR MIN2/MAX at R2 between 0.9818 and 1.000, without GWAR's costly numerical integrations.

## Related tools

- **metacp** (Runs the p-value combination, accepting a correlation matrix file for dependent p-values and writing combined p-values to the output file) — github.com/pbagos/metacp
- **Python** (Execution environment for the metacp.py command-line script) — github.com/pbagos/metacp

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values Yes correlation_matrix.txt cct minp mcm cmc
```

## Evaluation signals

- Output file contains one combined p-value per input row for each requested dependent-test method (cct, minp, mcm, cmc)
- On the 67-SNP type-2 diabetes GWAS benchmark, R2 on -log10 p-values against GWAR MIN2/MAX falls in the range 0.9818 to 1.000
- The command exits without error and the 5th argument is a square correlation matrix consistent with the number of columns of the input file
- MCM output equals 2*min(pCCT, pMinP, 0.5) and CMC output matches CCT applied to pCCT and pMinP

## Limitations

- If no correlation matrix is provided, metacp calculates one by default from the observations according to the specifics of the requested method, which may not reflect the true dependence structure
- The various implemented methods are not all suitable for the same analysis; for dependent tests the optimal choice is highly problem dependent
- In some multi-omics studies the correlation structure of the test statistics to be combined is hard to decipher

## Evidence
