---
name: meta-analysis-of-duplicate-rows
description: Use when the metacp input text file (n rows x k columns of p-values or
  z-scores) contains multiple rows for the same SNP/gene/probe and per-variable combined
  p-values should be computed after a prior meta-analysis of the duplicate rows.
when_to_use_negative:
- When each row of the input file is a unique SNP/gene and no duplicate-row aggregation
  is needed — set the 4th argument to 'No' instead
- When the statistics are not p-values or z-scores tabulated per variable, since the
  program expects a text file of n rows and k columns of these statistics
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0092
tools:
- name: metacp
  role: Python command-line program that performs the duplicate-row meta-analysis
    flag ('Yes'/'No') and then combines p-values/z-scores by requested methods
  repo: https://github.com/pbagos/metacp
- name: Python
  role: Runtime for metacp.py, invoked as 'python metacp.py' on the command line
  repo: https://github.com/pbagos/metacp
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:0178affefd05226976ffd74a9e2303dd503d2b39b7df775197ef66c8d3a4f454
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
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# meta-analysis-of-duplicate-rows

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pre-combination meta-analysis step in metacp: when the same SNP/gene appears in multiple rows of the input p-value/z-score table, setting the 4th argument to 'Yes' aggregates duplicate rows before combining p-values across columns, avoiding pseudo-replication.

## When to use

Use when the metacp input text file (n rows x k columns of p-values or z-scores) contains multiple rows for the same SNP/gene/probe and per-variable combined p-values should be computed after a prior meta-analysis of the duplicate rows.

## When NOT to use

- When each row of the input file is a unique SNP/gene and no duplicate-row aggregation is needed — set the 4th argument to 'No' instead
- When the statistics are not p-values or z-scores tabulated per variable, since the program expects a text file of n rows and k columns of these statistics

## Inputs

- Text file of n rows and k columns (plus headers) containing p-values or z-scores, where different rows may correspond to SNPs/genes from a GWAS or probes/genes from transcriptomics studies
- Optional text file with the correlation matrix (for dependent p-values)
- Optional text file with the weight matrix (for weighting methods such as wstouffer)

## Outputs

- User-specified output text file containing one combined p-value per input row for every requested combination method

## How to apply

Run metacp via 'python metacp.py' with the input file, output file, a 'p-values' or 'z-scores' flag, and set the 4th argument to 'Yes' to trigger meta-analysis of duplicate rows. metacp then identifies rows sharing the same SNP/gene identifier and aggregates their statistics prior to combining the different p-values from different columns. The 5th argument (optional correlation matrix file for dependent p-values) and 6th argument (optional weight matrix file for weighted methods like wstouffer) follow; if the correlation matrix is omitted, metacp calculates one by default from the observations according to the specifics of the requested method. Finally list one or more method keywords (e.g. fisher, meanp, bonferroni, wstouffer) and metacp writes one combined p-value per row for each requested method to the output text file. If duplicates do not exist or a per-row direct combination is desired, set the flag to 'No' and omit the matrix arguments.

## Related tools

- **metacp** (Python command-line program that performs the duplicate-row meta-analysis flag ('Yes'/'No') and then combines p-values/z-scores by requested methods) — https://github.com/pbagos/metacp
- **Python** (Runtime for metacp.py, invoked as 'python metacp.py' on the command line) — https://github.com/pbagos/metacp

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values Yes correlation_matrix.txt weight_matrix.txt fisher meanp bonferroni wstouffer
```

## Evaluation signals

- The command includes 'Yes' as the 4th positional argument only when duplicate SNP/gene rows are present; with 'No' the invocation omits the correlation/weight matrix arguments in the single-run form
- Output text file contains exactly one combined p-value per row per requested method keyword, matching the rows of the input file
- For dependent-p-value methods (e.g. corstouffer, cct, kost), the correlation matrix file is supplied or a default matrix is computed from the observations
- For weighted methods (wstouffer, wcorstouffer), a weight matrix file is provided in the 6th argument
- Agreement with an independent tool: on 67 SNPs from a type-2 diabetes GWAS, R2 values between metacp (CCT, MinP, MCM, CMC) and GWAR range from 0.9818 to 1.000

## Limitations

- The various implemented methods are not all suitable for the same analysis; for combination of dependent tests the optimal choice is highly problem dependent
- In some multi-omics studies the correlation structure of the test statistics to be combined is hard to decipher
- No changelog was found for the repository

## Evidence
