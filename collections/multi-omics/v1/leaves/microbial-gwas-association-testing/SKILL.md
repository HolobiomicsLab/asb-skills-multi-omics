---
name: microbial-gwas-association-testing
description: Use when a pan-genome gene presence/absence matrix (orthogroups from
  OrthoFinder or Roary) and a numeric phenotype table (e.g. 3889 metabolomics traits
  for 182 yogurt isolates) are available and the goal is to associate orthogenes with
  traits.
when_to_use_negative:
- When no pan-genome orthogroup presence/absence matrix exists and only raw assemblies
  are available — orthogroups must first be inferred with OrthoFinder or Roary before
  association testing
- When traits are not numeric/pre-processable phenotypic measurements; the original
  Scoary cannot pre-process numeric traits, and Scoary2 targets multi-omics phenotype
  tables rather than categorical-only designs
- When a human GWAS or QTL method is required — established hGWAS/QTL methods cannot
  be directly applied to microbes due to differences between human and microbial genomes
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0622
tools:
- name: Scoary2
  role: 'Primary mGWAS engine: associates orthogenes with traits, applies fdr_bh:0.1
    or trait-wise bonferroni:0.999 correction, and reports genes ranked by association
    strength per trait'
  repo: https://github.com/MrTomRod/scoary-2
- name: Scoary
  role: Original command-line tool used as the runtime baseline (22 min full / 16
    min reduced yogurt dataset) against which Scoary2's speed is validated
- name: OrthoFinder
  role: Generates the orthogroups/hierarchical orthogroup presence-absence matrix
    that serves as Scoary2's gene input
  repo: https://github.com/davidemms/OrthoFinder/
- name: Roary
  role: Alternative pan-genome tool cited by the Scoary2 README for generating orthogenes
    prior to association testing
  repo: https://sanger-pathogens.github.io/Roary/
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:30be12831879778ee8b09cc29d94dbe66aa5910588f93411cfb224093c385ee5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MrTomRod/scoary-2
  tools:
  - OrthoFinder
  - Roary
  - Scoary
  - Scoary2
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# microbial-gwas-association-testing

## Summary

Perform microbial genome-wide association studies (mGWAS) by correlating pan-genome orthogene presence/absence matrices against large phenotypic trait tables (e.g. yogurt metabolomics) using Scoary2, with trait-wise multiple-testing correction, to identify candidate genes underlying bacterial phenotypes such as carnitine metabolism.

## When to use

Use when a pan-genome gene presence/absence matrix (orthogroups from OrthoFinder or Roary) and a numeric phenotype table (e.g. 3889 metabolomics traits for 182 yogurt isolates) are available and the goal is to associate orthogenes with traits.

## When NOT to use

- When no pan-genome orthogroup presence/absence matrix exists and only raw assemblies are available — orthogroups must first be inferred with OrthoFinder or Roary before association testing
- When traits are not numeric/pre-processable phenotypic measurements; the original Scoary cannot pre-process numeric traits, and Scoary2 targets multi-omics phenotype tables rather than categorical-only designs
- When a human GWAS or QTL method is required — established hGWAS/QTL methods cannot be directly applied to microbes due to differences between human and microbial genomes

## Inputs

- Pan-genome gene presence/absence matrix (hierarchical orthogroups, e.g. from OrthoFinder or Roary; 10,358 orthogroups x 182 isolates full set, or 1,466 orthogroups x 44 P. freudenreichii isolates reduced set)
- Numeric phenotypic trait table (yogurt metabolomics: 3,889 traits full set; 1466-orthogroup reduced subset)
- Scoary2 source/deposited dataset from github.com/MrTomRod/scoary-2

## Outputs

- Per-trait gene association results ranked by strength of association (significant orthogenes per metabolite trait)
- FDR-BH 0.1-filtered set of significant metabolite associations (20 metabolites in the reduced yogurt dataset)
- Trait-wise Bonferroni 0.999 result set covering 707 traits with candidate genes (MFS transporter, α/β-hydrolase)
- Benchmark runtime comparison table (e.g. benchmark_runtimes.csv)

