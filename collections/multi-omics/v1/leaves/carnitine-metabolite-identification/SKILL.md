---
name: carnitine-metabolite-identification
description: Reference for confirming the identity of carnitine/acylcarnitine features
  detected in a yogurt or microbial metabolome when a published standard-injection
  table with m/z, CAS, and mass/RT errors must be retrieved, verified, and reproduced
  as a structured table.
when_to_use_negative:
- The goal is to run the mGWAS association itself (Scoary2 pan-genome vs. metabolite
  traits) rather than to verify metabolite identities from the published standard
  table
- The metabolites of interest are not carnitine-related species, since Table 1 confirmation
  values (m/z, CAS, ppm, RT error) are specific to the five carnitine metabolites
edam_operation: http://edamontology.org/operation_0306
tools:
- name: Scoary2
  role: Source publication whose supplementary Table 1 provides the LC-MS standard-confirmation
    data for the carnitine metabolites
  repo: https://github.com/MrTomRod/scoary-2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:3874450fd3d3ce3dc66cf902e7b79368b61defa1ec37e17f96a0ffc9d4540527
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MrTomRod/scoary-2
  tools:
  - Scoary2
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# carnitine-metabolite-identification

## Summary

Reproduce the LC-MS standard-confirmation records (Table 1 of the Scoary2 paper) for five carnitine-related metabolites — carnitine, acetylcarnitine, hexanoylcarnitine, octanoylcarnitine, and decanoylcarnitine — capturing measured m/z, CAS number, mass error (ppm), and retention-time error from analysis of pure analytical standard solutions.

## When to use

Reference for confirming the identity of carnitine/acylcarnitine features detected in a yogurt or microbial metabolome when a published standard-injection table with m/z, CAS, and mass/RT errors must be retrieved, verified, and reproduced as a structured table.

## When NOT to use

- The goal is to run the mGWAS association itself (Scoary2 pan-genome vs. metabolite traits) rather than to verify metabolite identities from the published standard table
- The metabolites of interest are not carnitine-related species, since Table 1 confirmation values (m/z, CAS, ppm, RT error) are specific to the five carnitine metabolites

## Inputs

- Supplementary Table 1 of the Scoary2 paper (DOI 10.1186/s13059-024-03233-7) containing LC-MS identity confirmation of carnitine species
- List of target metabolite names: carnitine, acetylcarnitine, hexanoylcarnitine, octanoylcarnitine, decanoylcarnitine

## Outputs

- table1_acylcarnitine_identity_confirmation.csv with columns: metabolite, measured_mz, cas_number, mass_error_ppm, rt_error (five records)

## How to apply

Retrieve the supplementary Table 1 of the Scoary2 paper (DOI 10.1186/s13059-024-03233-7), which contains the identity confirmation of carnitine species detected in the yogurt metabolome. For each of the five metabolites, extract the measured m/z (e.g. decanoylcarnitine 316.24764, octanoylcarnitine 288.21635, hexanoylcarnitine 260.18515), CAS number, mass error in ppm, and retention-time error. Check each mass error against the reported <1 ppm accuracy threshold and flag deviations (octanoylcarnitine's 2.02 ppm and hexanoylcarnitine's RT error of 2.13% are the notable reported values). Write the five records to table1_acylcarnitine_identity_confirmation.csv with columns metabolite, measured_mz, cas_number, mass_error_ppm, rt_error, using pandas for extraction and validation.

## Related tools

- **Scoary2** (Source publication whose supplementary Table 1 provides the LC-MS standard-confirmation data for the carnitine metabolites) — https://github.com/MrTomRod/scoary-2

## Examples

```
import pandas as pd; df = pd.read_excel('scoary2_supplementary_table1.xlsx'); df[['metabolite','measured_mz','cas_number','mass_error_ppm','rt_error']].to_csv('table1_acylcarnitine_identity_confirmation.csv', index=False)
```

## Evaluation signals

- Each reproduced value (measured_mz, cas_number, mass_error_ppm, rt_error) matches the published Table 1 entry exactly (output_matches_reference)
- All five metabolite names are present in the output CSV, one record each
- All four fields (m/z, CAS, mass error, RT error) are populated and non-null for every record
- Mass errors are compared against the reported <1 ppm accuracy threshold; deviations such as octanoylcarnitine's 2.02 ppm are explicitly flagged rather than silently passed

## Limitations

- Only decanoylcarnitine, octanoylcarnitine, and hexanoylcarnitine are confirmed by standard injection with reported mass/RT errors; carnitine and acetylcarnitine records in the table may lack the same confirmation detail
- The reported mass errors are not all below the <1 ppm threshold (octanoylcarnitine at 2.02 ppm), so a naive pass/fail check on the threshold would incorrectly flag a published value
- The document's discussion section is not available, so no additional caveats about the LC-MS confirmation procedure could be extracted

## Evidence

- [abstract] we explore the metabolome of yogurts, each produced with a different Propionibacterium reichii strain and discover two genes affecting carnitine metabolism: "we explore the metabolome of yogurts, each produced with a different Propionibacterium reichii strain and discover two genes affecting carnitine metabolism"
- [intro] bacterial strains belonging to 20 different (sub-)species that were selected from the strain collection of Agroscope: "bacterial strains belonging to 20 different (sub-)species that were selected from the strain collection of Agroscope, the Swiss center of excellence for agricultural research"
- [readme] Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder] or [Roary][roary] to traits.: "Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder]
or [Roary][roary] to traits."
