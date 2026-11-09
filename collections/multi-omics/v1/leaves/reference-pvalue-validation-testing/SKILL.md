---
name: reference-pvalue-validation-testing
description: Use when implementing or porting a statistical p-value combination method
  and you need to verify correctness against published reference p-values or an established
  tool's output on the same dataset.
when_to_use_negative:
- When no reference p-values or independent implementation exist for the method, so
  no comparison target is available
- When the goal is to choose among combination methods for a new analysis rather than
  to verify an implementation's correctness
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2269
tools:
- name: metacp
  role: Provides the CCT, MinP, MCM and CMC combined p-values being validated against
    reference values
  repo: https://github.com/pbagos/metacp
- name: Python
  role: Runtime environment in which metacp is invoked and validation/comparison scripts
    are written
- name: GWAR
  role: Reference implementation (MAX, MIN2 methods) whose combined p-values serve
    as the comparison target on the 67 SNPs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:b7762eb3dea57b8248831ed5b7144fe9254a18d57dee8e47406f351c686259ce
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
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# reference-pvalue-validation-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validate that a newly implemented p-value combination method (e.g. metacp's CCT, MinP, MCM, CMC) produces correct combined p-values by checking range invariants and comparing against published reference values from alternative tools such as GWAR.

## When to use

Use when implementing or porting a statistical p-value combination method and you need to verify correctness against published reference p-values or an established tool's output on the same dataset.

## When NOT to use

- When no reference p-values or independent implementation exist for the method, so no comparison target is available
- When the goal is to choose among combination methods for a new analysis rather than to verify an implementation's correctness

## Inputs

- Combined p-values from the implemented method (e.g. output.txt rows of p_CCT, p_MinP, p_MCM, p_CMC per SNP)
- Reference combined p-values from an established tool (e.g. GWAR MAX/MIN2 on the same SNPs)
- Benchmark dataset (67 SNPs from a type-2 diabetes GWAS with p-values per SNP)

## Outputs

- Range check results confirming every combined p-value in [0,1]
- Agreement statistics (R^2) between implemented and reference combined p-values
- Pass/fail verdict per method relative to the reported agreement range 0.9818–1.000

## How to apply

Run the combination method (e.g. metacp's cct, minp, mcm, cmc) on a shared benchmark dataset — here 67 SNPs from a GWAS identifying susceptibility loci for type-2 diabetes — so that the same inputs feed both your implementation and the reference tool (GWAR with MAX/MIN2). First verify each combined p-value lies in [0,1] as a basic invariant. Then compute agreement statistics between your p-values and the reference values; the article reports R^2 values ranging from 0.9818 to 1.000 for metacp's CCT/MinP/MCM/CMC against GWAR, so treat R^2 near 1.0 (and within that reported range) as evidence of correct implementation. Use the workflow step of writing combined p-values to an output file and parsing them for comparison rather than eyeballing, and confirm the implementation avoids the complex numerical integrations GWAR requires, which serves as a performance sanity check.

## Related tools

- **metacp** (Provides the CCT, MinP, MCM and CMC combined p-values being validated against reference values) — https://github.com/pbagos/metacp
- **Python** (Runtime environment in which metacp is invoked and validation/comparison scripts are written)
- **GWAR** (Reference implementation (MAX, MIN2 methods) whose combined p-values serve as the comparison target on the 67 SNPs)

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values No cct minp mcm cmc
```

## Evaluation signals

- Every combined p-value falls in [0,1]
- R^2 between metacp CCT/MinP/MCM/CMC and GWAR MAX/MIN2 values on the 67 SNPs falls within the reported 0.9818–1.000 range
- Combined p-values written to the user-specified output file parse correctly and align row-wise with the input SNPs
- Runtime remains fast, confirming no reliance on the complex and time-consuming numerical integrations required by GWAR

## Limitations

- Agreement with a reference tool (e.g. R^2 of 0.9818–1.000) indicates consistency, not proof of statistical optimality; the article notes the various implemented methods are not all suitable for the same analysis and the optimal choice for dependent tests is highly problem dependent
- In some multi-omics studies the correlation structure of the test statistics to be combined is hard to decipher, complicating interpretation of reference comparisons

## Evidence
