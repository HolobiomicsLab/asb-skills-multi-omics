---
name: orthogroup-locus-synteny-analysis
description: Use when Scoary2 (or Scoary) mGWAS hits need genomic-context validation
  — e.g. checking whether trait-associated orthogenes are adjacent, syntenic, or disrupted
  (transposases/frameshifts) in positive vs. negative isolates.
when_to_use_negative:
- When the goal is the mGWAS association itself rather than validating hit locations
  — use Scoary2 on the gene-presence/trait matrix instead
- When candidate genes lack a known syntenic landmark gene (e.g. caiA) or GFF annotations,
  since adjacency and one-gene-from-caiA checks cannot be performed
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: OrthoFinder
  role: generates the orthogroups/orthogenes whose genomic neighborhoods are examined
    for synteny
  repo: https://github.com/davidemms/OrthoFinder/
- name: Roary
  role: pan-genome orthogroup assignments used as an alternative source of orthogene
    calls for locus mapping
  repo: https://sanger-pathogens.github.io/Roary/
- name: Scoary2
  role: produces the candidate gene calls (MFS transporter, α/β-hydrolase) whose locus
    positions are validated against the carnitine cluster
  repo: https://github.com/MrTomRod/scoary-2
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:d3e7f1cdba576079017b20797cf8c34b92438267119de5cc2f297aef531dea53
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/davidemms/OrthoFinder
  tools:
  - OrthoFinder
  - Roary
  - Scoary2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# orthogroup-locus-synteny-analysis

## Summary

A post-GWAS genomic-neighborhood verification skill: it maps Scoary2 candidate orthogenes (e.g. an MFS transporter and an α/β-hydrolase fold protein associated with carnitine metabolites) onto GFF annotations to confirm adjacency, one-gene-from-landmark relationships (caiA), and cluster membership (caiABC, fixABCX) across strains.

## When to use

Use when Scoary2 (or Scoary) mGWAS hits need genomic-context validation — e.g. checking whether trait-associated orthogenes are adjacent, syntenic, or disrupted (transposases/frameshifts) in positive vs. negative isolates.

## When NOT to use

- When the goal is the mGWAS association itself rather than validating hit locations — use Scoary2 on the gene-presence/trait matrix instead
- When candidate genes lack a known syntenic landmark gene (e.g. caiA) or GFF annotations, since adjacency and one-gene-from-caiA checks cannot be performed

## Inputs

- OrthoFinder orthogroup assignments (Orthogroups.tsv / orthogene table per strain)
- Roary pan-genome gene presence/absence matrix
- Scoary2 association results listing trait-associated orthogenes (e.g. MFS transporter, α/β-hydrolase fold protein vs. carnitine metabolites)
- GFF genome annotation files for each P. freudenreichii isolate
- Strain metadata distinguishing gene-positive vs. gene-negative isolates

## Outputs

- Per-strain locus comparison figure (Fig. 5E/F style) showing candidate gene adjacency relative to caiA/caiABC/fixABCX
- Pseudogenization summary table documenting transposase-disruption events in gene-negative isolates
- Verified syntenic cluster call per strain (candidates adjacent, one gene from caiA, within caiABC+fixABCX cluster)

## How to apply

Start from the OrthoFinder/Roary orthogroup assignments and the GFF genome annotations for each Propionibacterium freudenreichii isolate, plus the Scoary2 candidate gene calls (here: an MFS transporter and an α/β-hydrolase fold domain-containing protein correlated with carnitine-related metabolites). Extract the genomic neighborhoods containing the candidate genes and the known carnitine loci (caiA, caiABC, fixABCX) from each strain's GFF. Verify synteny by confirming that the two candidates are adjacent to each other and lie one gene away from caiA within the carnitine metabolism gene region. For gene-negative isolates, screen the same locus for transposase ORFs, frameshifts, and truncated ORFs that indicate insertion-driven disruption or pseudogenization. Render a per-strain locus comparison figure (Fig. 5E/F style) and write a pseudogenization summary table.

## Related tools

- **OrthoFinder** (generates the orthogroups/orthogenes whose genomic neighborhoods are examined for synteny) — https://github.com/davidemms/OrthoFinder/
- **Roary** (pan-genome orthogroup assignments used as an alternative source of orthogene calls for locus mapping) — https://sanger-pathogens.github.io/Roary/
- **Scoary2** (produces the candidate gene calls (MFS transporter, α/β-hydrolase) whose locus positions are validated against the carnitine cluster) — https://github.com/MrTomRod/scoary-2

## Examples

```
orthofinder -f ./proteome_fasta/ -o orthofinder_results  # orthogroups + GFF loci are then checked for candidate adjacency near caiA/caiABC/fixABCX
```

## Evaluation signals

- For every gene-positive isolate, the two candidate genes are confirmed adjacent to each other in the GFF coordinates
- Candidates lie exactly one gene away from the L-carnitine CoA transferase caiA within the carnitine metabolism gene region
- caiABC and fixABCX cluster membership is reproduced for gene-positive isolates across all strains
- Every gene-negative isolate has a documented transposase-disruption, frameshift, or truncated-ORF pseudogenization event at the candidate locus
- The per-strain locus comparison figure and pseudogenization summary table match the Fig. 5E/F patterns

## Limitations

- Validation depends on the quality/completeness of each strain's GFF annotation; frameshifts and truncated ORFs may reflect assembly or annotation artifacts rather than true pseudogenization
- Requires a recognizable landmark locus (caiA/caiABC/fixABCX) conserved enough to anchor the synteny comparison across isolates

## Evidence

- [abstract] we explore the metabolome of yogurts, each produced with a different Propionibacterium reichii strain and discover two genes affecting carnitine metabolism: "we explore the metabolome of yogurts, each produced with a different Propionibacterium reichii strain and discover two genes affecting carnitine metabolism"
- [readme] Scoary2 README on orthogenes: "Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder] or [Roary][roary] to traits."
- [readme] OrthoFinder README on orthogroup inference: [evidence span withheld — anchor status: external_doc]
- [intro] Study aim on pan-genome vs. phenotype: "The aim of this study was to investigate the effect of the pan-genome of the added bacterial strains on the phenotype of the yogurts."
