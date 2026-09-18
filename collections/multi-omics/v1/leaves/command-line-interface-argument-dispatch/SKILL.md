---
name: command-line-interface-argument-dispatch
description: Use when reconstructing, extending, or re-implementing a CLI tool that
  combines p-values/z-scores and needs a fixed positional-argument order plus keyword
  dispatch to multiple statistical combination methods.
when_to_use_negative:
- Input statistics are neither p-values nor z-scores, or the correlation structure
  of the test statistics is hard to decipher and cannot be supplied or estimated (per
  the discussion, some multi-omics studies fall into this case)
- The analysis requires only one specific combination method already available in
  a narrower tool (poolr, metap) — metacp's broad keyword dispatch is unnecessary
  overhead there
- Working within STATA where argument conventions differ — the README redirects to
  the metacp.hlp file instead of the python CLI
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_0203
tools:
- name: metacp
  role: The Python command-line program whose interface is invoked and whose method
    keywords are dispatched; combines dependent or independent p-values
  repo: github.com/pbagos/metacp
- name: Python
  role: Runtime for metacp.py; the CLI is invoked as 'python metacp.py' with positional
    arguments
- name: STATA
  role: Alternative implementation of metacp with its own documented interface (metacp.hlp)
    rather than the Python CLI
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-025-06126-z
    title: 'metacp: a versatile software package for combining dependent or independent
      p-values'
  source_content_hash: sha256:6f66799bef1917e63d282be65c59d93a50c777bd12a1a8ddc77ce1697f9e7636
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
  - STATA
  - metacp
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# command-line-interface-argument-dispatch

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Designs and documents the ordered command-line interface of metacp (python metacp.py), where positional arguments specify the input file, output file, statistic type (p-values/z-scores), a Yes/No meta-analysis flag, optional correlation and weight matrix files, and one or more method keywords that are dispatched to their statistical implementations.

## When to use

Use when reconstructing, extending, or re-implementing a CLI tool that combines p-values/z-scores and needs a fixed positional-argument order plus keyword dispatch to multiple statistical combination methods.

## When NOT to use

- Input statistics are neither p-values nor z-scores, or the correlation structure of the test statistics is hard to decipher and cannot be supplied or estimated (per the discussion, some multi-omics studies fall into this case)
- The analysis requires only one specific combination method already available in a narrower tool (poolr, metap) — metacp's broad keyword dispatch is unnecessary overhead there
- Working within STATA where argument conventions differ — the README redirects to the metacp.hlp file instead of the python CLI

## Inputs

- Text file of n rows and k columns (plus headers) containing p-values or z-scores to be combined, e.g. SNP_p_values.txt
- Yes/No flag selecting whether meta-analysis of duplicate SNP/gene rows is performed
- Optional text file with a correlation matrix (for dependent p-value methods)
- Optional text file with a weight matrix (for weighted methods such as wstouffer/wcorstouffer)
- One or more method keywords: fisher, stouffer, wstouffer, corstouffer, wcorstouffer, meanp, logit, invchi, binomial, cct, minp, mcm, cmc, bonferroni, adbonferroni, cn, lj, gao, galwey, ebm, kost, yang

## Outputs

- Output text file (e.g. output.txt) containing one combined p-value per input row for every requested combination method

## How to apply

Parse arguments in the documented positional order: (1) input text file of n rows and k columns with headers containing p-values or z-scores, (2) output file, (3) statistic type declared as 'p-values' or 'z-scores', (4) Yes/No flag controlling pre-combination meta-analysis of duplicate rows for the same SNP/gene, (5) optional correlation-matrix file for dependent p-values (if omitted, compute a default correlation matrix from the observations per the requested method), (6) optional weight-matrix file for weighted methods such as wstouffer, and (7+) one or more method keywords. Dispatch each keyword to its implementation: fisher (sum of -2*ln(p) vs chi-squared with 2k df), stouffer/wstouffer/corstouffer/wcorstouffer (normal CDF on summed z-scores with the appropriate variance/weight denominators), meanp (normal CDF on (0.5 - mean p)*sqrt(12k)), logit (t-distribution on summed logits), invchi, binomial, cct/minp/mcm/cmc (Cauchy and minimum-p based methods), bonferroni/adbonferroni/cn/lj/gao/galwey (effective-number-of-tests Bonferroni variants), and ebm/kost/yang (empirical Brown's method family). When the Yes/No flag is Yes, aggregate duplicate rows before combining columns. Finally write one combined p-value per input row for every requested method to the output text file.

## Related tools

- **metacp** (The Python command-line program whose interface is invoked and whose method keywords are dispatched; combines dependent or independent p-values) — github.com/pbagos/metacp
- **Python** (Runtime for metacp.py; the CLI is invoked as 'python metacp.py' with positional arguments)
- **STATA** (Alternative implementation of metacp with its own documented interface (metacp.hlp) rather than the Python CLI)

## Examples

```
python metacp.py SNP_p_values.txt output.txt p-values Yes correlation_matrix.txt weight_matrix.txt fisher meanp bonferroni wstouffer
```

## Evaluation signals

- Argument order check: the invocation must place input file, output file, p-values/z-scores, Yes/No, correlation matrix, weight matrix, and method keywords in the documented positions 1–7+
- Dispatch check: every accepted method keyword maps to a implemented statistical method; unrecognized keywords should not silently produce output
- Output schema check: output.txt contains exactly one combined p-value per input row for each requested method
- Flag semantics check: with Yes, duplicate rows for the same SNP/gene are aggregated before column-wise combination; with No, each row is combined independently
- Cross-tool agreement: combined p-values for cct/minp/mcm/cmc vs GWAR's MAX/MIN2 on the 67-SNP type-2 diabetes GWAS give R2 between 0.9818 and 1.000

## Limitations

- The various implemented methods are not all suitable for the same analysis; for dependent tests the optimal choice is highly problem dependent
- In some multi-omics studies the correlation structure of the test statistics to be combined is hard to decipher, weakening methods that rely on a correlation matrix
- If the correlation-matrix argument is omitted, the program computes a default correlation matrix from the observations according to the specifics of the requested method, which may not match the true dependence structure

## Evidence
