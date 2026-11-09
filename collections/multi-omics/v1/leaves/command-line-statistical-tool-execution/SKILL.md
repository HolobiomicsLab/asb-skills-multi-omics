---
name: command-line-statistical-tool-execution
description: Use when you have a text file of p-values or z-scores (n rows x k columns,
  e.g. SNPs from a GWAS) and need combined per-row p-values via one or more named
  p-value combination methods from the terminal.
when_to_use_negative:
- The input statistics are not p-values or z-scores, or the rows/columns do not correspond
  to variables the user can correctly specify for combination (e.g. mixing unrelated
  variables)
- A single p-value combination method with custom numerical integration behavior is
  needed that only another tool (e.g. GWAR) provides
- The input is a single-column file with no combination to perform and no meta-analysis
  across rows requested
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: metacp
  role: The command-line Python tool whose metacp.py script is invoked to combine
    p-values/z-scores by named methods
  repo: github.com/pbagos/metacp
- name: Python
  role: Runtime for executing metacp.py from the terminal/command line
- name: poolr
  role: R package used to run the equivalent shared methods (fisher, stouffer, wstouffer,
    invchi, galwey, kost) for cross-package verification of metacp output
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:21806632c17303cf4597d0f314f6c1186024739d22ea6eed4144cd6c146edb81
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
  - poolr
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# command-line-statistical-tool-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Execute the metacp Python command-line tool to combine dependent or independent p-values (or z-scores) by passing an input file, output file, statistic type, meta-analysis flag, optional correlation/weight matrices, and one or more of ~20 named combination methods (fisher, stouffer, wstouffer, invchi, galwey, kost, cct, minp, mcm, cmc, etc.).

## When to use

Use when you have a text file of p-values or z-scores (n rows x k columns, e.g. SNPs from a GWAS) and need combined per-row p-values via one or more named p-value combination methods from the terminal.

## When NOT to use

- The input statistics are not p-values or z-scores, or the rows/columns do not correspond to variables the user can correctly specify for combination (e.g. mixing unrelated variables)
- A single p-value combination method with custom numerical integration behavior is needed that only another tool (e.g. GWAR) provides
- The input is a single-column file with no combination to perform and no meta-analysis across rows requested

## Inputs

- Input text file of p-values or z-scores (n rows x k columns with headers), e.g. SNP_p_values.txt for 67 SNPs from a type-2 diabetes GWAS
- Optional correlation matrix text file (for dependent p-value methods)
- Optional weight matrix text file (for weighted methods such as wstouffer)
- Method name arguments (logit, meanp, fisher, stouffer, wstouffer, invchi, binomial, cct, minp, mcm, cmc, bonferroni, adbonferroni, cn, lj, gao, galwey, ebm, kost, yang, corstouffer, wcorstouffer)

## Outputs

- Output text file (e.g. metacp_output.txt) containing the combined p-value for each input row and each requested method
- Side-by-side comparison table (e.g. comparison.csv) when used for cross-package reproducibility checks

## How to apply

Build a metacp.py command with the positional arguments documented in the README: input file (e.g. SNP_p_values.txt), output file, statistic type ('p-values' or 'z-scores'), a Yes/No flag for whether to meta-analyze rows of the same SNP/gene, then optionally a correlation matrix file (for dependent p-values; if omitted metacp calculates one by default per the requested method) and a weight matrix file (for weighted methods), followed by the method names to run. For the reproducibility check described in the source task, run metacp with 'No' meta-analysis and methods such as fisher, stouffer, wstouffer, invchi, galwey, kost, then run the equivalent shared methods in the poolr R package and the Robust tests package on the same input. Consolidate combined p-values from all packages into a side-by-side comparison table (one row per SNP, paired columns per package-method) and verify numerical identity per method. Correct execution means the output file exists, contains one combined p-value per input row per requested method, and shared methods match across packages.

## Related tools

- **metacp** (The command-line Python tool whose metacp.py script is invoked to combine p-values/z-scores by named methods) — github.com/pbagos/metacp
- **Python** (Runtime for executing metacp.py from the terminal/command line)
- **poolr** (R package used to run the equivalent shared methods (fisher, stouffer, wstouffer, invchi, galwey, kost) for cross-package verification of metacp output)

## Examples

```
python metacp.py SNP_p_values.txt metacp_output.txt p-values No fisher stouffer wstouffer invchi galwey kost
```

## Evaluation signals

- The output file exists and contains one combined p-value per input row for every requested method
- For shared methods (fisher, stouffer, wstouffer, invchi, galwey, kost), metacp combined p-values are numerically identical to poolr and the Robust tests package on the same input (finding: identical for every shared method)
- Combined p-values fall in [0,1] for every row and method
- Setting the meta-analysis flag to 'No' with no matrix files still runs correctly, with metacp computing a default correlation matrix from the observations per the requested method
- For dependent-p-value methods (CCT, MinP, MCM, CMC), results are consistent with GWAR (MAX, MIN2) with R2 between 0.9818 and 1.000 on the 67-SNP dataset

## Limitations

- The various implemented methods are not all suitable for the same analysis; for dependent tests the optimal choice is highly problem dependent
- In some multi-omics studies the correlation structure of the test statistics to be combined is hard to decipher, complicating dependent-p-value methods
- If the correlation matrix argument is omitted, metacp calculates a default correlation matrix from the observations, which may not match the study's true correlation structure

## Evidence
