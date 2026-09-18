---
name: pangenome-neighborhood-reconstruction
description: 'Use when Scoary2/pan-GWAS candidate orthogenes must be validated in
  genomic context: e.g. checking whether correlated genes sit adjacent to known pathway
  loci (caiA/caiABC, fixABCX) and whether gene-negative isolates carry pseudogene
  disruptions.'
when_to_use_negative:
- Input is already a curated per-locus feature table with verified gene order — neighborhood
  reconstruction would be redundant
- The GWAS candidates have no annotated reference pathway genes nearby to anchor the
  neighborhood comparison (nothing to verify synteny against)
- Only a single genome is available and no cross-isolate synteny or pseudogenization
  screen is intended
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0194
tools:
- name: OrthoFinder
  role: Infers the orthogroups whose per-strain members are placed into genomic neighborhoods
    for the synteny check
  repo: https://github.com/davidemms/OrthoFinder/
- name: Roary
  role: Builds the pan-genome (gene presence/absence) against which candidate-gene-positive
    and -negative isolates are contrasted
  repo: https://sanger-pathogens.github.io/Roary/
- name: Scoary2
  role: Supplies the GWAS candidate orthogenes (correlating with carnitine-related
    metabolites) whose neighborhood is reconstructed
  repo: https://github.com/MrTomRod/scoary-2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:1be4e754fbbf94b6b7f3493a9af02180b6ea5cd2cd56e27856260199888222b3
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
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Pangenome neighborhood reconstruction

## Summary

Reconstructs the genomic neighborhood of mGWAS candidate orthogenes by extracting loci from per-strain GFF annotations and verifying synteny (adjacency, cluster membership) across isolates, as done for the carnitine cluster of Propionibacterium freudenreichii yogurt strains.

## When to use

Use when Scoary2/pan-GWAS candidate orthogenes must be validated in genomic context: e.g. checking whether correlated genes sit adjacent to known pathway loci (caiA/caiABC, fixABCX) and whether gene-negative isolates carry pseudogene disruptions.

## When NOT to use

- Input is already a curated per-locus feature table with verified gene order — neighborhood reconstruction would be redundant
- The GWAS candidates have no annotated reference pathway genes nearby to anchor the neighborhood comparison (nothing to verify synteny against)
- Only a single genome is available and no cross-isolate synteny or pseudogenization screen is intended

## Inputs

- OrthoFinder or Roary orthogroup assignment table for the P. reichii yogurt strain pan-genome
- Per-strain GFF genome annotation files
- Scoary2 association results listing candidate orthogenes (e.g. MFS transporter, alpha/beta-hydrolase fold protein) with trait correlations
- Annotation of known reference loci (caiA, caiABC, fixABCX) in each strain

## Outputs

- Per-strain locus comparison figure (Fig. 5E/F style) showing syntenic neighborhoods of the candidate genes
- Pseudogenization summary table documenting transposase-disruption or truncation events in gene-negative isolates
- Synteny verification report (adjacency of candidate pair; one-gene distance to caiA; caiABC+fixABCX cluster membership)

## How to apply

Load the OrthoFinder/Roary orthogroup assignments together with the GFF genome annotations for each Propionibacterium reichii isolate, plus the Scoary2 GWAS candidate gene calls (here, an MFS transporter and an alpha/beta-hydrolase fold domain-containing protein correlating with carnitine-related metabolites). Extract the genomic neighborhoods containing the candidate genes and reference loci (caiA, caiABC, fixABCX) from each strain's annotation. Verify synteny: confirm the two candidates are adjacent to each other and lie one gene away from caiA within a cluster containing caiABC and fixABCX in gene-positive isolates. Screen gene-negative isolates' annotations for transposase ORFs, frameshifts, and truncated ORFs indicating insertion-driven disruption or pseudogenization. Rationale: orthogroup identity alone ignores locus architecture, and insertion elements can explain GWAS-absent phenotypes without true gene absence.

## Related tools

- **OrthoFinder** (Infers the orthogroups whose per-strain members are placed into genomic neighborhoods for the synteny check) — https://github.com/davidemms/OrthoFinder/
- **Roary** (Builds the pan-genome (gene presence/absence) against which candidate-gene-positive and -negative isolates are contrasted) — https://sanger-pathogens.github.io/Roary/
- **Scoary2** (Supplies the GWAS candidate orthogenes (correlating with carnitine-related metabolites) whose neighborhood is reconstructed) — https://github.com/MrTomRod/scoary-2

## Examples

```
orthofinder -f proteomes/   # then map Scoary2 hit orthogroups onto per-strain GFF loci for synteny check
```

## Evaluation signals

- For every gene-positive isolate, the two Scoary2 candidates are adjacent to each other in the GFF-derived neighborhood
- The candidate pair lies exactly one gene away from the L-carnitine CoA transferase caiA in the reconstructed locus
- The reconstructed cluster also contains caiABC and fixABCX members, reproducing the reported carnitine-metabolism region
- Every isolate lacking the candidate genes has a documented transposase-disruption, frameshift, or truncated-ORF event at the locus
- The per-strain locus comparison figure matches the reported Fig. 5E/F pattern across all isolates

## Limitations

- Roary/OrthoFinder orthogroup calls are annotation-dependent; frameshifts or truncated ORFs can split or hide orthologs, requiring manual GFF inspection
- Neighborhood conclusions hold only for the pan-genome analyzed (20 Agroscope (sub)species strains in the source study); other collections may show different synteny
- Transposase-driven disruption explains gene absence only in the isolates examined; absence in other strains may have other causes

## Evidence

- [abstract] we explore the metabolome of yogurts, each produced with a different Propionibacterium reichii strain and discover two genes affecting carnitine metabolism: "we explore the metabolome of yogurts, each produced with a different Propionibacterium reichii strain and discover two genes affecting carnitine metabolism"
- [readme] Scoary2 associates orthogenes generated with OrthoFinder or Roary to traits: "Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder]
or [Roary][roary] to traits."
- [abstract] 40-60% of bacterial genes are undescribed: "linking them to genomic data is challenging, leaving 40-60% of bacterial genes undescribed"