## How to apply

Build or load a pan-genome gene presence/absence matrix of hierarchical orthogroups (e.g. 10,358 orthogroups for the full 182-isolate Agroscope yogurt dataset, or 1466 orthogroups for the reduced 44-isolate Propionibacterium freudenreichii subset) alongside the metabolomics trait table, and run Scoary2 with the desired multiple-testing settings. For a stringent gene-vs-trait screen use multiple_testing = fdr_bh:0.1, which recovers 20 significant metabolite associations in the reduced yogurt dataset; for a broad trait-wise screen set trait-wise-correction = True with multiple_testing = bonferroni:0.999, which yields a result set covering 707 traits. Scoary2 reports genes sorted by strength of association per trait, so rank the trait-corrected output to surface top candidates — in the yogurt carnitine analysis these are an MFS transporter and an α/β-hydrolase fold domain-containing protein. Validate by confirming the expected counts of significant metabolites/traits and by comparing runtimes against the original Scoary benchmark (Scoary2 ~23 s on the full dataset vs ~22 min for Scoary; ~43 s vs ~16 min on the reduced dataset). Results can be explored interactively with Scoary2's static HTML/JS app.

## Related tools

- **Scoary2** (Primary mGWAS engine: associates orthogenes with traits, applies fdr_bh:0.1 or trait-wise bonferroni:0.999 correction, and reports genes ranked by association strength per trait) — https://github.com/MrTomRod/scoary-2
- **Scoary** (Original command-line tool used as the runtime baseline (22 min full / 16 min reduced yogurt dataset) against which Scoary2's speed is validated)
- **OrthoFinder** (Generates the orthogroups/hierarchical orthogroup presence-absence matrix that serves as Scoary2's gene input) — https://github.com/davidemms/OrthoFinder/
- **Roary** (Alternative pan-genome tool cited by the Scoary2 README for generating orthogenes prior to association testing) — https://sanger-pathogens.github.io/Roary/

## Evaluation signals

- FDR-BH 0.1 run on the reduced yogurt dataset (44 P. freudenreichii isolates) yields exactly 20 significant metabolite associations
- Trait-wise-correction run (bonferroni:0.999) yields a result set covering 707 traits
- Ranked trait-corrected results place the MFS transporter and the α/β-hydrolase fold domain-containing protein as the two top candidate genes for carnitine-related metabolites
- Runtime benchmark reproduces ~23 s (Scoary2) vs ~22 min (Scoary) on the full 3,889-trait dataset and ~43 s (Scoary2) vs ~16 min (Scoary) on the reduced dataset
- Output is a gene list sorted by strength of association per trait and is loadable in Scoary2's static HTML/JS exploration app

## Limitations

- The original Scoary has performance constraints, inability to pre-process numeric traits, and a lack of post-GWAS methods, preventing high-throughput analyses of large trait sets
- 40–60% of bacterial genes are undescribed, so associations identify candidate orthogenes without known function rather than annotated mechanisms
- Discussion-section content (e.g. limitations of Scoary2 itself) was not available in the extracted document, so tool-specific caveats are incompletely documented

## Evidence

- [abstract] Scoary2 intro: "we introduce Scoary2, an ultra-fast microbial genome-wide association studies (mGWAS) software"
- [abstract] carnitine finding: "we explore the metabolome of yogurts, each produced with a different Propionibacterium reichii strain and discover two genes affecting carnitine metabolism"
- [readme] README usage description: "Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder] or [Roary][roary] to traits. It reports a list of genes sorted by strength of association per trait."
- [intro] Scoary limitations motivating Scoary2: "Three limitations prevent Scoary from such high-throughput analyses: performance constraints, the inability to pre-process numeric traits, and a lack of post-GWAS methods optimized"
- [abstract] large-dataset capability: "With its data exploration app and improved performance, Scoary2 is the first tool to enable the study of large phenotypic datasets using mGWAS"
