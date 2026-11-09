---
name: pan-genome-phenotype-association
description: Use when a study must link measured phenotypes (e.g. omics traits such
  as yogurt metabolites) to orthologous gene families (orthogroups from OrthoFinder
  or Roary) across dozens to hundreds of bacterial isolates using mGWAS.
when_to_use_negative:
- When the analysis targets human or other eukaryotic GWAS — established hGWAS or
  QTL methods cannot be directly applied to microbes and this mGWAS-specific toolchain
  is inappropriate for them
- When traits are already linked to genes and only visualization or simple statistics
  are needed — Scoary2's association machinery is unnecessary overhead
- When no orthogroup matrix exists and pan-genome inference (OrthoFinder/Roary) cannot
  be run on the isolate genomes
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Scoary2
  role: Primary mGWAS engine that associates orthogenes with traits and produces ranked
    association results plus the exploration app
  repo: https://github.com/MrTomRod/scoary-2
- name: Scoary
  role: Original command-line association tool used as the performance baseline against
    Scoary2 on equivalent settings
- name: OrthoFinder
  role: Infers the hierarchical orthogroups (pan-genome matrix) that serve as Scoary2's
    gene input
  repo: https://github.com/davidemms/OrthoFinder/
- name: Roary
  role: Alternative pan-genome pipeline that can generate the orthogroup input for
    Scoary2
  repo: https://sanger-pathogens.github.io/Roary/
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:44ace9806b8fbeba8a31b64f264073e8e60ad28553ec763dd660f8fc45b839f1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MrTomRod/scoary-2
  tools:
  - OrthoFinder
  - Roary
  - Scoary
  - Scoary2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# pan-genome-phenotype-association

## Summary

Runs microbial GWAS (mGWAS) with Scoary2 to associate hierarchical orthogroups from a microbial pan-genome with phenotypic traits, scaling to thousands of traits (e.g. yogurt metabolome features) where the original Scoary is too slow. Scoary2 reports genes ranked by association strength per trait and supports interactive exploration via a static HTML/JS app.

## When to use

Use when a study must link measured phenotypes (e.g. omics traits such as yogurt metabolites) to orthologous gene families (orthogroups from OrthoFinder or Roary) across dozens to hundreds of bacterial isolates using mGWAS.

## When NOT to use

- When the analysis targets human or other eukaryotic GWAS — established hGWAS or QTL methods cannot be directly applied to microbes and this mGWAS-specific toolchain is inappropriate for them
- When traits are already linked to genes and only visualization or simple statistics are needed — Scoary2's association machinery is unnecessary overhead
- When no orthogroup matrix exists and pan-genome inference (OrthoFinder/Roary) cannot be run on the isolate genomes

## Inputs

- Pan-genome matrix of hierarchical orthogroups (e.g. OrthoFinder output; 10,358 hierarchical orthogroups x 182 isolates in the yogurt study)
- Phenotype/traits table (binary or numeric omics traits, e.g. 3,889 yogurt metabolome traits) with sample identifiers matching isolates
- Optionally a reduced dataset subset (e.g. 44 isolates, 1,466 orthogroups) for benchmarking or filtering

## Outputs

- Per-trait list of genes/orthogroups sorted by strength of association with p-values
- Benchmark runtime table comparing Scoary2 and Scoary wall-clock times (e.g. benchmark_runtimes.csv)
- Interactive HTML/JS exploration app linking association results to the trait and genome data

## How to apply

Generate a pan-genome matrix of hierarchical orthogroups using OrthoFinder (or Roary) over the isolate genome set — in the Scoary2 paper this yielded 10,358 hierarchical orthogroups for 182 yogurt isolates from the Agroscope strain collection. Prepare a traits table (binary or pre-processed numeric phenotypes, e.g. 3,889 yogurt traits) with rows matching the isolates. Run Scoary2 (github.com/MrTomRod/scoary-2) on the orthogroup and trait tables using the paper's parameters; for benchmarking, Scoary2 completed a 100-trait subset on the full dataset in ~23 s versus ~22 min for Scoary, and the reduced dataset (44 isolates, 1,466 orthogroups) in ~43 s versus 16 min. Inspect ranked association output per trait, applying Scoary2's corrected p-value framework, and use the bundled HTML/JS app for interactive data exploration. Validate hits biologically, as done for the two carnitine-metabolism genes found across Propionibacterium reichii yogurt strains.

## Related tools

- **Scoary2** (Primary mGWAS engine that associates orthogenes with traits and produces ranked association results plus the exploration app) — https://github.com/MrTomRod/scoary-2
- **Scoary** (Original command-line association tool used as the performance baseline against Scoary2 on equivalent settings)
- **OrthoFinder** (Infers the hierarchical orthogroups (pan-genome matrix) that serve as Scoary2's gene input) — https://github.com/davidemms/OrthoFinder/
- **Roary** (Alternative pan-genome pipeline that can generate the orthogroup input for Scoary2) — https://sanger-pathogens.github.io/Roary/

## Evaluation signals

- Scoary2 completes the full-dataset association (3,889 traits, 182 isolates, 10,358 hierarchical orthogroups) in approximately 23 s, versus approximately 22 min for Scoary on the 100-trait subset
- Reduced-dataset runs finish in approximately 43 s (Scoary2) and approximately 16 min (Scoary), matching the paper's reported performance figures
- Output contains genes ranked by association strength per trait with Scoary2's p-value annotations (see wiki 'Understanding the p values')
- Biological sanity check: known metabolic associations (e.g. the two carnitine-metabolism genes across Propionibacterium reichii yogurt strains) are recovered among top hits
- Trait and isolate identifiers in the traits table align one-to-one with the orthogroup matrix columns

## Limitations

- The original Scoary cannot handle large numbers of microbes or traits due to performance constraints, inability to pre-process numeric traits, and lack of post-GWAS methods — use Scoary2 for such scales
- 40-60% of bacterial genes are undescribed, so associations are annotation-independent and require follow-up interpretation
- The provided article text lacks a Discussion section; additional limitations documented there could not be extracted from this context

## Evidence

- [abstract] we introduce Scoary2, an ultra-fast microbial genome-wide association studies (mGWAS) software: "we introduce Scoary2, an ultra-fast microbial genome-wide association studies (mGWAS) software"
- [abstract] Two genes affecting carnitine metabolism discovered in yogurt produced with different P. reichii strains: "we explore the metabolome of yogurts, each produced with a different Propionibacterium reichii strain and discover two genes affecting carnitine metabolism"
- [intro] Scoary's three limitations preventing high-throughput analyses: "Three limitations prevent Scoary from such high-throughput analyses: performance constraints, the inability to pre-process numeric traits, and a lack of post-GWAS"
- [readme] Scoary2 README description of tool purpose: "Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder] or [Roary][roary] to traits. It reports a list of genes sorted by strength of"
