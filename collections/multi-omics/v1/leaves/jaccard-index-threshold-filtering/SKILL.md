---
name: jaccard-index-threshold-filtering
description: Use when preparing RNA-seq simulation inputs and you must restrict candidate
  genes to a biologically coherent, GO-annotated set (e.g. protein-coding genes on
  a single chromosome such as chr22) before count-matrix generation.
when_to_use_negative:
- The analysis goal is GO enrichment of already-identified DE genes rather than pre-selecting
  a simulation gene set (eduomics handles that separately, e.g. with clusterProfiler)
- Input genes lack GO annotations or chromosome assignment, since the Jaccard network
  and chr22 restriction both depend on these
- A genome-wide or multi-chromosome gene set is required; eduomics intentionally focuses
  on single chromosomes (typically chr22) to reduce computational requirements
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_0203
tools:
- name: org.Hs.eg.db
  role: Provides human gene-to-GO-term annotations mapped to Entrez identifiers for
    building the similarity network
- name: igraph
  role: Builds the gene-gene network weighted by Jaccard index of GO-term sets and
    prunes edges below simthreshold
- name: clusterProfiler
  role: GO annotation/enrichment toolkit used in the RNA workflow around gene selection
    and downstream enrichment analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:b949bf7898f0c36e44061a4ebf77efc0bf75870a5e67e33494086917da8a12e5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - clusterProfiler
  - igraph
  - org.Hs.eg.db
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# jaccard-index-threshold-filtering

## Summary

Selects genes suitable for differential expression simulation in the eduomics RNA workflow by building a gene-gene similarity network from Gene Ontology annotations and pruning edges below a Jaccard-index threshold (simthreshold, default 0.3).

## When to use

Use when preparing RNA-seq simulation inputs and you must restrict candidate genes to a biologically coherent, GO-annotated set (e.g. protein-coding genes on a single chromosome such as chr22) before count-matrix generation.

## When NOT to use

- The analysis goal is GO enrichment of already-identified DE genes rather than pre-selecting a simulation gene set (eduomics handles that separately, e.g. with clusterProfiler)
- Input genes lack GO annotations or chromosome assignment, since the Jaccard network and chr22 restriction both depend on these
- A genome-wide or multi-chromosome gene set is required; eduomics intentionally focuses on single chromosomes (typically chr22) to reduce computational requirements

## Inputs

- Human gene-to-GO-term annotations (via org.Hs.eg.db)
- Gene annotation with chromosome/coordinate and biotype information (e.g. GTF from the GATK.GRCh38 reference)
- simthreshold parameter from the samplesheet (default 0.3)
- Target chromosome identifier (e.g. chr22)

## Outputs

- selected_genes.txt: one gene identifier per line, suitable for differential expression simulation
- Filtered gene-gene similarity network (edges with Jaccard weight >= simthreshold)

## How to apply

Load human gene-to-GO-term annotations via org.Hs.eg.db and map all annotated genes to Entrez identifiers. Using igraph, build a gene-gene network in which genes are connected by shared GO terms, weighting each edge by the Jaccard index of the two genes' GO-term sets. Prune the network by removing edges whose Jaccard similarity falls below simthreshold = 0.3 (the value passed via the samplesheet's simthreshold column). Restrict network nodes to protein-coding genes on the target chromosome (typically chr22), using chromosome/coordinate information from the gene annotation. The genes remaining in the filtered network are written to selected_genes.txt, one identifier per line, for downstream count-matrix generation with known differential expression patterns.

## Related tools

- **org.Hs.eg.db** (Provides human gene-to-GO-term annotations mapped to Entrez identifiers for building the similarity network)
- **igraph** (Builds the gene-gene network weighted by Jaccard index of GO-term sets and prunes edges below simthreshold)
- **clusterProfiler** (GO annotation/enrichment toolkit used in the RNA workflow around gene selection and downstream enrichment analysis)

## Examples

```
simthreshold <- 0.3; g <- delete_edges(g, E(g)[weight < simthreshold])
```

## Evaluation signals

- selected_genes.txt exists and is non-empty, with one gene identifier per line
- Every listed gene maps to chromosome 22
- All retained network edges have Jaccard weight >= simthreshold = 0.3
- Genes excluded from the network have no GO-term set meeting the 0.3 Jaccard cutoff against any retained gene
- Selected genes are protein-coding according to the gene annotation biotype

## Limitations

- Restriction to protein-coding genes on a single chromosome (typically chr22) trades biological breadth for computational tractability, so the selected gene set is not genome-wide
- The Jaccard cutoff (default 0.3) determines network density; genes whose GO-term sets share no sufficiently similar neighbours are dropped entirely
- Only simulations passing validation are included in the final output, so a failed gene-selection validation invalidates downstream RNA simulation steps

## Evidence

- [intro] Identifies genes suitable for differential expression based on Gene Ontology (GO) annotation.: "Identifies genes suitable for differential expression based on Gene Ontology (GO) annotation."
- [readme] Chromosome restriction rationale: [evidence span withheld — anchor status: not_found]
- [readme] simthreshold column in samplesheet: [evidence span withheld — anchor status: not_found]
- [intro] Downstream use of selected genes: "Creates realistic count matrices with known differential expression patterns."
