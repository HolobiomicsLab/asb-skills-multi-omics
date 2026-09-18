---
name: candidate-gene-identification-from-gwas
description: Use when you have a microbial pan-genome orthogene matrix and numeric
  or binary trait data (e.g. metabolomics of strains) and need to find genes statistically
  associated with trait differences via mGWAS.
when_to_use_negative:
- Input is not a pan-genome orthogene matrix (e.g. already-annotated gene functions)
  — Scoary2 associates annotation-independent orthogenes with traits, not pre-called
  gene functions
- The dataset is very small or the traits are not suitable for presence/absence association
  testing (e.g. no measurable strain-to-strain trait variation)
- You need hGWAS/QTL methods designed for recombining human-style genomes rather than
  microbial pan-genome presence/absence data
edam_topics:
- http://edamontology.org/topic_0625
tools:
- name: Scoary2
  role: 'Performs the mGWAS: associates orthogenes with traits, applies multiple-testing
    correction (fdr_bh, trait-wise bonferroni), and outputs per-trait ranked gene
    lists'
  repo: https://github.com/MrTomRod/scoary-2
- name: OrthoFinder
  role: Generates orthogroups/orthogenes used as the pan-genome gene presence/absence
    input to Scoary2
  repo: https://github.com/davidemms/OrthoFinder/
- name: Roary
  role: Alternative pan-genome tool to build the gene presence/absence matrix consumed
    by Scoary2
  repo: https://sanger-pathogens.github.io/Roary/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:45d211a9dbd999d6303d52505a4602f0c35ed7279852e44697536c53a3f8cc8c
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
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# candidate-gene-identification-from-gwas

## Summary

Identify candidate genes underlying phenotypic variation by associating a pan-genome gene presence/absence matrix with multi-omics trait data (e.g. yogurt metabolomics) using Scoary2, then ranking the associated orthogenes per trait.

## When to use

Use when you have a microbial pan-genome orthogene matrix and numeric or binary trait data (e.g. metabolomics of strains) and need to find genes statistically associated with trait differences via mGWAS.

## When NOT to use

- Input is not a pan-genome orthogene matrix (e.g. already-annotated gene functions) — Scoary2 associates annotation-independent orthogenes with traits, not pre-called gene functions
- The dataset is very small or the traits are not suitable for presence/absence association testing (e.g. no measurable strain-to-strain trait variation)
- You need hGWAS/QTL methods designed for recombining human-style genomes rather than microbial pan-genome presence/absence data

## Inputs

- pan-genome gene presence/absence matrix (orthogenes from OrthoFinder or Roary) for the isolate set
- phenotypic trait table (e.g. yogurt metabolomics of 44 Propionibacterium freudenreichii isolates)
- multiple-testing correction parameters (e.g. fdr_bh:0.1; trait-wise bonferroni:0.999)

## Outputs

- per-trait association results: genes sorted by strength of association for each trait
- list of significant metabolite associations under fdr_bh:0.1 (20 metabolites in the yogurt dataset)
- trait-corrected result set of 707 traits under trait-wise bonferroni:0.999
- ranked candidate genes (MFS transporter and α/β-hydrolase affecting carnitine metabolism)

## How to apply

Load the pan-genome gene presence/absence matrix (orthogenes from OrthoFinder or Roary) and the trait table — e.g. the yogurt metabolomics table for 44 Propionibacterium freudenreichii isolates from the Agroscope strain collection — into Scoary2. Run the association with a multiple-testing correction such as fdr_bh:0.1 to recover significant metabolite-gene associations (20 metabolites in the yogurt analysis). Re-run with trait-wise correction enabled and bonferroni:0.999 to obtain a broad trait-corrected result set (707 traits). Scoary2 reports a list of genes sorted by strength of association per trait; rank these to pick top candidates — in the yogurt case an MFS transporter and an α/β-hydrolase fold domain-containing protein affecting carnitine metabolism. Results can be inspected with the bundled static HTML/JS data exploration app.

## Related tools

- **Scoary2** (Performs the mGWAS: associates orthogenes with traits, applies multiple-testing correction (fdr_bh, trait-wise bonferroni), and outputs per-trait ranked gene lists) — https://github.com/MrTomRod/scoary-2
- **OrthoFinder** (Generates orthogroups/orthogenes used as the pan-genome gene presence/absence input to Scoary2) — https://github.com/davidemms/OrthoFinder/
- **Roary** (Alternative pan-genome tool to build the gene presence/absence matrix consumed by Scoary2) — https://sanger-pathogens.github.io/Roary/

## Evaluation signals

- The fdr_bh:0.1 run recovers exactly 20 significant metabolite associations on the reduced 44-isolate yogurt dataset
- The trait-wise corrected run (bonferroni:0.999, trait-wise-correction = True) yields 707 traits
- Ranking the trait-corrected associations places an MFS transporter and an α/β-hydrolase fold domain-containing protein as the top two candidate genes for carnitine-related metabolites
- Output conforms to Scoary2's per-trait schema: a list of genes sorted by strength of association for each trait
- Results are reproducible with the released Scoary2 version (Zenodo 10.5281/zenodo.10352170)

## Limitations

- Scoary's original limitations motivated the rewrite: performance constraints, inability to pre-process numeric traits, and lack of post-GWAS methods — older Scoary should not be used for large trait sets
- Functions of many bacterial genes (~40–60%) are unknown, so candidate genes from the association are annotation-independent hits requiring downstream functional interpretation
- The candidate-gene calls are correlations from a 44-isolate dataset; carnitine-metabolism assignments (MFS transporter, α/β-hydrolase) rest on the yogurt metabolome analysis rather than direct functional validation

## Evidence

- [abstract] two genes affecting carnitine metabolism were discovered in yogurt produced with different Propionibacterium reichii strains: "we explore the metabolome of yogurts, each produced with a different Propionibacterium reichii strain and discover two genes affecting carnitine metabolism"
- [readme] Scoary2 associates orthogenes with traits and reports genes sorted by association strength per trait: "Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder]
or [Roary][roary] to traits. It reports a list of genes sorted by strength of
association per trait."
- [abstract] Scoary is an mGWAS tool enabling study of large phenotypic datasets: "we introduce Scoary2, an ultra-fast microbial genome-wide association studies (mGWAS) software"
- [intro] Existing mGWAS tools could not handle large phenotypic datasets: "Three limitations prevent Scoary from such high-throughput analyses: performance constraints, the inability to pre-process numeric traits, and a lack of post-GWAS methods"
