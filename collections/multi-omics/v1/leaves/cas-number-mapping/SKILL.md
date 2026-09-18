---
name: cas-number-mapping
description: Use when a metabolomics result (e.g. features detected in yogurt samples
  by LC-MS) requires unambiguous chemical identity confirmation via CAS numbers and
  reference standard mass/retention-time error checks.
when_to_use_negative:
- The metabolites of interest were not confirmed by pure analytical standard injection
  — no CAS number or mass/RT error can be claimed from Table 1
- The task is the mGWAS association itself (linking carnitine phenotypes to Propionibacterium
  reichii pan-genomes) rather than metabolite identity annotation
- The source table values are already present in your dataset — do not re-extract,
  just join on CAS number
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: Scoary2
  role: Source publication whose supplementary Table 1 provides the CAS numbers, measured
    m/z, mass errors, and retention-time errors for the carnitine species detected
    in the yogurt metabolome
  repo: https://github.com/MrTomRod/scoary-2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:8975d7db4379a328b8a296a79143f1b0988b7359bad8bf2cbfa84624dabd852f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MrTomRod/scoary-2
  tools:
  - Scoary2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# cas-number-mapping

## Summary

Annotates LC-MS-detected metabolites (e.g. carnitine and acylcarnitines in the yogurt metabolome) with CAS registry numbers by reproducing a published standard-confirmation table that ties each metabolite to its measured m/z, mass error in ppm, and retention-time error from pure analytical standard injections.

## When to use

Use when a metabolomics result (e.g. features detected in yogurt samples by LC-MS) requires unambiguous chemical identity confirmation via CAS numbers and reference standard mass/retention-time error checks.

## When NOT to use

- The metabolites of interest were not confirmed by pure analytical standard injection — no CAS number or mass/RT error can be claimed from Table 1
- The task is the mGWAS association itself (linking carnitine phenotypes to Propionibacterium reichii pan-genomes) rather than metabolite identity annotation
- The source table values are already present in your dataset — do not re-extract, just join on CAS number

## Inputs

- Supplementary Table 1 (identity confirmation of carnitine species) from the Scoary2 paper
- List of carnitine-related metabolites: decanoylcarnitine, octanoylcarnitine, hexanoylcarnitine, carnitine, acetylcarnitine

## Outputs

- table1_acylcarnitine_identity_confirmation.csv with columns metabolite, measured_mz, cas_number, mass_error_ppm, rt_error
- Per-metabolite flags for entries whose mass error exceeds the <1 ppm threshold

## How to apply

Retrieve the supplementary table (Table 1 of the Scoary2 paper, DOI 10.1186/s13059-024-03233-7) that reports identity confirmation for carnitine-related metabolites confirmed by injecting pure analytical standard solutions. For each metabolite — decanoylcarnitine, octanoylcarnitine, hexanoylcarnitine, carnitine, acetylcarnitine — extract the measured m/z, CAS number, mass error in ppm, and retention-time error using pandas. Check each mass error against the reported <1 ppm accuracy threshold and flag any entry that deviates (e.g. octanoylcarnitine at 2.02 ppm). Write the five records to a CSV (e.g. table1_acylcarnitine_identity_confirmation.csv) with columns metabolite, measured_mz, cas_number, mass_error_ppm, rt_error, and validate each reproduced value against the published entry.

## Related tools

- **Scoary2** (Source publication whose supplementary Table 1 provides the CAS numbers, measured m/z, mass errors, and retention-time errors for the carnitine species detected in the yogurt metabolome) — https://github.com/MrTomRod/scoary-2

## Examples

```
import pandas as pd; df = pd.read_csv('table1_acylcarnitine_identity_confirmation.csv'); assert set(df.columns) == {'metabolite','measured_mz','cas_number','mass_error_ppm','rt_error'}; print(df[df.mass_error_ppm >= 1])
```

## Evaluation signals

- Each reproduced value matches the published Table 1 entry (output_matches_reference), e.g. decanoylcarnitine m/z 316.24764 / CAS 3992-45-8, octanoylcarnitine m/z 288.21635 / CAS 25243-95-2, hexanoylcarnitine m/z 260.18515 / CAS 22671-29-0
- All mass errors in the output satisfy the reported <1 ppm accuracy threshold or are explicitly flagged (octanoylcarnitine at 2.02 ppm deviates)
- All five metabolite names plus the m/z, CAS, and error fields are present in the output CSV
- Schema check: the CSV contains exactly the columns metabolite, measured_mz, cas_number, mass_error_ppm, rt_error
- Retention-time errors reproduce the published values (decanoylcarnitine 0.67%, octanoylcarnitine 0.47%, hexanoylcarnitine 2.13%)

## Limitations

- Only the carnitine-related metabolites confirmed by pure standard injection carry CAS numbers and error metrics; unconfirmed features cannot be annotated this way
- The mass-error threshold is <1 ppm as reported, but at least one table entry (octanoylcarnitine, 2.02 ppm) deviates from that threshold and must be flagged rather than silently accepted
- Retention-time errors are reported as percentages and should not be conflated with ppm mass errors

## Evidence

- [supplementary] Table 1 confirms decanoylcarnitine (measured m/z 316.24764, CAS 3992-45-8, mass error <1 ppm, retention-time error 0.67%), octanoylcarnitine (m/z 288.21635, CAS 25243-95-2, mass error 2.02 ppm, retention-time error 0.47%): [evidence span withheld — anchor status: not_found]
- [abstract] we explore the metabolome of yogurts, each produced with a different Propionibacterium reichii strain and discover two genes affecting carnitine metabolism: "we explore the metabolome of yogurts, each produced with a different Propionibacterium reichii strain and discover two genes affecting carnitine metabolism"
