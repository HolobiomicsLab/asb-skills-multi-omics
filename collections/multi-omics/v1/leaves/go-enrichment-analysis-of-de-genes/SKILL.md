---
name: go-enrichment-analysis-of-de-genes
description: Use when a differential expression table (e.g., deseq2_de_genes.txt from
  DESeq2 on simulated RNA-seq counts) must be functionally characterized and its GO
  term enrichment validated for biological plausibility.
when_to_use_negative:
- Input is a DNA variant simulation branch output — the GO enrichment step belongs
  only to the RNA differential expression branch
- No statistically significant/coherent DE genes were detected (e.g., empty deseq2_de_genes.txt),
  so enrichment has no meaningful gene set to test
- The gene set is real experimental RNA-seq where ground truth is unknown and educational
  validation against known DE patterns is not the goal
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: DESeq2
  role: Upstream step that identifies the differentially expressed genes fed into
    GO enrichment
  repo: https://github.com/lescai-teaching/eduomics
- name: Salmon
  role: Quantifies transcript expression from simulated FASTQ, providing the expression
    estimates DESeq2 uses to call DE genes prior to enrichment
  repo: https://github.com/lescai-teaching/eduomics
- name: Polyester
  role: Generates the simulated FASTQ files matching the count matrices that originate
    the DE gene list
  repo: https://github.com/lescai-teaching/eduomics
- name: Nextflow
  role: Orchestrates the RNA simulation branch including the enrichment and validation
    modules
  repo: https://github.com/lescai-teaching/eduomics
- name: Docker
  role: Container system used to run the pipeline reproducibly (e.g., -profile docker)
  repo: https://github.com/lescai-teaching/eduomics
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1101/2025.11.24.690117
    title: 'Eduomics: a Nextflow pipeline to simulate -omics data for education'
  source_content_hash: sha256:4ff7efdf7282281e43a115e0f08b52837fd2f3e2b497f5598f2de72f7984f19a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lescai-teaching/eduomics
  tools:
  - DESeq2
  - Docker
  - FastQC
  - MultiQC
  - Nextflow
  - Polyester
  - Salmon
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# GO-enrichment-analysis-of-DE-genes

## Summary

Conducts Gene Ontology (GO) over-representation enrichment analysis on differentially expressed (DE) genes identified by DESeq2, producing enrichment_results.rds plus dotplot and cnetplot PNGs, and verifying that the enriched terms are biologically plausible for teaching simulations.

## When to use

Use when a differential expression table (e.g., deseq2_de_genes.txt from DESeq2 on simulated RNA-seq counts) must be functionally characterized and its GO term enrichment validated for biological plausibility.

## When NOT to use

- Input is a DNA variant simulation branch output — the GO enrichment step belongs only to the RNA differential expression branch
- No statistically significant/coherent DE genes were detected (e.g., empty deseq2_de_genes.txt), so enrichment has no meaningful gene set to test
- The gene set is real experimental RNA-seq where ground truth is unknown and educational validation against known DE patterns is not the goal

## Inputs

- deseq2_de_genes.txt (list of differentially expressed genes from DESeq2)
- deseq2_results.tsv (full DESeq2 differential expression results)
- Gene Ontology (GO) annotation of genes selected for differential expression
- Simulated RNA-seq count matrices with known differential expression patterns

## Outputs

- enrichment_results.rds
- dotplot PNG of enriched GO terms
- cnetplot PNG of enriched GO terms
- validation_result.txt (from the validation module)

## How to apply

After DESeq2 identifies differentially expressed genes (deseq2_results.tsv and deseq2_de_genes.txt) from Salmon-quantified, Polyester-simulated RNA-seq data, run GO enrichment analysis on the DE gene list using Gene Ontology (GO) annotations as the reference. The pipeline saves the enrichment object as enrichment_results.rds and visualizes it as dotplot and cnetplot PNGs. In the eduomics validation module, the enrichment must yield detectable and biologically plausible functional enrichment — enriched terms should be coherent with the GO annotation used during gene selection for differential expression. Simulations failing this plausibility check (or the DE significance check) are excluded from the final rna_simulations output directory, where only simulations passing validation are retained alongside the RNA reference bundle.

## Related tools

- **DESeq2** (Upstream step that identifies the differentially expressed genes fed into GO enrichment) — https://github.com/lescai-teaching/eduomics
- **Salmon** (Quantifies transcript expression from simulated FASTQ, providing the expression estimates DESeq2 uses to call DE genes prior to enrichment) — https://github.com/lescai-teaching/eduomics
- **Polyester** (Generates the simulated FASTQ files matching the count matrices that originate the DE gene list) — https://github.com/lescai-teaching/eduomics
- **Nextflow** (Orchestrates the RNA simulation branch including the enrichment and validation modules) — https://github.com/lescai-teaching/eduomics
- **Docker** (Container system used to run the pipeline reproducibly (e.g., -profile docker)) — https://github.com/lescai-teaching/eduomics

## Examples

```
nextflow run eduomics -profile docker --input samplesheet.csv --genome GATK.GRCh38 --outdir <OUTDIR>
```

## Evaluation signals

- enrichment_results.rds exists alongside dotplot and cnetplot PNG files in the simulation output
- validation_result.txt reports that the simulated data produce detectable and biologically plausible functional enrichment
- Enriched GO terms are coherent with the Gene Ontology annotation used to select genes suitable for differential expression
- The simulation is retained in the final rna_simulations directory (only simulations passing validation are included in the final output)
- The DE gene list (deseq2_de_genes.txt) is non-empty and statistically significant, since enrichment is only meaningful over detected DE genes

## Limitations

- The pipeline focuses on single chromosomes (typically chr22) to reduce computational requirements, so the gene universe and enrichment background are limited to that subset
- Enrichment plausibility depends on the known differential expression pattern injected into the count matrices; it is only validated for simulations designed with such ground truth
- Only simulations passing validation are included in the final output, so a failed enrichment check silently drops the simulation from rna_simulations

## Evidence

- [intro] Conducts GO enrichment analysis on DE genes.: "Conducts GO enrichment analysis on DE genes."
- [intro] Only simulations passing validation are included in final output: "Only simulations passing validation are included in the final output"
- [intro] Validation module verifies biologically plausible functional enrichment: "Verifies that the simulated data produce a statistically significant and coherent differential expression pattern, with detectable and biologically plausible functional enrichment."
- [readme] Pipeline validates simulated data quality: [evidence span withheld — anchor status: not_found]
- [readme] Pipeline run command with genome and samplesheet parameters: "nextflow run eduomics \
   -profile <docker/singularity/.../institute> \
   --input samplesheet.csv \
   --genome GATK.GRCh38"
