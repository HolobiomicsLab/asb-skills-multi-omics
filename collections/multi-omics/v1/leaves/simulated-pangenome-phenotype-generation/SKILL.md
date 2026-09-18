---
name: simulated-pangenome-phenotype-generation
description: Use when benchmarking whether a gene-trait association tool recovers
  a simulated causal gene from continuous numeric phenotypes, varying isolate count
  (25–200) and effect size (σ), with the causal gene's rank as the metric.
when_to_use_negative:
- The input is real observational pan-genome/phenotype data rather than a simulation,
  so no ground-truth causal gene rank is available to validate against
- The trait is already binary/presence-absence, making Scoary2's automatic binarization
  of numeric traits unnecessary — run association directly instead
- The goal is sample-level or strain-level association rather than linking orthogenes
  to traits, since this skill is built around the orthogene-by-isolate matrix format
  Scoary2 consumes
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Scoary2
  role: Used via its Python library API to run automatic binarization of the simulated
    numeric trait and the mGWAS association test that yields ranked gene-level p-values
  repo: github.com/MrTomRod/scoary-2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:f8962f174718d40e545f40106cbc94a095b64bd8bc506c2350a93763d91f0201
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MrTomRod/scoary-2
  tools:
  - Scoary2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# simulated-pangenome-phenotype-generation

## Summary

Generates simulated microbial pan-genome presence/absence matrices paired with continuously measured numeric phenotypes, planting a causal gene at controlled effect sizes (0.5–3 σ), to benchmark how well Scoary2's automatic binarization and mGWAS association test recover the causal gene across dataset sizes.

## When to use

Use when benchmarking whether a gene-trait association tool recovers a simulated causal gene from continuous numeric phenotypes, varying isolate count (25–200) and effect size (σ), with the causal gene's rank as the metric.

## When NOT to use

- The input is real observational pan-genome/phenotype data rather than a simulation, so no ground-truth causal gene rank is available to validate against
- The trait is already binary/presence-absence, making Scoary2's automatic binarization of numeric traits unnecessary — run association directly instead
- The goal is sample-level or strain-level association rather than linking orthogenes to traits, since this skill is built around the orthogene-by-isolate matrix format Scoary2 consumes

## Inputs

- Simulated pan-genome presence/absence matrix (orthogene rows x isolate columns) for 25–200 genomes
- Simulated continuous (numeric) phenotype values, one per isolate
- Planted causal gene definition: which orthogene carries the effect and its effect size in sigma (0.5–3 σ)

## Outputs

- Gene-level association p-values ranked per trait from Scoary2's mGWAS test
- Rank of the planted causal gene for each simulated condition (dataset size x effect size)
- Aggregated causal-gene rank table written to fig1_causal_gene_ranks.csv

## How to apply

Generate simulated pan-genome presence/absence matrices with paired numeric phenotypes for dataset sizes of 25–200 genomes, planting causal genes with effect sizes of 0.5–3 σ, using Scoary2 via its Python library API. Run Scoary2's automatic binarization on the simulated numeric trait to discretize it for association testing — this pre-processing of numeric traits is the capability the original Scoary lacked. Execute Scoary2's mGWAS association test on each simulated dataset to obtain gene-level association p-values, then extract the rank of the planted causal gene from each run's ranked results. Aggregate ranks across dataset sizes and effect sizes and verify the reproduction criteria: top rank at 3 σ / 50 isolates and top-four rank at 1.5 σ / 75 isolates, each with 90% probability. Write the aggregated causal-gene rank table to fig1_causal_gene_ranks.csv.

## Related tools

- **Scoary2** (Used via its Python library API to run automatic binarization of the simulated numeric trait and the mGWAS association test that yields ranked gene-level p-values) — github.com/MrTomRod/scoary-2

## Evaluation signals

- The planted causal gene reaches top rank at 3 σ effect size with 50 isolates in 90% of simulation replicates
- The planted causal gene reaches top-four rank at 1.5 σ effect size with 75 isolates in 90% of replicates
- fig1_causal_gene_ranks.csv contains a rank value for every simulated condition (every dataset size x effect-size combination)
- Each Scoary2 run produces a ranked gene-level p-value list in which the planted causal gene is present and its rank is extractable

## Limitations

- The 90%-probability rank targets (top rank at 3 σ / 50 isolates; top-four at 1.5 σ / 75 isolates) are specific to Scoary2's automatic binarization and may not transfer to other association tools
- Simulation-based results validate recovery of planted causal genes under the tested conditions (25–200 genomes, 0.5–3 σ) and do not by themselves establish performance on real multi-omics datasets
- The provided article text lacks a discussion section, so broader caveats about the simulation methodology could not be extracted

## Evidence

- [abstract] Scoary2 introduces ultra-fast mGWAS: "we introduce Scoary2, an ultra-fast microbial genome-wide association studies (mGWAS) software"
- [intro] Scoary cannot pre-process numeric traits: "Three limitations prevent Scoary from such high-throughput analyses: performance constraints, the inability to pre-process numeric traits"
- [readme] Scoary2 reports genes sorted by association per trait: "Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder] or [Roary][roary] to traits. It reports a list of genes sorted by strength of association per trait."
