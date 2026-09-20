---
name: recovery-rate-validation-against-reference-proteome
description: Use when a protein recovery or missing-protein prediction method outputs
  per-protein probabilities and you need to quantify the recovery rate against an
  independent reference proteome (e.g. diaPASEF, Astral HeLa) under upper- or lower-bound
  masking.
when_to_use_negative:
- The method output contains only observed protein intensities with no probability
  scores for unreported proteins — there is nothing to validate as a recovery prediction
- No independent reference proteome (e.g. diaPASEF or Astral) exists for the cell
  line or tissue, so recovery rates cannot be computed without circular validation
- You need abundance-level quantification rather than presence/absence recovery validation
  — Protrec2 operates on a binary presence/absence basis
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3520
tools:
- name: Protrec2
  role: Generates the per-sample recovered-protein probability lists that are validated
    against the reference proteomes
  repo: https://github.com/miaomiao6606/Protrec2
- name: PROTREC
  role: Baseline recovery method run under identical upper- and lower-bound masking
    scenarios for recovery-rate comparison
- name: Python
  role: Runtime environment and CLI entry point (main.py) used to execute Protrec2
    and evaluation scripts
  repo: https://github.com/miaomiao6606/Protrec2
- name: pip
  role: Installs Protrec2 dependencies from requirements.txt before running the benchmark
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bib/bbaf692
    title: 'Protrec2: tissue-specific network-based missing protein recovery method'
  source_content_hash: sha256:4c917df210fdbca7724e0fb5d7d230e6d1cd089f62b11106cb0488c58e12b7d5
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
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3520
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# recovery-rate-validation-against-reference-proteome

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates predicted (recovered) missing proteins from Protrec2 by matching per-sample Protein,Probability outputs against independent reference proteomes (diaPASEF meta-analysis and Astral HeLa, 8796 proteins), scoring the fraction of predictions supported at probability cutoffs from 0.50 to 0.95.

## When to use

Use when a protein recovery or missing-protein prediction method outputs per-protein probabilities and you need to quantify the recovery rate against an independent reference proteome (e.g. diaPASEF, Astral HeLa) under upper- or lower-bound masking.

## When NOT to use

- The method output contains only observed protein intensities with no probability scores for unreported proteins — there is nothing to validate as a recovery prediction
- No independent reference proteome (e.g. diaPASEF or Astral) exists for the cell line or tissue, so recovery rates cannot be computed without circular validation
- You need abundance-level quantification rather than presence/absence recovery validation — Protrec2 operates on a binary presence/absence basis

## Inputs

- Per-sample recovery output CSVs with columns Protein and Probability (0-1) from Protrec2
- External reference proteome sets (diaPASEF multi-omics/meta-analysis reference; Astral HeLa reference set of 8796 proteins)
- Expression matrix CSV (rows: proteins as UniProt IDs or symbols; columns: samples) used to construct upper- and lower-bound masking scenarios

## Outputs

- Recovery rates (% of predicted proteins validated) per scenario and cutoff (0.50-0.95)
- Validated-protein counts (e.g. 453 diaPASEF upper-bound, 450 Astral upper-bound for HeLa)
- Comparative recovery curves against baselines (PROTREC, FCS, HE, GSEA)

## How to apply

Run the recovery method (e.g. Protrec2 via 'python main.py --expression <heLa_expression.csv> --output_dir results/ --tissue <tissue> --fdr 0.01 --threshold 5') to produce per-sample CSVs in Protein,Probability format. Mask the expression matrix under upper- and lower-bound scenarios so predictions target proteins held out from the observed list, then collect recovered proteins at probability cutoffs ranging from 0.50 to 0.95. Intersect each prediction list with an external reference proteome — the diaPASEF multi-omics/meta-analysis reference or the Astral HeLa reference set (8796 proteins) — and compute the recovery rate as validated proteins divided by predictions. For HeLa lower-bound validation, expect ~74.6% recovery against diaPASEF and 69.0% against Astral; upper-bound rates should approach 96.5% (diaPASEF, 453 validated proteins) and 96.0% (Astral, 450 validated proteins).

## Related tools

- **Protrec2** (Generates the per-sample recovered-protein probability lists that are validated against the reference proteomes) — https://github.com/miaomiao6606/Protrec2
- **PROTREC** (Baseline recovery method run under identical upper- and lower-bound masking scenarios for recovery-rate comparison)
- **Python** (Runtime environment and CLI entry point (main.py) used to execute Protrec2 and evaluation scripts) — https://github.com/miaomiao6606/Protrec2
- **pip** (Installs Protrec2 dependencies from requirements.txt before running the benchmark)

## Examples

```
python main.py --expression heLa_expression.csv --output_dir results/ --tissue cervical --fdr 0.01 --threshold 5
```

## Evaluation signals

- HeLa lower-bound recovery against the integrated diaPASEF reference should be ~74.6%, close to PROTREC (75.5%) and above FCS (68.7%), HE (62.6%), GSEA (63.4%)
- HeLa upper-bound recovery should reach ~96.5% (diaPASEF, 453 validated proteins) and ~96.0% (Astral, 450 validated proteins)
- Recovery should improve monotonically as the probability cutoff increases from 0.50 to 0.95
- Astral lower-bound recovery for HeLa should be ~69.0%, confirming platform-agnostic agreement across references
- Output CSVs must follow the Protein,Probability schema with probabilities in [0,1]

## Limitations

- Validation performance is constrained by proteomics dataset coverage; reported recovery rates represent lower bounds on true performance
- Protrec2 operates on a binary presence/absence basis and does not incorporate protein abundance, modification status, or subcellular localization
- Tissue-specific annotations were derived from population-level data and might not fully reflect interpatient heterogeneity

## Evidence

- [results] In the HeLa lower-bound scenario validated against the integrated diaPASEF reference, Protrec2 achieved a mean recovery rate of 74.6%: [evidence span withheld — anchor status: not_found]
- [readme] HeLa proteomes: 96.5% recovery rate (453 proteins validated): [evidence span withheld — anchor status: external_doc]
