---
name: kegg-database-querying
description: Use when an interaction table or node list uses identifiers from a source
  database (e.g. BRENDA EC numbers like 2.7.1.1) that must be converted to KEGG reaction
  IDs (rn:RXXXXX) or KEGG gene IDs before integrating with KEGG pathway maps.
when_to_use_negative:
- When the input identifiers are already KEGG IDs or KEGG reaction IDs — no namespace
  conversion is needed and the pathway-layer import will consume them directly
- When the target pathway database is not KEGG (e.g. Reactome, Wikipathways, SBGN
  formats), which transomics2cytoscape does not yet support
- When mapping identifiers outside KEGG's supported namespace coverage, since only
  KEGG ID and pathway file format conversions are automated
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3172
tools:
- name: KEGGREST
  role: R client used to query KEGG — retrieve KGML pathway XML and link identifiers
    (e.g. EC numbers) to reaction IDs via keggLink/keggFind
  repo: https://github.com/ecell/transomics2cytoscape
- name: transomics2cytoscape
  role: Provides the ec2reaction conversion function that consumes KEGG query results
    so BRENDA metabolite-protein interactions attach to the same enzyme nodes as phosphorylation
    interactions in the 2.5D network
  repo: https://github.com/ecell/transomics2cytoscape
- name: R
  role: Language environment in which the read.delim/keggLink conversion workflow
    is executed
  repo: https://github.com/ecell/transomics2cytoscape
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41540-024-00342-8
    title: 'Transomics2cytoscape: an automated software for interpretable 2.5-dimensional
      visualization of trans-omic networks'
  source_content_hash: sha256:8379c9902a9017a0ede4006c3de0cb8592e78b7b5ccf60b6bc6a058284b766cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ecell/transomics2cytoscape
  tools:
  - KEGGREST
  - R
  - transomics2cytoscape
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3172
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# kegg-database-querying

## Summary

Queries the KEGG database (via KEGGREST) to map molecule identifiers — e.g. EC numbers in a trans-omic interaction table — into the KEGG namespace (reaction IDs such as rn:RXXXXX, gene IDs, KGML pathway XML) so that heterogeneous interaction layers can attach to the same nodes in a KEGG pathway map.

## When to use

Use when an interaction table or node list uses identifiers from a source database (e.g. BRENDA EC numbers like 2.7.1.1) that must be converted to KEGG reaction IDs (rn:RXXXXX) or KEGG gene IDs before integrating with KEGG pathway maps.

## When NOT to use

- When the input identifiers are already KEGG IDs or KEGG reaction IDs — no namespace conversion is needed and the pathway-layer import will consume them directly
- When the target pathway database is not KEGG (e.g. Reactome, Wikipathways, SBGN formats), which transomics2cytoscape does not yet support
- When mapping identifiers outside KEGG's supported namespace coverage, since only KEGG ID and pathway file format conversions are automated

## Inputs

- Tab-separated trans-omic interaction TSV with an EC-number column (e.g. column 6 containing values like 2.7.1.1, possibly prefixed 'ec:')
- Identifier strings to be mapped (EC numbers, gene symbols/IDs) requiring conversion into KEGG namespaces

## Outputs

- allosteric_reactions.tsv: tab-separated UTF-8 table with a new KEGG reaction-ID column (rn:R followed by five digits) alongside the original EC column, same row count as input
- KEGG reaction ID(s) per EC number, including concatenated one-to-many mappings

## How to apply

Extract the identifier column to convert — e.g. column 6 of an allosteric metabolite-protein interaction TSV, which holds EC numbers such as 2.7.1.1 — and strip any 'ec:' prefix. Use KEGGREST functions (keggLink/keggFind) in R to map each identifier into the target namespace, here 'rn' reaction IDs, handling one-to-many mappings by concatenating all returned reaction IDs. Rebuild the table with the new reaction-ID column added alongside the original identifier column, leaving all other columns unchanged, and write it as tab-separated UTF-8 (e.g. allosteric_reactions.tsv) preserving the input row count. The rationale is that transomics2cytoscape attaches both BRENDA-style metabolite-protein edges and enzyme phosphorylation edges to the same metabolic enzyme nodes in the KEGG global metabolic pathway map, which requires all edges to share KEGG-compatible IDs; the package itself uses KEGGREST to retrieve KGML (KEGG pathway XML) for pathway layers.

## Related tools

- **KEGGREST** (R client used to query KEGG — retrieve KGML pathway XML and link identifiers (e.g. EC numbers) to reaction IDs via keggLink/keggFind) — https://github.com/ecell/transomics2cytoscape
- **transomics2cytoscape** (Provides the ec2reaction conversion function that consumes KEGG query results so BRENDA metabolite-protein interactions attach to the same enzyme nodes as phosphorylation interactions in the 2.5D network) — https://github.com/ecell/transomics2cytoscape
- **R** (Language environment in which the read.delim/keggLink conversion workflow is executed) — https://github.com/ecell/transomics2cytoscape

## Examples

```
Rscript -e 'library(KEGGREST); rn <- keggLink("reaction", "ec:2.7.1.1"); cat(rn, "\n")'
```

## Evaluation signals

- allosteric_reactions.tsv exists and has exactly the same number of rows as the input interaction TSV
- Every row contains a reaction-ID field matching the KEGG reaction-ID format 'rn:R' followed by five digits (e.g. rn:R00756)
- The original EC-number column and all other columns are preserved unchanged relative to the input TSV
- One-to-many EC-to-reaction mappings yield concatenated reaction IDs rather than dropped or truncated rows
- Round-trip spot-check: keggLink/keggFind on a sampled EC number returns the reaction ID recorded in the output file

## Limitations

- One-to-many mappings (a single EC number maps to multiple rn:R reaction IDs) must be handled by concatenation, which complicates downstream edge attachment
- Currently, transomics2cytoscape supports only the KEGG ID and pathway file format; Reactome, Wikipathways, and SBGN are not supported
- Retrieval of inter-layer edges from source databases such as BRENDA is not automated — the interaction table must be prepared manually before the ID conversion

## Evidence

- [discussion] Transomics2cytoscape uses KEGGREST to retrieve KGML (KEGG pathway XML).: "Transomics2cytoscape uses KEGGREST to retrieve KGML (KEGG pathway XML)."
- [discussion] IDs converted into KEGG-compatible IDs: "The IDs of molecules were converted into KEGG–compatible IDs such as KEGG gene ID and EC numbers by using Ensembl BioMart42"
- [discussion] Only KEGG format supported: "Currently, transomics2cytoscape supports only the KEGG ID and pathway file format."
- [discussion] Interactions obtainable from BRENDA: "Interactions between omic layers can also be obtained from databases such as BRENDA as well as pathway data from KEGG."
- [readme] Package installation path: [evidence span withheld — anchor status: external_doc]
