---
name: chromosome-region-gene-subsetting
description: Use when building an RNA differential expression simulation for a single
  chromosome and you need to define a biologically plausible, GO-connected gene universe
  (e.g., protein-coding genes on chr22 passing a Jaccard simthreshold of 0.3) before
  simulating counts.
when_to_use_negative:
- Input is already a curated gene list or expression matrix — this skill is for defining
  the gene universe before simulation, not for re-filtering existing DE results.
- The analysis goal is genome-wide rather than single-chromosome — the pipeline explicitly
  focuses on single chromosomes (typically chr22) and the subsetting logic assumes
  that scope.
- The task is DNA variant simulation or variant calling — the RNA gene-selection step
  does not apply to the DNA branch of eduomics.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: org.Hs.eg.db
  role: provides human gene-to-GO-term annotations mapped to Entrez identifiers as
    the basis for the similarity network
- name: igraph
  role: builds and prunes the gene-gene similarity network weighted by the Jaccard
    index of GO-term sets at simthreshold = 0.3
- name: clusterProfiler
  role: GO annotation tooling supporting enrichment-style gene selection logic
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:d178bf7095cbeed6560d76c560f42ee38d12b85692e9ffc49336b0dd9eb90afb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - clusterProfiler
  - igraph
  - org.Hs.eg.db
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# chromosome-region-gene-subsetting

## Summary

Restricts genes eligible for differential expression simulation in the eduomics RNA workflow to those retained after GO-annotation similarity filtering and confinement to a single chromosome (typically chr22), producing selected_genes.txt as the ground-truth gene universe for downstream count matrix generation.

## When to use

Use when building an RNA differential expression simulation for a single chromosome and you need to define a biologically plausible, GO-connected gene universe (e.g., protein-coding genes on chr22 passing a Jaccard simthreshold of 0.3) before simulating counts.

## When NOT to use

- Input is already a curated gene list or expression matrix — this skill is for defining the gene universe before simulation, not for re-filtering existing DE results.
- The analysis goal is genome-wide rather than single-chromosome — the pipeline explicitly focuses on single chromosomes (typically chr22) and the subsetting logic assumes that scope.
- The task is DNA variant simulation or variant calling — the RNA gene-selection step does not apply to the DNA branch of eduomics.

## Inputs

- human gene-to-GO-term annotations via org.Hs.eg.db
- gene annotation with chromosome/coordinate information for protein-coding gene assignment
- simthreshold parameter from the samplesheet (e.g., 0.3)

## Outputs

- selected_genes.txt (one gene identifier per line, genes suitable for differential expression simulation)
- filtered gene-gene similarity network used to justify gene selection

## How to apply

Load human gene-to-GO-term annotations via org.Hs.eg.db and map all annotated genes to Entrez identifiers. Build a gene-gene similarity network with igraph in which genes are connected by shared GO terms, weighting each edge by the Jaccard index of the two genes' GO-term sets. Prune the network by removing edges whose Jaccard similarity falls below simthreshold = 0.3, then restrict the network nodes to protein-coding genes located on the target chromosome (chr22) using chromosome/coordinate information from the gene annotation. The genes remaining in the filtered network are the simulation universe; write them to selected_genes.txt, one gene identifier per line. The chromosome restriction mirrors the pipeline's key assumption of chromosome-specific analysis to reduce computational requirements while maintaining biological realism.

## Related tools

- **org.Hs.eg.db** (provides human gene-to-GO-term annotations mapped to Entrez identifiers as the basis for the similarity network)
- **igraph** (builds and prunes the gene-gene similarity network weighted by the Jaccard index of GO-term sets at simthreshold = 0.3)
- **clusterProfiler** (GO annotation tooling supporting enrichment-style gene selection logic)

## Examples

```
nextflow run eduomics -profile docker --input samplesheet.csv --genome GATK.GRCh38 --outdir <OUTDIR>
```

## Evaluation signals

- selected_genes.txt exists and is non-empty
- every gene listed in selected_genes.txt maps to chromosome 22 (or the specified target chromosome)
- all edges retained in the similarity network meet the simthreshold = 0.3 Jaccard cutoff
- selected_genes.txt contains one gene identifier per line with no malformed entries

## Limitations

- Restricting to a single chromosome (chr22) reduces the number of eligible genes, so simulation complexity is capped by the genes present on the target chromosome.
- The simthreshold default of 0.3 is a Jaccard cutoff on GO-term sets; genes with sparse GO annotations may be lost as isolated nodes after pruning.
- Only protein-coding genes are retained, so non-coding transcripts are excluded from the RNA simulation universe.

## Evidence

- [intro] Gene selection based on GO annotation: "Identifies genes suitable for differential expression based on Gene Ontology (GO) annotation."
- [readme] single-chromosome assumption: [evidence span withheld — anchor status: not_found]
- [readme] samplesheet simthreshold parameter: [evidence span withheld — anchor status: not_found]
