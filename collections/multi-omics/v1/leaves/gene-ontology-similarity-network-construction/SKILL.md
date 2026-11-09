---
name: gene-ontology-similarity-network-construction
description: 'Use when preparing the gene universe for RNA differential expression
  simulation: you need protein-coding genes on a target chromosome (e.g. chr22) that
  share sufficient GO-term content, with a simthreshold (default 0.3) defining edge
  retention.'
when_to_use_negative:
- Running the DNA simulation branch — variant selection and read simulation with SimuSCoP
  do not depend on GO-based gene selection
- You need the complete annotated transcriptome as the gene universe rather than a
  chromosome-restricted (chr22) protein-coding subset
- You are performing GO enrichment analysis on already-computed DE gene lists — that
  is downstream analysis, not gene-set pre-selection
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: org.Hs.eg.db
  role: Supplies human gene-to-GO-term annotations and maps genes to Entrez identifiers
- name: igraph
  role: Builds the gene-gene similarity network with Jaccard-weighted edges and prunes
    edges below simthreshold
- name: clusterProfiler
  role: GO annotation/enrichment support used alongside the gene-selection step in
    the RNA workflow
- name: eduomics
  role: Nextflow pipeline whose Gene Selection step implements this GO-similarity
    network construction (simthreshold column in samplesheet)
  repo: https://github.com/lescai-teaching/eduomics
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:94f454cbc3058d77b3b1f2b128ebc77ab7eb2e50207959754a5b08cf5fdd7ee6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lescai-teaching/eduomics
  tools:
  - clusterProfiler
  - eduomics
  - igraph
  - org.Hs.eg.db
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# gene-ontology-similarity-network-construction

## Summary

Constructs a Jaccard-index gene-gene similarity network from Gene Ontology annotations to identify a biologically plausible, chromosome-restricted set of genes for differential expression simulation in the eduomics RNA workflow.

## When to use

Use when preparing the gene universe for RNA differential expression simulation: you need protein-coding genes on a target chromosome (e.g. chr22) that share sufficient GO-term content, with a simthreshold (default 0.3) defining edge retention.

## When NOT to use

- Running the DNA simulation branch — variant selection and read simulation with SimuSCoP do not depend on GO-based gene selection
- You need the complete annotated transcriptome as the gene universe rather than a chromosome-restricted (chr22) protein-coding subset
- You are performing GO enrichment analysis on already-computed DE gene lists — that is downstream analysis, not gene-set pre-selection

## Inputs

- Human gene-to-GO-term annotations (org.Hs.eg.db, mapped to Entrez gene identifiers)
- Gene annotation with chromosome and coordinate information (from GATK.GRCh38 reference bundle)
- simthreshold parameter (e.g. 0.3 from the samplesheet's simthreshold column)

## Outputs

- selected_genes.txt (one gene Entrez identifier per line, suitable for DE simulation)
- Pruned GO-similarity graph restricted to protein-coding chr22 genes

## How to apply

Load human gene-to-GO-term annotations via org.Hs.eg.db and map all annotated genes to Entrez identifiers. Build a gene-gene graph in igraph connecting genes that share GO terms, weighting each edge by the Jaccard index of the two genes' GO-term sets, then prune edges whose Jaccard similarity falls below simthreshold = 0.3 (the value passed in the samplesheet). Restrict nodes to protein-coding genes on the target chromosome (chr22) using chromosome/coordinate information from the gene annotation. The genes remaining in the filtered network are written to selected_genes.txt, one Entrez identifier per line, as the input universe for count matrix generation (Polyester simulation, DESeq2 differential expression). The rationale is that genes retained by GO similarity form coherent functional groups, so injected differential expression patterns yield statistically coherent and biologically plausible enrichment.

## Related tools

- **org.Hs.eg.db** (Supplies human gene-to-GO-term annotations and maps genes to Entrez identifiers)
- **igraph** (Builds the gene-gene similarity network with Jaccard-weighted edges and prunes edges below simthreshold)
- **clusterProfiler** (GO annotation/enrichment support used alongside the gene-selection step in the RNA workflow)
- **eduomics** (Nextflow pipeline whose Gene Selection step implements this GO-similarity network construction (simthreshold column in samplesheet)) — https://github.com/lescai-teaching/eduomics

## Examples

```
nextflow run eduomics -profile docker --input samplesheet.csv --genome GATK.GRCh38 --outdir <OUTDIR>  # samplesheet row: rna_simulation_1,rna,chr22,30,,3,2,0.3
```

## Evaluation signals

- selected_genes.txt exists and is non-empty
- Every gene listed in selected_genes.txt maps to chromosome 22
- All retained network edges have a Jaccard similarity >= simthreshold = 0.3
- All selected genes are protein-coding and have Entrez identifiers resolvable via org.Hs.eg.db
- Downstream simulation produces statistically significant, biologically plausible GO enrichment when DESeq2 is run on the simulated counts

## Limitations

- The analysis is chromosome-specific (typically chr22) to reduce computational cost, so the selected gene set is not genome-wide
- Only protein-coding genes are retained; non-coding RNAs and other biotypes are excluded from simulation
- The Jaccard cutoff is fixed by the simthreshold parameter (default 0.3); stricter or looser thresholds change which functional groups survive network pruning

## Evidence

- [other] task_007 workflow: [evidence span withheld — anchor status: not_found]
- [intro] article intro: "Identifies genes suitable for differential expression based on Gene Ontology (GO) annotation."
- [readme] readme key assumptions: [evidence span withheld — anchor status: not_found]
- [readme] readme samplesheet: [evidence span withheld — anchor status: not_found]
