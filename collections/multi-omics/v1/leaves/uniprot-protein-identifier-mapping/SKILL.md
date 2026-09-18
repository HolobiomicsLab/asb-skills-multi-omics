---
name: uniprot-protein-identifier-mapping
description: 'Use when an input expression matrix (rows: proteins, columns: samples)
  mixes or lacks UniProt IDs, since Protrec2 filters complexes for biological relevance
  (>80% valid UniProt IDs) and fails to recover proteins whose identifiers cannot
  be matched to the complex database.'
when_to_use_negative:
- The expression matrix already uses consistent UniProt IDs or gene symbols matching
  Protrec2's expected format — no mapping step is needed
- The analysis goal is differential abundance quantification rather than missing-protein
  recovery; identifier mapping alone will not recover absent rows
- Identifiers are for non-human organisms, since Protrec2's complex resources (CORUM,
  Complex Portal, Reactome, STRING) and 27 tissue priors are human-specific
edam_operation: http://edamontology.org/operation_1811
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Protrec2
  role: Downstream consumer of the mapped expression matrix; its complex-loading step
    filters complexes for biological relevance (>80% valid UniProt IDs), so identifier
    quality directly determines recovery
  repo: https://github.com/miaomiao6606/Protrec2
- name: Python
  role: Used to run main.py on the normalized expression matrix and to pre-validate
    identifier formats before recovery
- name: pip
  role: Installs Protrec2 dependencies (pip install -r requirements.txt) in the environment
    where the mapping and recovery workflow runs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:8732cd96d0ae69953c1e909980bb198ccd0045774bdb2aeaac6163c6e7937aa0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/miaomiao6606/Protrec2
  repo_url: https://github.com/miaomiao6606/Protrec2
  tools:
  - PROTREC
  - Protrec2
  - Python
  - pip
  edam_operation: http://edamontology.org/operation_1811
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.36363636363636365
n_claims: 11
---

# uniprot-protein-identifier-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Normalize protein identifiers in an expression matrix to consistent UniProt IDs (or gene symbols) so that network-based recovery tools such as Protrec2 can match the matrix rows against its built-in protein complex databases (CORUM, Complex Portal, Reactome, STRING-expanded complexes) and tissue-specific annotations.

## When to use

Use when an input expression matrix (rows: proteins, columns: samples) mixes or lacks UniProt IDs, since Protrec2 filters complexes for biological relevance (>80% valid UniProt IDs) and fails to recover proteins whose identifiers cannot be matched to the complex database.

## When NOT to use

- The expression matrix already uses consistent UniProt IDs or gene symbols matching Protrec2's expected format — no mapping step is needed
- The analysis goal is differential abundance quantification rather than missing-protein recovery; identifier mapping alone will not recover absent rows
- Identifiers are for non-human organisms, since Protrec2's complex resources (CORUM, Complex Portal, Reactome, STRING) and 27 tissue priors are human-specific

## Inputs

- Expression matrix CSV with rows as protein identifiers (UniProt IDs or gene symbols) and columns as samples, e.g. example/example_expression.csv
- Unnormalized protein identifier lists (gene symbols, IPI/RefSeq/Ensembl accessions) requiring conversion to UniProt

## Outputs

- Normalized expression matrix CSV whose rows are consistently keyed UniProt IDs (or gene symbols) with non-negative numeric expression values
- Mapping table linking original identifiers to UniProt accessions used by the recovery pipeline

## How to apply

Before running Protrec2, inspect the first column of the expression CSV and determine whether the protein identifiers are UniProt accessions (e.g. P12345) or gene symbols; the tool accepts either but requires internal consistency. Map non-UniProt identifiers to UniProt accessions so each row can be joined to the built-in complex resources (`data/complex_list.csv`, `human_complexes_2022.txt` CORUM complexes, Complex Portal, Reactome, STRING-expanded complexes with confidence > 780) and the tissue-specific priors in `data/updated_tissue_complex_*.csv`. Ensure expression values are numeric and non-negative, then run the pipeline with `--tissue lung --fdr 0.01 --threshold 5`; proteins that remain unmapped will be invisible to the Bayesian recovery step that integrates prior detection probability, cocomplex structural support, and tissue-specific complex confidence. The rationale is that Protrec2's complex-based matching operates on identifier equivalence: any identifier that does not resolve to the same namespace as the complex database prevents cocomplex support and therefore depresses recovery probability.

## Related tools

- **Protrec2** (Downstream consumer of the mapped expression matrix; its complex-loading step filters complexes for biological relevance (>80% valid UniProt IDs), so identifier quality directly determines recovery) — https://github.com/miaomiao6606/Protrec2
- **Python** (Used to run main.py on the normalized expression matrix and to pre-validate identifier formats before recovery)
- **pip** (Installs Protrec2 dependencies (pip install -r requirements.txt) in the environment where the mapping and recovery workflow runs)

## Examples

```
python main.py --expression example/example_expression.csv --output_dir results/ --tissue lung --fdr 0.01 --threshold 5
```

## Evaluation signals

- Protrec2's complex generation step filters for biological relevance (>80% valid UniProt IDs); a mapped matrix should keep this filter non-degenerate rather than discarding most complexes
- Output files PROTREC2_result_{sample_name}.csv should contain a Protein column with UniProt IDs or gene symbols in the same namespace as the input rows
- Tissue type is recognized by main.py (e.g. --tissue lung among the 27 available tissues); an unrecognized tissue or zero recovered proteins usually indicates a format/identifier problem, per the README troubleshooting section
- Recovery on a benchmark matrix (e.g. A549 24-sample DDA-PASEF) should reach the expected magnitude (lower-bound mean recovery 90.8%, upper-bound up to 98.4% for Protrec2); far lower rates suggest identifier mismatch
- All expression values in the mapped matrix must be numeric and non-negative, matching the documented input format

## Limitations

- Protrec2 depends on identifier equivalence with its built-in complex databases; identifiers outside the UniProt/gene-symbol namespace cannot gain cocomplex support
- The protein complex repository (3,885 complexes) remains incomplete for novel or condition-specific complexes, so correctly mapped proteins may still lack cocomplex evidence
- Tissue-specific annotations were derived from population-level data and might not fully reflect interpatient heterogeneity

## Evidence

- [methods] Rows: proteins (UniProt ID or symbol)
- Columns: samples: "Rows: proteins (UniProt ID or symbol)
- Columns: samples"
- [readme] Filter for biological relevance (>80% valid UniProt IDs): [evidence span withheld — anchor status: external_doc]
- [readme] Check expression matrix format
   - Verify protein IDs are UniProt format
   - Ensure expression values are non-negative: "Check expression matrix format
   - Verify protein IDs are UniProt format
   - Ensure expression values are non-negative"
- [methods] python main.py \
  --expression example/example_expression.csv \
  --output_dir results/ \
  --tissue lung \
  --fdr 0.01 \
  --threshold 5: "python main.py \
  --expression example/example_expression.csv \
  --output_dir results/ \
  --tissue lung \
  --fdr 0.01 \
  --threshold 5"
