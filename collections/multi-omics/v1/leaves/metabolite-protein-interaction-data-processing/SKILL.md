---
name: metabolite-protein-interaction-data-processing
description: Use when a trans-omic interaction table (e.g. BRENDA allosteric metabolite-protein
  TSV) carries EC numbers in one column but the target pathway layer (KEGG rn map)
  requires reaction IDs, blocking cross-layer edge attachment.
when_to_use_negative:
- The interaction table already uses KEGG reaction IDs or KEGG-compatible IDs (transomics2cytoscape
  currently supports only the KEGG ID namespace, so no conversion is needed)
- The pathway layers are not KEGG-based (e.g. Reactome, Wikipathways, or SBGN layouts),
  since the package does not yet support those ID systems or file formats
- The input table's EC numbers are not in a single well-defined column (e.g. column
  6) — column mapping must be verified before conversion or IDs will be silently misassigned
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3172
tools:
- name: transomics2cytoscape
  role: R Bioconductor package whose ec2reaction conversion function this skill reconstructs;
    it consumes the converted reaction-ID table to attach inter-layer edges to KEGG
    pathway layers
  repo: https://github.com/ecell/transomics2cytoscape
- name: KEGGREST
  role: Bioconductor package used to query KEGG (keggLink/keggFind) and map each EC
    number to KEGG reaction IDs in the 'rn' namespace
- name: R
  role: Execution environment for read.delim, the EC-to-reaction conversion script,
    and writing the UTF-8 tab-separated output
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41540-024-00342-8
    title: 'Transomics2cytoscape: an automated software for interpretable 2.5-dimensional
      visualization of trans-omic networks'
  source_content_hash: sha256:32054481627621d2310bb0c1c1b7c970cb55eac06fdbe3dfa503b000de435f7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ecell/transomics2cytoscape
  tools:
  - KEGGREST
  - R
  - transomics2cytoscape
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# metabolite-protein-interaction-data-processing

## Summary

Reconstructs transomics2cytoscape's ec2reaction conversion, which maps the EC numbers (e.g. 2.7.1.1) used by BRENDA-derived metabolite-protein interactions to KEGG reaction IDs (rn:R#####) so that allosteric edges and enzyme phosphorylation edges share one ID namespace with the KEGG global metabolic pathway map.

## When to use

Use when a trans-omic interaction table (e.g. BRENDA allosteric metabolite-protein TSV) carries EC numbers in one column but the target pathway layer (KEGG rn map) requires reaction IDs, blocking cross-layer edge attachment.

## When NOT to use

- The interaction table already uses KEGG reaction IDs or KEGG-compatible IDs (transomics2cytoscape currently supports only the KEGG ID namespace, so no conversion is needed)
- The pathway layers are not KEGG-based (e.g. Reactome, Wikipathways, or SBGN layouts), since the package does not yet support those ID systems or file formats
- The input table's EC numbers are not in a single well-defined column (e.g. column 6) — column mapping must be verified before conversion or IDs will be silently misassigned

## Inputs

- Allosteric metabolite-protein interaction TSV (tab-separated) with EC numbers in column 6 (e.g. 2.7.1.1)
- KEGGREST KEGG database access for EC-to-reaction linkage

## Outputs

- allosteric_reactions.tsv (tab-separated, UTF-8) with a new KEGG reaction-ID column alongside the original EC column, same row count as input

## How to apply

Load the allosteric interaction TSV in R with read.delim, preserving column order so that column 6 holds the EC numbers. Extract unique EC strings, strip any 'ec:' prefix, and query KEGGREST (keggLink/keggFind) to map each EC number to one or more KEGG reaction IDs in the 'rn' namespace, concatenating reaction IDs when a mapping is one-to-many. Rebuild the table with a new reaction-ID column alongside the original EC column, leaving all other columns untouched, and write it as allosteric_reactions.tsv (tab-separated, UTF-8). The rationale is that transomics2cytoscape consumes only KEGG-compatible IDs, so unifying EC and reaction namespaces is what lets BRENDA-style metabolite-protein edges attach to the same metabolic enzyme nodes as phosphorylation edges in the 2.5D network.

## Related tools

- **transomics2cytoscape** (R Bioconductor package whose ec2reaction conversion function this skill reconstructs; it consumes the converted reaction-ID table to attach inter-layer edges to KEGG pathway layers) — https://github.com/ecell/transomics2cytoscape
- **KEGGREST** (Bioconductor package used to query KEGG (keggLink/keggFind) and map each EC number to KEGG reaction IDs in the 'rn' namespace)
- **R** (Execution environment for read.delim, the EC-to-reaction conversion script, and writing the UTF-8 tab-separated output)

## Examples

```
library(KEGGREST); rn_ids <- keggLink("reaction", sub("^ec:", "", ecs))  # ecs = unique EC strings from column 6
```

## Evaluation signals

- allosteric_reactions.tsv exists and has the same number of rows as the input interaction TSV
- The output contains a reaction-ID field populated for the converted rows
- Every converted identifier matches the KEGG reaction-ID format (rn:R followed by five digits)
- Reaction IDs trace back to KEGG via KEGGREST linkage for the original EC numbers, with one-to-many EC mappings represented by concatenated reaction IDs
- All non-EC columns of the input TSV are preserved unchanged in the output

## Limitations

- EC-to-reaction mappings can be one-to-many, so a single interaction row may expand to concatenated reaction IDs; downstream consumers must handle this
- transomics2cytoscape currently supports only the KEGG ID and pathway file format; Reactome, Wikipathways, and SBGN ID namespaces are not supported
- Retrieval of inter-layer edges from databases such as BRENDA is not automated by the package — the input interaction TSV must be prepared manually before conversion

## Evidence

- [discussion] Interactions between omic layers can also be obtained from databases such as BRENDA as well as pathway data from KEGG.: "Interactions between omic layers can also be obtained from databases such as BRENDA as well as pathway data from KEGG."
- [discussion] Currently, transomics2cytoscape supports only the KEGG ID and pathway file format.: "Currently, transomics2cytoscape supports only the KEGG ID and pathway file format."
- [discussion] Transomics2cytoscape uses KEGGREST to retrieve KGML (KEGG pathway XML).: "Transomics2cytoscape uses KEGGREST to retrieve KGML (KEGG pathway XML)."
- [readme] Here we propose transomics2cytoscape, an R package that automatically creates 3D network visualization in combination with Cytoscape: "Here we propose **transomics2cytoscape**, an R package that automatically creates
3D network visualization in combination with
Cytoscape"
