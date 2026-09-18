---
name: causal-gene-rank-evaluation
description: Use when validating a microbial GWAS pipeline's power on continuously
  measured numeric traits, by simulating datasets with a known planted causal gene
  and evaluating how its rank depends on isolate count and effect size.
when_to_use_negative:
- The phenotype is already binarized (case/control) — automatic binarization of a
  continuous trait is the capability being benchmarked and is unnecessary
- Analyzing real observational data with no planted/known causal gene — rank evaluation
  of a true causal gene is undefined
- The goal is ortholog clustering itself (e.g. building the pan-genome with OrthoFinder
  or Roary) rather than evaluating association power downstream
edam_operation: http://edamontology.org/operation_0451
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Scoary2
  role: Performs automatic binarization of the continuous numeric trait and the mGWAS
    association test; invoked via its Python library API on simulated pan-genome matrices
  repo: github.com/MrTomRod/scoary-2
- name: OrthoFinder
  role: Example upstream tool for generating the orthogene presence/absence pan-genome
    matrices that Scoary2 associates with traits
  repo: github.com/davidemms/OrthoFinder
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:1bbac58d9ffa379734736b4f1193aa2bfe8a085279d9ce0d5310eb3cb5193695
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MrTomRod/scoary-2
  tools:
  - OrthoFinder
  - Scoary2
  edam_operation: http://edamontology.org/operation_0451
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 3
---

# causal-gene-rank-evaluation

## Summary

Benchmark Scoary2's automatic binarization and mGWAS association testing by planting simulated causal genes (0.5–3 sigma effect sizes) into pan-genome presence/absence matrices with paired numeric phenotypes, then measuring the rank of the planted causal gene among gene-level association p-values across dataset sizes (25–200 isolates).

## When to use

Use when validating a microbial GWAS pipeline's power on continuously measured numeric traits, by simulating datasets with a known planted causal gene and evaluating how its rank depends on isolate count and effect size.

## When NOT to use

- The phenotype is already binarized (case/control) — automatic binarization of a continuous trait is the capability being benchmarked and is unnecessary
- Analyzing real observational data with no planted/known causal gene — rank evaluation of a true causal gene is undefined
- The goal is ortholog clustering itself (e.g. building the pan-genome with OrthoFinder or Roary) rather than evaluating association power downstream

## Inputs

- Simulated pan-genome presence/absence matrix (gene x isolate) with a planted causal gene
- Paired continuously measured numeric phenotype per isolate (with effect sizes of 0.5–3 sigma)
- Scoary2 accessible via its Python library API

## Outputs

- Ranked gene-level association p-values from Scoary2's mGWAS test per simulated dataset
- Rank of the planted causal gene for each dataset-size x effect-size condition
- Aggregated causal-gene rank table written to fig1_causal_gene_ranks.csv

## How to apply

Generate simulated pan-genome presence/absence matrices paired with numeric phenotypes for dataset sizes from 25 to 200 isolates, planting a causal gene with effect sizes between 0.5 and 3 sigma. Call Scoary2 through its Python library API, first applying its automatic binarization to discretize the continuous trait, then running the mGWAS association test to obtain gene-level p-values. Extract the rank of the planted causal gene from the ranked p-value output and aggregate ranks across dataset sizes and effect sizes. The reproduction criteria are that the causal gene achieves top rank at 3 sigma with 50 isolates and a top-four rank at 1.5 sigma with 75 isolates with 90% probability. Write the aggregated rank table to fig1_causal_gene_ranks.csv.

## Related tools

- **Scoary2** (Performs automatic binarization of the continuous numeric trait and the mGWAS association test; invoked via its Python library API on simulated pan-genome matrices) — github.com/MrTomRod/scoary-2
- **OrthoFinder** (Example upstream tool for generating the orthogene presence/absence pan-genome matrices that Scoary2 associates with traits) — github.com/davidemms/OrthoFinder

## Evaluation signals

- The planted causal gene attains top rank at 3 sigma effect size with 50 isolates
- The planted causal gene falls within the top four ranks with 90% probability at 1.5 sigma effect size with 75 isolates
- The output CSV (fig1_causal_gene_ranks.csv) contains rank values for every simulated dataset-size x effect-size condition
- Ranks were extracted from Scoary2's ranked p-value results after automatic binarization, not from raw continuous-trait associations
- Aggregated rank table spans the full grid of dataset sizes (25–200 isolates) and effect sizes (0.5–3 sigma)

## Limitations

- Performance limits of the predecessor tool Scoary: performance constraints, inability to pre-process numeric traits, and lack of post-GWAS methods — automatic binarization in Scoary2 addresses only the numeric-trait limitation
- Results are specific to simulated pan-genome/phenotype settings; power at other isolate counts or effect sizes outside the tested grid (25–200 isolates, 0.5–3 sigma) is not established

## Evidence

- [abstract] we introduce Scoary2, an ultra-fast microbial genome-wide association studies (mGWAS) software: "we introduce Scoary2, an ultra-fast microbial genome-wide association studies (mGWAS) software"
- [intro] Scoary cannot handle large numbers of microbes or large phenotypic datasets: "Three limitations prevent Scoary from such high-throughput analyses: performance constraints, the inability to pre-process numeric traits, and a lack of post-GWAS methods optimized for the"
- [readme] Scoary2 associates orthogenes with traits: "Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder]
or [Roary][roary] to traits."
