---
name: hybrid-minp-cct-pvalue-combination
description: Use when p-values to be combined are dependent (correlated) — e.g. SNP/gene
  rows from a GWAS or transcriptomics study — and a robust, correlation-free combined
  p-value is needed without numerical integration.
when_to_use_negative:
- The p-values are independent — independent-case methods such as Fisher's, Stouffer's
  or Edgington's (also in metacp) are the standard choice; the article notes the various
  methods are not all suitable for the same analysis
- The correlation structure of the test statistics is precisely known and an exact
  correlation-aware method (e.g. Brown's method or Kost's adjustment) is preferred
  over correlation-free hybrids
- Input statistics are z-scores without conversion — metacp must be told via the p-values/z-scores
  argument, and mismatching this flag invalidates the combination
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: metacp
  role: Python/STATA package that implements cct, minp, mcm and cmc for combining
    dependent p-values and writes combined p-values to an output file
  repo: https://github.com/pbagos/metacp
- name: Python
  role: Runtime for metacp, invoked as 'python metacp.py <input file> <output file>
    <p-values/z-scores> <Yes/No> ... <method1> <method2> ...'
- name: GWAR
  role: Reference tool requiring numerical integrations, used for comparison/validation
    of metacp's CCT/MinP/MCM/CMC outputs (R2 0.9818–1.000)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:c02e49e8d52a001ec18a669992b1b0db9e220c42747a5c5db0cb2bbe4f3c10b1
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
  - GWAR
  - Python
  - metacp
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# hybrid-minp-cct-pvalue-combination

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Combines dependent p-values using the hybrid MinP-Cauchy-MinP (MCM) and Cauchy-MinP-Cauchy (CMC) constructions built on the Cauchy Combination Test (CCT) and Tippett's MinP, as implemented in metacp. These hybrids avoid the complex, time-consuming numerical integrations that comparable tools such as GWAR require.

## When to use

Use when p-values to be combined are dependent (correlated) — e.g. SNP/gene rows from a GWAS or transcriptomics study — and a robust, correlation-free combined p-value is needed without numerical integration.

## When NOT to use

- The p-values are independent — independent-case methods such as Fisher's, Stouffer's or Edgington's (also in metacp) are the standard choice; the article notes the various methods are not all suitable for the same analysis
- The correlation structure of the test statistics is precisely known and an exact correlation-aware method (e.g. Brown's method or Kost's adjustment) is preferred over correlation-free hybrids
- Input statistics are z-scores without conversion — metacp must be told via the p-values/z-scores argument, and mismatching this flag invalidates the combination

## Inputs

- Text file of n rows and k columns containing p-values or z-scores (with headers), e.g. SNP_p_values.txt
- Optional correlation matrix text file for dependent p-values
- Optional weight matrix text file (for methods that allow weighting)
- Method-selection arguments (cct, minp, mcm, cmc) and Yes/No meta-analysis flag

## Outputs

- Text file of combined p-values, one per row of the input (e.g. output.txt)
- Per-row combined p-values for each requested method (p_CCT, p_MinP, p_MCM, p_CMC)

## How to apply

Parse the input text file of n rows and k columns into per-row vectors of k p-values (or z-scores). Compute p_CCT by transforming each p-value with the tangent function, summing the transformed values, and evaluating the upper tail of the standard Cauchy(0,1) distribution; compute p_MinP via Tippett's method as 1 - (1 - min(p_1,...,p_k))^k, taking the minimum p-value and adjusting it for the number of tests. Then form the hybrids: MCM = 2*min(p_CCT, p_MinP, 0.5) and CMC = CCT(p_CCT, p_MinP). In metacp, supply a correlation matrix file for dependent p-values (or omit it so the program calculates one by default per the requested method), and invoke the cct, minp, mcm and cmc methods in the command line; combined p-values are written to the user-specified output text file. Validate that each combined p-value lies in [0,1] and check agreement against GWAR-style MAX/MIN2 results, where the article reports R2 values of 0.9818–1.000 on 67 SNPs from a type-2 diabetes GWAS.

## Related tools

- **metacp** (Python/STATA package that implements cct, minp, mcm and cmc for combining dependent p-values and writes combined p-values to an output file) — https://github.com/pbagos/metacp
- **Python** (Runtime for metacp, invoked as 'python metacp.py <input file> <output file> <p-values/z-scores> <Yes/No> ... <method1> <method2> ...')
- **GWAR** (Reference tool requiring numerical integrations, used for comparison/validation of metacp's CCT/MinP/MCM/CMC outputs (R2 0.9818–1.000))

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values Yes correlation_matrix.txt weight_matrix.txt cct minp mcm cmc
```

## Evaluation signals

- Every combined p-value lies in [0,1]
- MCM result equals 2*min(p_CCT, p_MinP, 0.5) and CMC equals the CCT applied to (p_CCT, p_MinP) on the same input row
- Agreement with GWAR MAX/MIN2 combined p-values falls within the reported R2 range of 0.9818–1.000 on the 67-SNP type-2 diabetes GWAS benchmark
- Combined p-values are reproduced without numerical integration (runtime far below GWAR's integration-based computation)
- Output file contains one combined p-value per input row and one column per requested method

## Limitations

- The various implemented methods are not all suitable for the same analysis; for dependent tests the optimal choice is highly problem dependent
- In some multi-omics studies the correlation structure of the test statistics to be combined is hard to decipher, which limits correlation-aware alternatives
- If the correlation matrix argument is omitted, metacp calculates a default correlation matrix from the observations according to the specifics of the requested method, which may not match the true correlation structure

## Evidence
