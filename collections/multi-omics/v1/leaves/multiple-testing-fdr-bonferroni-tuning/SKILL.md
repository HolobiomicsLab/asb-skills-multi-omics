---
name: multiple-testing-fdr-bonferroni-tuning
description: Use when running Scoary2 associations between a pan-genome gene presence/absence
  matrix and numeric multi-omics traits and you must choose between strict FDR-BH
  filtering and a relaxed trait-wise Bonferroni correction to recover candidate genes.
when_to_use_negative:
- When traits are binary rather than the large numeric multi-omics trait tables Scoary2
  was built for, since the article motivates Scoary2 by prior tools' performance constraints
  and inability to pre-process numeric traits at high throughput
- When you need only a single strict significance list with no exploratory candidate-gene
  ranking — the FDR-BH 0.1 run alone suffices and the relaxed bonferroni:0.999 run
  adds hundreds of unfiltered traits
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Scoary2
  role: Runs the mGWAS association of orthogenes with yogurt metabolomics traits,
    applying either fdr_bh:0.1 or trait-wise bonferroni:0.999 multiple-testing correction
  repo: https://github.com/MrTomRod/scoary-2
- name: OrthoFinder
  role: Generates the orthogroups/orthogenes whose presence/absence matrix serves
    as the genotype input corrected for multiple testing in Scoary2
  repo: https://github.com/davidemms/OrthoFinder/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:98ecb141d1d579343b7348b3219a5b948f6bf42393c6a95a4dca9be9923fd5c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MrTomRod/scoary-2
  tools:
  - OrthoFinder
  - Scoary2
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# multiple-testing-fdr-bonferroni-tuning

## Summary

Calibrates Scoary2 mGWAS significance thresholds on the yogurt metabolomics pan-genome association study, contrasting an FDR-BH 0.1 correction (yielding 20 significant metabolites) with a trait-wise relaxed Bonferroni 0.999 correction (yielding 707 traits) to balance false-discovery control against candidate-gene recall for carnitine metabolism.

## When to use

Use when running Scoary2 associations between a pan-genome gene presence/absence matrix and numeric multi-omics traits and you must choose between strict FDR-BH filtering and a relaxed trait-wise Bonferroni correction to recover candidate genes.

## When NOT to use

- When traits are binary rather than the large numeric multi-omics trait tables Scoary2 was built for, since the article motivates Scoary2 by prior tools' performance constraints and inability to pre-process numeric traits at high throughput
- When you need only a single strict significance list with no exploratory candidate-gene ranking — the FDR-BH 0.1 run alone suffices and the relaxed bonferroni:0.999 run adds hundreds of unfiltered traits

## Inputs

- Pan-genome gene presence/absence matrix for the 44 Propionibacterium freudenreichii isolates (orthogenes from OrthoFinder or Roary)
- Yogurt metabolomics trait table (numeric traits per isolate, including carnitine-related metabolites)

## Outputs

- FDR-BH 0.1 result set: 20 significant metabolite associations on the reduced yogurt dataset
- Trait-wise Bonferroni 0.999 result set: 707 traits with gene associations sorted by strength per trait
- Ranked candidate-gene list (MFS transporter and α/β-hydrolase fold domain-containing protein for carnitine-related metabolites)

## How to apply

Load the orthogene presence/absence matrix (e.g. from OrthoFinder or Roary) and the yogurt metabolomics trait table for the 44 Propionibacterium freudenreichii isolates into Scoary2. First run with multiple_testing set to fdr_bh:0.1 to obtain the strict, per-trait-corrected result set — on the reduced yogurt dataset this recovers 20 significant metabolite associations. Then re-run with trait-wise-correction enabled and multiple_testing set to bonferroni:0.999 to obtain a broad trait-corrected set of 707 traits that retains weak but biologically relevant signals. Rank the genes in the trait-corrected output by association strength per trait to surface top candidates — here an MFS transporter and an α/β-hydrolase fold domain-containing protein correlating strongly with carnitine-related metabolites. The rationale is that FDR-BH 0.1 controls the expected false-discovery proportion among reported metabolites, while the relaxed trait-wise Bonferroni (0.999) trades specificity for recall when hunting candidate genes among many numeric traits.

## Related tools

- **Scoary2** (Runs the mGWAS association of orthogenes with yogurt metabolomics traits, applying either fdr_bh:0.1 or trait-wise bonferroni:0.999 multiple-testing correction) — https://github.com/MrTomRod/scoary-2
- **OrthoFinder** (Generates the orthogroups/orthogenes whose presence/absence matrix serves as the genotype input corrected for multiple testing in Scoary2) — https://github.com/davidemms/OrthoFinder/

## Evaluation signals

- First run with fdr_bh:0.1 on the 44-isolate reduced yogurt dataset yields exactly 20 significant metabolite associations
- Second run with trait-wise-correction True and bonferroni:0.999 yields exactly 707 traits
- The MFS transporter and the α/β-hydrolase fold domain-containing protein rank as the two top candidate genes in the trait-corrected results
- Output is a list of genes sorted by strength of association per trait, as documented for Scoary2
- Carnitine-related metabolites are the traits driving the top gene associations

## Limitations

- Scoary (and, by extension, naive mGWAS pipelines) cannot handle large numbers of microbes or large phenotypic datasets due to performance constraints, inability to pre-process numeric traits, and lack of optimized post-GWAS methods — hence the need for Scoary2 with its tuned correction settings
- Linking bacterial phenotypes to genomic data is confounded because ~40–60% of bacterial genes are undescribed, so relaxed Bonferroni results include genes of unknown function
- The discussion section is not available in the provided document, so additional caveats on the correction procedures could not be extracted

## Evidence

- [abstract] Scoary2 first tool for large phenotypic mGWAS: "With its data exploration app and improved performance, Scoary2 is the first tool to enable the study of large phenotypic datasets using mGWAS"
- [abstract] Two carnitine-metabolism genes discovered across yogurt strains: "we explore the metabolome of yogurts, each produced with a different Propionibacterium reichii strain and discover two genes affecting carnitine metabolism"
- [intro] Scoary limitations motivating the two-run correction strategy: "Three limitations prevent Scoary from such high-throughput analyses: performance constraints, the inability to pre-process numeric traits, and a lack of post-GWAS methods"
- [readme] Scoary2 output format for ranked gene associations per trait: "It reports a list of genes sorted by strength of association per trait. The results can be explored interactively with a simple, static HTML/JS app."
