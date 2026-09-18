---
name: proteomics-expression-matrix-formatting
description: Use when preparing a protein-by-sample abundance table (e.g., from DDA-PASEF
  or Astral runs) for Protrec2 recovery, or when a CSV fails because protein IDs or
  the rows/columns layout do not match the expected format.
when_to_use_negative:
- Input is already a Protrec2 output per-sample CSV in Protein,Probability format
  — that is a downstream result, not an expression matrix
- Identifiers are not mappable to UniProt IDs or gene symbols (complexes are filtered
  for >80% valid UniProt IDs, so non-conforming IDs break recovery)
- The analysis requires protein abundance, modification status, or subcellular localization
  — Protrec2 operates on a binary presence/absence basis
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Protrec2
  role: Consumer of the formatted expression matrix via the --expression argument
    of main.py
  repo: https://github.com/miaomiao6606/Protrec2
- name: Python
  role: Runtime environment in which main.py loads and validates the CSV expression
    matrix
- name: pip
  role: Installs package dependencies (pip install -r requirements.txt) needed to
    run main.py on the formatted matrix
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:63d4bdbf3bf22448a25a95671ec6872095af280fb71a326d826b448f6fe8af46
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# proteomics-expression-matrix-formatting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Formats a mass spectrometry proteomics expression matrix (e.g., HeLa DDA-PASEF with 3 samples) as a CSV with protein identifiers as rows and sample columns so it can serve as the --expression input to Protrec2's Bayesian missing-protein recovery.

## When to use

Use when preparing a protein-by-sample abundance table (e.g., from DDA-PASEF or Astral runs) for Protrec2 recovery, or when a CSV fails because protein IDs or the rows/columns layout do not match the expected format.

## When NOT to use

- Input is already a Protrec2 output per-sample CSV in Protein,Probability format — that is a downstream result, not an expression matrix
- Identifiers are not mappable to UniProt IDs or gene symbols (complexes are filtered for >80% valid UniProt IDs, so non-conforming IDs break recovery)
- The analysis requires protein abundance, modification status, or subcellular localization — Protrec2 operates on a binary presence/absence basis

## Inputs

- Raw protein abundance table from MS-based proteomics (e.g., HeLa DDA-PASEF expression matrix with 3 samples)
- Protein identifiers as UniProt accessions or gene symbols
- Numeric, non-negative expression values per sample (0 for missing)

## Outputs

- CSV expression matrix with an unnamed first column of protein IDs, one column per sample, e.g. ',Sample1,Sample2,Sample3' header rows like 'P12345,10.5,0,8.3'
- Validated --expression input file compatible with main.py (e.g., example/example_expression.csv format)

## How to apply

Build a CSV whose rows are protein identifiers (UniProt IDs or gene symbols) and whose columns are sample names, with numeric non-negative expression values (zeros denote undetected/missing proteins). An unnamed first column holds the protein identifier, as in the shipped example/example_expression.csv with columns Sample1, Sample2, Sample3. Verify that protein IDs resolve to UniProt-format accessions, since complexes are filtered for >80% valid UniProt IDs downstream; keep values non-negative or the run may recover no proteins. For large datasets that hit memory errors, batch samples rather than reshaping the matrix. The formatted matrix is then passed directly to main.py via --expression alongside --tissue, --fdr 0.01, and --threshold 5.

## Related tools

- **Protrec2** (Consumer of the formatted expression matrix via the --expression argument of main.py) — https://github.com/miaomiao6606/Protrec2
- **Python** (Runtime environment in which main.py loads and validates the CSV expression matrix)
- **pip** (Installs package dependencies (pip install -r requirements.txt) needed to run main.py on the formatted matrix)

## Examples

```
python main.py --expression example/example_expression.csv --output_dir results/ --tissue lung --fdr 0.01 --threshold 5
```

## Evaluation signals

- CSV has an unnamed first column of protein IDs and one column per sample, matching the documented example header ',Sample1,Sample2,Sample3'
- All expression values are numeric and non-negative; negative values trigger the documented 'No proteins recovered' failure mode
- Protein identifiers resolve to UniProt IDs or gene symbols; UniProt format is verified per the README troubleshooting guidance
- The formatted file loads in main.py without error and produces per-sample outputs named PROTREC2_result_{sample_name}.csv with Protein,Probability columns

## Limitations

- Large expression matrices can cause memory errors; README advises processing samples in batches or reducing the complex size threshold
- Wrong formatting (non-numeric, negative, or malformed IDs) leads to 'No proteins recovered'; check expression matrix format and ensure non-negative values
- The binary presence/absence formulation discards abundance, modification status, and subcellular localization information during formatting
- Tissue-specific priors are population-level and may not reflect interpatient heterogeneity

## Evidence

- [methods] Rows: proteins (UniProt ID or symbol)
- Columns: samples: "Rows: proteins (UniProt ID or symbol)
- Columns: samples"
- [readme] Rows: Protein identifiers (UniProt IDs or gene symbols)
  - Columns: Sample names
  - Values: Expression levels (numeric, non-negative): "- **Structure**: 
  - Rows: Protein identifiers (UniProt IDs or gene symbols)
  - Columns: Sample names
  - Values: Expression levels (numeric, non-negative)"
- [readme] Check expression matrix format
   - Verify protein IDs are UniProt format
   - Ensure expression values are non-negative: "2. **No proteins recovered**:
   - Check expression matrix format
   - Verify protein IDs are UniProt format
   - Ensure expression values are non-negative"
