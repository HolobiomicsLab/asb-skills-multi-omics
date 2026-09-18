---
name: carnitine-metabolism-gene-annotation
description: Use when Scoary2 (or similar mGWAS) hits on carnitine-related yogurt
  metabolites must be localized within a strain's genome to verify whether candidate
  orthogenes sit inside a known carnitine-metabolism gene cluster.
when_to_use_negative:
- Input is only raw metabolomics data without genome annotations or orthogroup assignments
  — the skill requires GFFs and pan-genome orthologs, not spectra
- The phenotype of interest is not linked to candidate genes from a pan-genome association
  run (e.g., you only have a single reference genome with no between-strain variation
  to compare loci)
- The candidates have already known functional annotation and cluster membership,
  making locus-based inference of function redundant
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3172
tools:
- name: Scoary2
  role: Performs the mGWAS that surfaces the two candidate orthogenes correlated with
    carnitine-related metabolites; results are the input calls to be placed in genomic
    context
  repo: https://github.com/MrTomRod/scoary-2
- name: OrthoFinder
  role: Generates the orthogroup assignments that define the candidate genes and their
    strain-wise presence/absence used for locus extraction
  repo: https://github.com/davidemms/OrthoFinder
- name: Roary
  role: Alternative pan-genome pipeline producing the orthogene presence/absence matrix
    fed to Scoary2 and used for neighborhood comparison
  repo: https://sanger-pathogens.github.io/Roary/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1186/s13059-024-03233-7
    title: 'Scoary2: rapid association of phenotypic multi-omics data with microbial
      pan-genomes'
  source_content_hash: sha256:3d77304b1f41c46e3a2e8c175e1d6fc2bd2d9268298ad194123897028d9a2b3e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MrTomRod/scoary-2
  tools:
  - OrthoFinder
  - Roary
  - Scoary2
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# carnitine-metabolism-gene-annotation

## Summary

Annotate and validate the syntenic carnitine gene cluster in Propionibacterium freudenreichii isolates by placing Scoary2 mGWAS candidate genes (an MFS transporter and an α/β-hydrolase fold protein) into their genomic context relative to caiA (L-carnitine CoA transferase), the caiABC locus, and the fixABCX operon.

## When to use

Use when Scoary2 (or similar mGWAS) hits on carnitine-related yogurt metabolites must be localized within a strain's genome to verify whether candidate orthogenes sit inside a known carnitine-metabolism gene cluster.

## When NOT to use

- Input is only raw metabolomics data without genome annotations or orthogroup assignments — the skill requires GFFs and pan-genome orthologs, not spectra
- The phenotype of interest is not linked to candidate genes from a pan-genome association run (e.g., you only have a single reference genome with no between-strain variation to compare loci)
- The candidates have already known functional annotation and cluster membership, making locus-based inference of function redundant

## Inputs

- OrthoFinder/Roary orthogroup assignment tables (pan-genome matrix) for P. freudenreichii strains
- GFF genome annotation files per yogurt strain
- Scoary2 association results for carnitine-related metabolite traits (top candidate orthogenes)
- Protein FASTA files per species (one per species, for OrthoFinder)

## Outputs

- Per-strain carnitine locus comparison figure (Fig. 5E/F style)
- Pseudogenization summary table documenting transposase-disruption events
- Verified adjacency/one-gene-from-caiA map of the two candidate genes within the caiABC+fixABCX cluster

## How to apply

Load the OrthoFinder/Roary orthogroup assignments and the per-strain GFF genome annotations for the P. reichii yogurt strains, plus the significant Scoary2 candidate gene calls for carnitine-related metabolites. Extract the genomic neighborhoods containing the two candidates (MFS transporter, α/β-hydrolase fold domain protein) and the cai/caiABC and fixABCX loci. Verify synteny: the two candidates should be adjacent to each other and lie one gene away from caiA within the cluster containing caiABC and fixABCX. For gene-negative isolates, screen the annotations for transposase ORFs, frameshifts, and truncated ORFs that indicate insertion-driven disruption or pseudogenization of the candidate locus. The rationale is that adjacency to the carnitine CoA transferase caiA gives functional support to unannotated GWAS candidates, addressing the fact that 40-60% of bacterial genes remain undescribed.

## Related tools

- **Scoary2** (Performs the mGWAS that surfaces the two candidate orthogenes correlated with carnitine-related metabolites; results are the input calls to be placed in genomic context) — https://github.com/MrTomRod/scoary-2
- **OrthoFinder** (Generates the orthogroup assignments that define the candidate genes and their strain-wise presence/absence used for locus extraction) — https://github.com/davidemms/OrthoFinder
- **Roary** (Alternative pan-genome pipeline producing the orthogene presence/absence matrix fed to Scoary2 and used for neighborhood comparison) — https://sanger-pathogens.github.io/Roary/

## Examples

```
orthofinder -f proteomes/ -M msa && python -m scoary2 --traits carnitine_metabolites.csv --genes gene_presence_absence.csv
```

## Evaluation signals

- The two candidate genes are verified adjacent to each other in every gene-positive isolate's annotation
- The candidates lie exactly one gene away from caiA (L-carnitine CoA transferase) in the syntenic cluster
- The cluster membership includes both caiABC and fixABCX loci for all gene-positive strains
- Every isolate lacking the candidate genes has a documented transposase-disruption, frameshift, or truncated ORF event at the locus, consistent with pseudogenization
- The rendered per-strain locus comparison reproduces the Fig. 5E/F-style synteny pattern

## Limitations

- The two Scoary2 candidate genes are undescribed (consistent with 40-60% of bacterial genes lacking function), so their role is inferred from cluster adjacency rather than direct annotation
- Association from mGWAS is not causal proof; the carnitine-effect claim rests on correlation plus genomic context
- Gene absence must not be equated with true absence until transposase/frameshift/truncated-ORF screening rules out assembly or annotation artifacts

## Evidence

- [abstract] we explore the metabolome of yogurts, each produced with a different Propionibacterium reichii strain and discover two genes affecting carnitine metabolism: "we explore the metabolome of yogurts, each produced with a different Propionibacterium reichii strain and discover two genes affecting carnitine metabolism"
- [readme] Scoary2 associates orthogenes with traits: "Scoary2 associates orthogenes (e.g. generated using [OrthoFinder][orthofinder]
or [Roary][roary] to traits. It reports a list of genes sorted by strength of
association per trait."
- [abstract] 40-60% of bacterial genes are undescribed: "linking them to genomic data is challenging, leaving 40-60% of bacterial genes undescribed"
- [abstract] Scoary2 is the first tool to enable large phenotypic datasets: "With its data exploration app and improved performance, Scoary2 is the first tool to enable the study of large phenotypic datasets using mGWAS"
- [readme] OrthoFinder input is protein FASTA per species: [evidence span withheld — anchor status: external_doc]
