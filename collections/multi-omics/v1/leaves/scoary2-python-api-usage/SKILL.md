---
name: scoary2-python-api-usage
description: Use when you need to call Scoary2 as a Python library (github.com/MrTomRod/scoary-2)
  to simulate, batch, or script pan-genome association runs — e.g. planting causal
  genes in simulated presence/absence matrices and extracting ranked p-values for
  numeric traits.
when_to_use_negative:
- Your traits are already binary (e.g. resistance/susceptibility labels) — automatic
  binarization of continuous traits via k-means/GMM is unnecessary and may distort
  the calls
- You need interactive exploration of one finished analysis rather than scripted or
  simulated runs — the static HTML/JS data exploration app is the intended interface
  there
- Your input is raw genome assemblies or protein FASTA rather than a pan-genome presence/absence
  matrix — first run an orthology tool such as OrthoFinder or Roary
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Scoary2
  role: Core tool invoked via its Python library API to perform automatic binarization
    of continuous numeric traits and mGWAS association testing on pan-genome presence/absence
    matrices
  repo: https://github.com/MrTomRod/scoary-2
- name: OrthoFinder
  role: Upstream orthology inference used to generate the orthogene presence/absence
    matrix that Scoary2 consumes as input
  repo: https://github.com/davidemms/OrthoFinder/
- name: Roary
  role: Alternative upstream pan-genome pipeline whose presence/absence output can
    be used as Scoary2 input
  repo: https://sanger-pathogens.github.io/Roary/
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:589cbea1a5e4baf881c92bfc5351a687fb59f27b3d90888ae72f1763795e6ea0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MrTomRod/scoary-2
  tools:
  - OrthoFinder
  - Roary
  - Scoary2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# scoary2-python-api-usage

## Summary

Drive Scoary2 programmatically through its Python library API to run automatic binarization of continuous phenotypes and mGWAS association testing on pan-genome presence/absence matrices, enabling scripted benchmarking and reproducible pipelines beyond the command-line and app interfaces.

## When to use

Use when you need to call Scoary2 as a Python library (github.com/MrTomRod/scoary-2) to simulate, batch, or script pan-genome association runs — e.g. planting causal genes in simulated presence/absence matrices and extracting ranked p-values for numeric traits.

## When NOT to use

- Your traits are already binary (e.g. resistance/susceptibility labels) — automatic binarization of continuous traits via k-means/GMM is unnecessary and may distort the calls
- You need interactive exploration of one finished analysis rather than scripted or simulated runs — the static HTML/JS data exploration app is the intended interface there
- Your input is raw genome assemblies or protein FASTA rather than a pan-genome presence/absence matrix — first run an orthology tool such as OrthoFinder or Roary

## Inputs

- Pan-genome presence/absence matrix (orthogenes generated with OrthoFinder or Roary), one column per gene, one row per isolate
- Numeric/continuous phenotypic trait table keyed by isolate ID (e.g. metabolome measurements of yogurts)
- Simulation parameters when benchmarking: number of isolates (25-200) and causal-gene effect size (0.5-3 sigma)

## Outputs

- Ranked gene-level association results per trait (genes sorted by strength of association, including the p-values used for causal-gene ranking)
- Binarized trait assignments from automatic binarization, with isolates below the 85% posterior-probability cutoff treated as missing/discarded
- Aggregated result tables (e.g. causal-gene rank tables such as fig1_causal_gene_ranks.csv, binary trait tables such as binary_traits.csv)

## How to apply

Import Scoary2 as a Python library (per its 'Usage as Python library' wiki) and pass a pan-genome presence/absence matrix (e.g. from OrthoFinder or Roary) together with a numeric trait table keyed by isolate ID. For continuous traits, invoke Scoary2's automatic binarization, which fits k-means (k=2) and a two-component Gaussian mixture model and discards isolates classified with less than 85% predicted posterior probability. Then run Scoary2's mGWAS association test on each dataset and collect the ranked gene-level p-values from the results object. Rationale: the paper introduced Scoary2 as an 'ultra-fast' re-write of Scoary specifically to overcome Scoary's performance constraints, inability to pre-process numeric traits, and lack of post-GWAS methods, making the library API suitable for high-throughput simulations (25-200 genomes) and repeated runs across effect sizes (0.5-3 sigma).

## Related tools

- **Scoary2** (Core tool invoked via its Python library API to perform automatic binarization of continuous numeric traits and mGWAS association testing on pan-genome presence/absence matrices) — https://github.com/MrTomRod/scoary-2
- **OrthoFinder** (Upstream orthology inference used to generate the orthogene presence/absence matrix that Scoary2 consumes as input) — https://github.com/davidemms/OrthoFinder/
- **Roary** (Alternative upstream pan-genome pipeline whose presence/absence output can be used as Scoary2 input) — https://sanger-pathogens.github.io/Roary/

## Evaluation signals

- Causal-gene rank targets from the benchmark are reproduced: top rank at 3 sigma / 50 isolates and top-four rank at 1.5 sigma / 75 isolates with 90% probability
- Every simulated condition (dataset size x effect size) appears as a row with a rank value in the aggregated output CSV
- For binarized traits, all retained binary assignments have a recorded GMM posterior probability >= 0.85, and uncertain isolates appear as missing rather than forced binary labels
- Row counts of the binary trait table match the number of isolates in the input presence/absence matrix, and each continuous trait column is represented
- Result structure matches Scoary2's documented output: a list of genes sorted by strength of association per trait

## Limitations

- Isolates classified with less than 85% predicted posterior probability by the Gaussian mixture model are ignored from further analysis, reducing effective sample size for weakly separated traits
- The tool requires a pre-computed pan-genome presence/absence matrix (e.g. from OrthoFinder or Roary); it does not process raw assemblies
- No discussion-section limitations were extractable from the provided document text

## Evidence

- [abstract] we introduce Scoary2, an ultra-fast microbial genome-wide association studies (mGWAS) software: "we introduce Scoary2, an ultra-fast microbial genome-wide association studies (mGWAS) software"
- [intro] Three limitations prevent Scoary from high-throughput analyses: "Three limitations prevent Scoary from such high-throughput analyses: performance constraints, the inability to pre-process numeric traits, and a lack of post-GWAS methods"
- [readme] Scoary2 associates orthogenes to traits: "Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder] or [Roary][roary] to traits."
- [readme] Usage as Python library wiki page exists: "- [Usage as Python library](https://github.com/MrTomRod/scoary-2/wiki/Usage-as-Python-library)"
