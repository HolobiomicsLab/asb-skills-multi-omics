---
name: genome-neighborhood-visualization
description: Use when mGWAS/GWAS candidate genes need genomic-context validation —
  e.g. after Scoary2 flags orthogenes correlating with metabolites, to check synteny,
  adjacency, cluster membership, and pseudogenization across isolates.
when_to_use_negative:
- Candidate genes are already annotated with known functions and no genomic-context
  or synteny question remains
- The trait association is plasmid- or mobile-element-driven with no fixed chromosomal
  locus to compare across strains
- Only a single reference genome is available, so per-strain locus comparison across
  isolates is impossible
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_3174
tools:
- name: Scoary2
  role: produces the candidate gene calls (orthogenes associated with carnitine-related
    metabolites) whose genomic neighborhoods are visualized
  repo: https://github.com/MrTomRod/scoary-2
- name: OrthoFinder
  role: provides orthogroup assignments used to map candidate orthogenes onto each
    strain's annotated genome
  repo: https://github.com/davidemms/OrthoFinder
- name: Roary
  role: pan-genome presence/absence matrix linking orthogenes to strains for locus
    extraction
  repo: https://sanger-pathogens.github.io/Roary/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:2b387414dbc0ae10c6675d2cb6285ebad9c7e225c0ab769c83ad16cc98eef14c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MrTomRod/scoary-2
  tools:
  - OrthoFinder
  - Roary
  - Scoary2
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 6
---

# genome-neighborhood-visualization

## Summary

A skill for extracting and rendering syntenic genomic loci around candidate genes from pan-genome orthogroup assignments and GFF annotations, used here to show that two Scoary2 mGWAS candidates (an MFS transporter and an α/β-hydrolase fold protein) sit adjacent to each other one gene from caiA inside the carnitine metabolism cluster (caiABC, fixABCX) of Propionibacterium freudenreichii isolates.

## When to use

Use when mGWAS/GWAS candidate genes need genomic-context validation — e.g. after Scoary2 flags orthogenes correlating with metabolites, to check synteny, adjacency, cluster membership, and pseudogenization across isolates.

## When NOT to use

- Candidate genes are already annotated with known functions and no genomic-context or synteny question remains
- The trait association is plasmid- or mobile-element-driven with no fixed chromosomal locus to compare across strains
- Only a single reference genome is available, so per-strain locus comparison across isolates is impossible

## Inputs

- OrthoFinder orthogroup assignment tables for the P. freudenreichii yogurt strains
- Roary pan-genome gene presence/absence matrix
- Per-strain GFF genome annotation files
- Scoary2 GWAS candidate gene calls (MFS transporter and α/β-hydrolase fold domain-containing protein)

## Outputs

- Per-strain locus comparison figure (Fig. 5E/F style) showing gene order around the candidate genes and the carnitine cluster
- Pseudogenization summary table documenting transposase-disruption events in gene-negative isolates
- Verified adjacency/one-gene-from-caiA relationship calls per strain

## How to apply

Load the OrthoFinder/Roary orthogroup assignments together with each strain's GFF genome annotation and the Scoary2 candidate gene calls. Extract the genomic neighborhoods containing the candidates plus the cai/caiABC and fixABCX loci from each annotation, and verify synteny: the MFS transporter and α/β-hydrolase fold genes are adjacent to each other and lie one gene away from the L-carnitine CoA transferase (caiA) within a cluster also containing caiABC and fixABCX. For gene-negative isolates, screen the corresponding locus for transposase ORFs, frameshifts, and truncated ORFs that indicate insertion-driven disruption or pseudogenization. Render a per-strain locus comparison figure (Fig. 5E/F style) and write a pseudogenization summary table linking absence of the candidate genes to a documented disruption event.

## Related tools

- **Scoary2** (produces the candidate gene calls (orthogenes associated with carnitine-related metabolites) whose genomic neighborhoods are visualized) — https://github.com/MrTomRod/scoary-2
- **OrthoFinder** (provides orthogroup assignments used to map candidate orthogenes onto each strain's annotated genome) — https://github.com/davidemms/OrthoFinder
- **Roary** (pan-genome presence/absence matrix linking orthogenes to strains for locus extraction) — https://sanger-pathogens.github.io/Roary/

## Examples

```
orthofinder -f proteomes/ -o pan_results && python scripts/render_loci.py --gff strains/*.gff --orthogroups pan_results/Orthogroups.tsv --candidates scoary2_hits.tsv --locus caiA --flank 5 --out fig5E_F_locus_comparison
```

## Evaluation signals

- For every gene-positive isolate, the two candidates are reproduced as adjacent to each other and one gene away from caiA within the caiABC+fixABCX cluster
- Every isolate lacking the candidate genes has a documented transposase-disruption or pseudogenization event (transposase ORF, frameshift, or truncated ORF) at the corresponding locus
- Gene order in the rendered per-strain locus figure is consistent with the GFF coordinates and orthogroup assignments
- The pseudogenization summary table covers all strains and matches the per-strain figure content (Fig. 5E/F style consistency)

## Limitations

- Synteny validation depends on complete, accurate GFF annotations; assembly gaps or misannotations at the carnitine locus can break adjacency calls
- Pseudogenization detection is limited to signatures findable in the annotation (transposase ORFs, frameshifts, truncated ORFs) and may miss subtler loss-of-function mutations
- Orthogroup-based locus identification inherits errors from OrthoFinder/Roary paralogous or split orthogroup assignments

## Evidence

- [abstract] we explore the metabolome of yogurts, each produced with a different Propionibacterium reichii strain and discover two genes affecting carnitine metabolism: "we explore the metabolome of yogurts, each produced with a different Propionibacterium reichii strain and discover two genes affecting carnitine metabolism"
- [readme] Scoary2 associates orthogenes with traits via OrthoFinder/Roary: "Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder] or [Roary][roary] to traits. It reports a list of genes sorted by strength of association per trait."
- [other] The two candidate genes are adjacent to each other, located in a syntenic gene cluster one gene away from caiA: [evidence span withheld — anchor status: not_found]
