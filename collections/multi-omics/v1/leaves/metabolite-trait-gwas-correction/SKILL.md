---
name: metabolite-trait-gwas-correction
description: Use when running mGWAS of a pan-genome orthogene matrix against a large
  numeric phenotypic dataset (e.g. metabolomics traits per strain) and you must control
  false positives with trait-aware correction before nominating candidate genes.
when_to_use_negative:
- Traits are not numeric or not pre-processed per isolate (e.g. categorical phenotypes),
  since Scoary2's corrected workflow targets large numeric multi-omics trait tables
- The pan-genome has not yet been built — orthogenes must first be generated with
  OrthoFinder or Roary before association
- The dataset is small and per-trait correction is unnecessary — trait-wise Bonferroni
  at 0.999 is deliberately relaxed for high-throughput trait screening, not confirmatory
  single-trait testing
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3173
tools:
- name: Scoary2
  role: Performs the gene–trait association and applies the fdr_bh:0.1 and trait-wise
    bonferroni:0.999 multiple-testing corrections on the reduced yogurt dataset
  repo: https://github.com/MrTomRod/scoary-2
- name: OrthoFinder
  role: Generates the orthogene presence/absence pan-genome matrix used as the Scoary2
    genotype input
  repo: https://github.com/davidemms/OrthoFinder/
- name: Roary
  role: Alternative pan-genome tool for generating the orthogene matrix fed to Scoary2
  repo: https://sanger-pathogens.github.io/Roary/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:5a49aa0db6621a4f5703d99534ad3b221c3a9ff44c24a7aee7ac98416d379728
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MrTomRod/scoary-2
  tools:
  - OrthoFinder
  - Roary
  - Scoary2
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# metabolite-trait-gwas-correction

## Summary

Apply multiple-testing correction (FDR-BH and trait-wise Bonferroni) to a Scoary2 microbial GWAS that links a pan-genome gene presence/absence matrix to numeric metabolomics traits, recovering a controlled set of significant metabolite associations and top candidate genes (e.g. carnitine-related MFS transporter and α/β-hydrolase in the yogurt P. freudenreichii dataset).

## When to use

Use when running mGWAS of a pan-genome orthogene matrix against a large numeric phenotypic dataset (e.g. metabolomics traits per strain) and you must control false positives with trait-aware correction before nominating candidate genes.

## When NOT to use

- Traits are not numeric or not pre-processed per isolate (e.g. categorical phenotypes), since Scoary2's corrected workflow targets large numeric multi-omics trait tables
- The pan-genome has not yet been built — orthogenes must first be generated with OrthoFinder or Roary before association
- The dataset is small and per-trait correction is unnecessary — trait-wise Bonferroni at 0.999 is deliberately relaxed for high-throughput trait screening, not confirmatory single-trait testing

## Inputs

- Pan-genome gene presence/absence matrix (orthogene × isolate binary matrix from OrthoFinder or Roary)
- Numeric metabolomics trait table (one row per isolate, e.g. 44 P. freudenreichii yogurt isolates, metabolites as traits)
- Correction parameters: multiple_testing = fdr_bh:0.1 (discovery run) and bonferroni:0.999 with trait-wise correction (screening run)

## Outputs

- Per-trait list of genes sorted by strength of association
- Filtered set of significant metabolite associations (20 metabolites at fdr_bh:0.1)
- Trait-wise Bonferroni-corrected result set (707 traits) with top-ranked candidate genes (MFS transporter, α/β-hydrolase fold domain-containing protein)

## How to apply

Load the pan-genome gene presence/absence matrix and the numeric metabolomics trait table (e.g. yogurt metabolome of 44 P. freudenreichii isolates) into Scoary2, which associates orthogenes (from OrthoFinder or Roary) to traits and reports genes sorted by strength of association per trait. First run with multiple_testing = fdr_bh:0.1 to obtain the filtered set of significant metabolite associations (expected 20 metabolites in the yogurt case). Then re-run with trait-wise-correction = True and multiple_testing = bonferroni:0.999 to obtain the broad trait-corrected result set (707 traits), because per-trait correction is needed when tens of thousands of traits are tested. Finally rank the associated genes within the trait-corrected results to nominate top candidates affecting the trait of interest (carnitine-related metabolites), and confirm both runs reproduce the reported counts and rankings.

## Related tools

- **Scoary2** (Performs the gene–trait association and applies the fdr_bh:0.1 and trait-wise bonferroni:0.999 multiple-testing corrections on the reduced yogurt dataset) — https://github.com/MrTomRod/scoary-2
- **OrthoFinder** (Generates the orthogene presence/absence pan-genome matrix used as the Scoary2 genotype input) — https://github.com/davidemms/OrthoFinder/
- **Roary** (Alternative pan-genome tool for generating the orthogene matrix fed to Scoary2) — https://sanger-pathogens.github.io/Roary/

## Examples

```
scoary2 --genes pangenome.csv --traits yogurt_metabolites.tsv --multiple_testing fdr_bh:0.1 ; scoary2 --genes pangenome.csv --traits yogurt_metabolites.tsv --multiple_testing bonferroni:0.999 --traitwise_correction
```

## Evaluation signals

- First Scoary2 run on the reduced yogurt dataset (44 isolates) yields exactly 20 metabolites passing fdr_bh:0.1
- Second run with trait-wise-correction = True and bonferroni:0.999 yields 707 traits in the corrected result set
- Ranking of the trait-corrected results places an MFS transporter and an α/β-hydrolase fold domain-containing protein as the two top candidate genes for the carnitine-related metabolites
- Output file contains genes sorted by strength of association per trait as documented in the Scoary2 wiki
- Associations are annotation-independent (gene IDs are orthogroups), so no functional annotation is required to reproduce the ranking

## Limitations

- Trait-wise Bonferroni 0.999 is deliberately relaxed, yielding a broad (707-trait) result set intended for ranking/candidate generation, not strict confirmatory significance
- Many bacterial genes (~40–60%) are undescribed, so associations are annotation-independent and candidate genes need follow-up functional validation
- Scoary2 depends on the quality of upstream pan-genome inference (OrthoFinder or Roary); divergent orthogroup definitions will change the genotype matrix

## Evidence

- [abstract] two genes affecting carnitine metabolism were discovered: "we explore the metabolome of yogurts, each produced with a different Propionibacterium reichii strain and discover two genes affecting carnitine metabolism"
- [readme] Scoary2 associates orthogenes to traits: "Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder]
or [Roary][roary] to traits. It reports a list of genes sorted by strength of
association per trait."
- [abstract] Scoary2 is the first tool for large phenotypic datasets: "With its data exploration app and improved performance, Scoary2 is the first tool to enable the study of large phenotypic datasets using mGWAS"
