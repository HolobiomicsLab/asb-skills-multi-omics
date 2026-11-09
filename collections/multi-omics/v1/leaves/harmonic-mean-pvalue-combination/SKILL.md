---
name: harmonic-mean-pvalue-combination
description: Use when a row of k p-values (e.g. SNPs from a GWAS) must be collapsed
  into one combined p-value and a robust, integration-free alternative to Fisher or
  Stouffer is wanted, especially when some p-values may be very small.
when_to_use_negative:
- The various methods implemented are not all suitable for the same analysis — do
  not default to HMP when the input statistics are z-scores requiring a Z-based method
  such as stouffer or wstouffer.
- When the correlation structure of the test statistics is known and a correlation-adjusted
  method (cn, lj, gao, galwey, kost) is explicitly required, an unadjusted harmonic
  mean may be the wrong choice.
- When input values are raw statistics rather than p-values — they must first be converted
  or specified as p-values/z-scores for metacp to interpret them correctly.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: metacp
  role: Python/STATA package whose combination framework (input file parsing, output
    file writing, method selection arguments) hosts the HMP computation alongside
    cct, minp, mcm and cmc
  repo: https://github.com/pbagos/metacp
- name: Python
  role: Implementation language for the metacp module that accepts p-value vectors
    and returns combined p-values
- name: GWAR
  role: External reference method used to benchmark combined p-values (R^2 0.9818-1.000
    on 67 SNPs); requires numerical integrations that metacp avoids
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:323cdf636dc1b3cd91a5625a0538775f2999b3e05ab97048fd90ad81b54de92b
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# harmonic-mean-pvalue-combination

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Combines a vector of p-values into a single combined p-value using the Harmonic Mean p-value (HMP) test, p_c = k / sum_i(1/p_i), as one of the robust combination methods offered by metacp alongside CCT, minP, MCM and CMC for dependent or independent tests.

## When to use

Use when a row of k p-values (e.g. SNPs from a GWAS) must be collapsed into one combined p-value and a robust, integration-free alternative to Fisher or Stouffer is wanted, especially when some p-values may be very small.

## When NOT to use

- The various methods implemented are not all suitable for the same analysis — do not default to HMP when the input statistics are z-scores requiring a Z-based method such as stouffer or wstouffer.
- When the correlation structure of the test statistics is known and a correlation-adjusted method (cn, lj, gao, galwey, kost) is explicitly required, an unadjusted harmonic mean may be the wrong choice.
- When input values are raw statistics rather than p-values — they must first be converted or specified as p-values/z-scores for metacp to interpret them correctly.

## Inputs

- Text file of n rows and k columns (plus headers) containing p-values or z-scores, e.g. SNP_p_values.txt
- Optional correlation matrix text file (for dependent p-values; omitted means metacp computes one by default)
- Optional weight matrix text file (for methods that allow weighting)

## Outputs

- Output text file (e.g. output.txt) containing the combined HMP p-value per row, alongside combined p-values from any other requested methods

## How to apply

Parse the input text file of n rows and k columns into numeric vectors of k p-values per row. For each row compute the HMP as p_c = k / sum_i(1/p_i), i.e. the harmonic mean of the individual p-values, which weights small p-values heavily without requiring the complex and time-consuming numerical integrations needed by GWAR. Expose the HMP alongside the other metacp combination methods (cct, minp, mcm, cmc) so all can be requested in one run and written to a user-provided output text file. Validate that each combined p-value lies in [0,1] and, where possible, check agreement with published reference values; the article reports R^2 in the range 0.9818-1.000 for metacp's CCT/MinP/MCM/CMC against GWAR on 67 type-2 diabetes GWAS SNPs, giving a benchmark for cross-method consistency.

## Related tools

- **metacp** (Python/STATA package whose combination framework (input file parsing, output file writing, method selection arguments) hosts the HMP computation alongside cct, minp, mcm and cmc) — https://github.com/pbagos/metacp
- **Python** (Implementation language for the metacp module that accepts p-value vectors and returns combined p-values)
- **GWAR** (External reference method used to benchmark combined p-values (R^2 0.9818-1.000 on 67 SNPs); requires numerical integrations that metacp avoids)

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values No hmp
```

## Evaluation signals

- Every combined p-value returned lies in [0,1] for all n rows of the input file.
- HMP values agree with published reference p-values for the harmonic mean p-value test on standard example vectors.
- Cross-method R^2 against GWAR (MAX, MIN2) on the 67 type-2 diabetes GWAS SNPs falls within the reported 0.9818-1.000 range for the metacp combination methods.
- Combined p-values are produced without numerical integration, so runtime scales linearly with the number of p-values rather than requiring the complex and time-consuming numerical integrations required by GWAR.
- Output file structure matches metacp's convention: one combined p-value per input row, written to the user-provided output text file.

## Limitations

- The various methods implemented are not all suitable for the same analysis; for combination of dependent tests the optimal choice is highly problem dependent.
- In some multi-omics studies the correlation structure of the test statistics to be combined is hard to decipher, complicating method choice.
- If the input file contains z-scores rather than p-values, the statistic form must be specified explicitly or results will be misinterpreted.

## Evidence

- [other] Implement the Harmonic Mean p-value (HMP) test as p_c = k / sum_i(1/p_i).: [evidence span withheld — anchor status: not_found]
