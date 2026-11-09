---
name: gene-identifier-mapping
description: Use when a simulation or analysis step requires harmonising gene identifiers
  (e.g., symbols to Entrez) before constructing GO-annotation-based similarity networks
  or filtering genes by annotation content.
when_to_use_negative:
- The target organism is not human — org.Hs.eg.db covers only Homo sapiens annotations
- Genes are already in Entrez identifier space and no GO-annotation-based selection
  or filtering is required
- The downstream task needs whole-genome gene sets rather than the chromosome-22-restricted,
  simthreshold-filtered subset eduomics produces
edam_topics:
- http://edamontology.org/topic_0089
tools:
- name: org.Hs.eg.db
  role: Provides human gene-to-GO-term annotations and maps annotated genes to Entrez
    identifiers
- name: igraph
  role: Builds and prunes the gene-gene similarity network whose nodes are the Entrez-mapped
    genes, with Jaccard-weighted edges cut at simthreshold = 0.3
- name: clusterProfiler
  role: Supports GO annotation handling in the gene-selection workflow
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:8edb0c379f0ec6bcb56a3264d96bc7a04d01ab9f860b870dfd65daeab4e3cc22
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - clusterProfiler
  - igraph
  - org.Hs.eg.db
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# gene-identifier-mapping

## Summary

Maps genes to Entrez identifiers via org.Hs.eg.db as the first step of the eduomics RNA workflow's GO-based gene selection, which builds a Jaccard-weighted GO similarity network to pick genes suitable for differential expression simulation.

## When to use

Use when a simulation or analysis step requires harmonising gene identifiers (e.g., symbols to Entrez) before constructing GO-annotation-based similarity networks or filtering genes by annotation content.

## When NOT to use

- The target organism is not human — org.Hs.eg.db covers only Homo sapiens annotations
- Genes are already in Entrez identifier space and no GO-annotation-based selection or filtering is required
- The downstream task needs whole-genome gene sets rather than the chromosome-22-restricted, simthreshold-filtered subset eduomics produces

## Inputs

- Human gene-to-GO-term annotations (org.Hs.eg.db)
- Gene annotation with chromosome/coordinate and protein-coding information
- Samplesheet with simthreshold (default 0.3) and chromosome (e.g. chr22) columns

## Outputs

- Entrez-mapped gene identifiers serving as network nodes
- selected_genes.txt with one gene identifier per line, suitable for differential expression simulation

## How to apply

Load human gene-to-GO-term annotations via org.Hs.eg.db and map all annotated genes to Entrez identifiers as the entry point of the gene-selection step. Use the mapped Entrez IDs as node identifiers when building the igraph gene-gene similarity network, where edges represent shared GO terms and are weighted by the Jaccard index of the two genes' GO-term sets. Prune edges with Jaccard similarity below simthreshold = 0.3 (the default passed in the samplesheet's simthreshold column), then restrict nodes to protein-coding genes on chromosome 22 using gene annotation coordinates. The mapping is a prerequisite: genes that fail to map to Entrez cannot participate in the GO similarity network and are implicitly excluded from selected_genes.txt.

## Related tools

- **org.Hs.eg.db** (Provides human gene-to-GO-term annotations and maps annotated genes to Entrez identifiers)
- **igraph** (Builds and prunes the gene-gene similarity network whose nodes are the Entrez-mapped genes, with Jaccard-weighted edges cut at simthreshold = 0.3)
- **clusterProfiler** (Supports GO annotation handling in the gene-selection workflow)

## Examples

```
Rscript -e 'library(org.Hs.eg.db); entrez <- mapIds(org.Hs.eg.db, keys=go_gene_symbols, keytype="SYMBOL", column="ENTREZID")'
```

## Evaluation signals

- selected_genes.txt exists and is non-empty
- Every gene listed in selected_genes.txt maps to chromosome 22
- All retained edges in the similarity network meet the simthreshold = 0.3 Jaccard cutoff
- Every gene entering the network resolves to a valid Entrez identifier via org.Hs.eg.db

## Limitations

- Selection is restricted to a single chromosome (typically chr22) to reduce computational requirements, so the gene pool is deliberately narrow
- The 0.3 Jaccard cutoff (simthreshold default) discards genes whose GO-term overlap is weak; genes with few or no GO annotations may be excluded entirely
- Only protein-coding genes on the target chromosome are retained; non-coding genes are dropped even if well annotated

## Evidence

- [intro] GO-based gene selection intro quote: "Identifies genes suitable for differential expression based on Gene Ontology (GO) annotation."
- [readme] README samplesheet with simthreshold: "id,type,chromosome,coverage,capture,reps,groups,simthreshold"
- [methods] Source card workflow description of mapping and Jaccard network: [evidence span withheld — anchor status: not_found]
