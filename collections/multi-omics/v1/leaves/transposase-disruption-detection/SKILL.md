---
name: transposase-disruption-detection
description: Use when Scoary2 mGWAS flags candidate genes present in only a subset
  of isolates, and GFF genome annotations plus orthogroup assignments (OrthoFinder/Roary)
  are available to test whether gene-negative isolates carry disrupted or pseudogenized
  copies of the locus.
when_to_use_negative:
- The Scoary2 association has not yet been run — this skill is a downstream validation
  step, not a GWAS method
- All isolates are gene-positive at the candidate locus, leaving no gene-negative
  annotations to screen for disruption events
- Only protein FASTA files are available without GFF/coordinate annotations, so frameshifts,
  truncations, and ORF positions cannot be assessed
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: OrthoFinder
  role: Provides the orthogroup assignments used to place the Scoary2 candidate genes
    into the syntenic carnitine cluster across isolates
  repo: https://github.com/davidemms/OrthoFinder/
- name: Roary
  role: Supplies pan-genome orthogene presence/absence and gene annotations per strain
    used to locate the candidate locus in each GFF
  repo: https://sanger-pathogens.github.io/Roary/
- name: Scoary2
  role: Produces the candidate gene calls (MFS transporter, α/β-hydrolase) correlating
    with carnitine-related metabolites, which this skill validates structurally
  repo: https://github.com/MrTomRod/scoary-2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:6dad97cef068b3e97ac8794bf2f419a63910f6db65202ae8c23b5af7c083e822
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/davidemms/OrthoFinder
  tools:
  - OrthoFinder
  - Roary
  - Scoary2
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 6
---

# transposase-disruption-detection

## Summary

A post-GWAS genomic-validation skill for microbial pan-genome studies: screen the GFF annotations of isolates that lack Scoary2-associated candidate genes for transposase ORFs, frameshifts, and truncated ORFs that explain the absence via insertion-driven disruption or pseudogenization, within the syntenic locus context (here the carnitine cluster with caiA/caiABC/fixABCX in Propionibacterium freudenreichii).

## When to use

Use when Scoary2 mGWAS flags candidate genes present in only a subset of isolates, and GFF genome annotations plus orthogroup assignments (OrthoFinder/Roary) are available to test whether gene-negative isolates carry disrupted or pseudogenized copies of the locus.

## When NOT to use

- The Scoary2 association has not yet been run — this skill is a downstream validation step, not a GWAS method
- All isolates are gene-positive at the candidate locus, leaving no gene-negative annotations to screen for disruption events
- Only protein FASTA files are available without GFF/coordinate annotations, so frameshifts, truncations, and ORF positions cannot be assessed

## Inputs

- Scoary2 candidate gene calls (gene-positive/gene-negative isolate assignments per trait)
- OrthoFinder or Roary orthogroup assignment tables for the P. reichii yogurt strains
- Per-isolate GFF genome annotation files covering the carnitine gene cluster (caiA, caiABC, fixABCX loci)

## Outputs

- Per-strain locus comparison figure (Fig. 5E/F style) showing intact vs. disrupted candidate loci
- Pseudogenization summary table (strain, disrupted ORF, transposase insertion, frameshift/truncation evidence)
- Verified syntenic map confirming candidate-gene adjacency and one-gene-from-caiA placement

## How to apply

First confirm synteny in the gene-positive isolates: extract the genomic neighborhoods containing the Scoary2 candidate genes (e.g. the MFS transporter and α/β-hydrolase fold domain protein) and verify the two candidates are adjacent and lie one gene away from caiA within a cluster containing caiABC and fixABCX, using OrthoFinder orthogroup assignments and per-strain GFF coordinates. Then, for every isolate whose pan-genome matrix shows the candidates absent, scan the same locus in its GFF annotation for transposase ORFs, frameshifts, and truncated ORFs that indicate insertion-driven disruption or pseudogenization of the candidate locus. Record the disrupted element, its position relative to the intact syntenic cluster, and the transposase family/ORF evidence per strain. Render a per-strain locus comparison figure in the Fig. 5E/F style and compile a pseudogenization summary table so the genotype–metabolite association (carnitine-related metabolites) is mechanistically explained.

## Related tools

- **OrthoFinder** (Provides the orthogroup assignments used to place the Scoary2 candidate genes into the syntenic carnitine cluster across isolates) — https://github.com/davidemms/OrthoFinder/
- **Roary** (Supplies pan-genome orthogene presence/absence and gene annotations per strain used to locate the candidate locus in each GFF) — https://sanger-pathogens.github.io/Roary/
- **Scoary2** (Produces the candidate gene calls (MFS transporter, α/β-hydrolase) correlating with carnitine-related metabolites, which this skill validates structurally) — https://github.com/MrTomRod/scoary-2

## Evaluation signals

- For every gene-positive isolate, the two candidate genes are reproduced as adjacent to each other and one gene away from caiA within a caiABC + fixABCX cluster
- For every gene-negative isolate, at least one disruption event (transposase ORF, frameshift, or truncated ORF) at the candidate locus is documented in the pseudogenization summary table
- Orthogroup membership from OrthoFinder/Roary is consistent with the GFF coordinates of the extracted genomic neighborhoods
- The per-strain locus comparison figure matches the disrupted/intact pattern reported in Fig. 5E/F style
- No candidate-locus disruption is claimed for strains where the intact syntenic cluster is present in the GFF

## Limitations

- Disruption calling depends entirely on GFF annotation quality — unannotated transposase remnants or frameshifts in poorly assembled genomes can be missed
- The skill validates only loci already flagged by Scoary2 association; it cannot discover disrupted genes when association signals are absent
- Scoary2's scope is linking orthogenes to traits (40-60% of bacterial genes remain undescribed), so functional annotation of the disrupted elements may be limited

## Evidence

- [other] Screen the gene-negative isolates' annotations for transposase ORFs, frameshifts, and truncated ORFs indicating insertion-driven disruption or pseudogenization of the candidate locus.: [evidence span withheld — anchor status: not_found]
- [other] The two candidate genes are adjacent to each other, located in a syntenic gene cluster one gene away from an L-carnitine CoA transferase (caiA).: [evidence span withheld — anchor status: not_found]
- [abstract] Two genes affecting carnitine metabolism were discovered in yogurt produced with different Propionibacterium reichii strains.: [evidence span withheld — anchor status: not_found]
- [readme] Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder]: "Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder]"
