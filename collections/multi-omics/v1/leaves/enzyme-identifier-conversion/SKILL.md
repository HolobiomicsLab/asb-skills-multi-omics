---
name: enzyme-identifier-conversion
description: Use when an inter-layer interaction TSV carries EC numbers (e.g. 2.7.1.1
  from BRENDA) but the target KEGG pathway layer indexes enzymes by KEGG reaction
  IDs, causing an ID namespace mismatch that must be bridged before visualization.
when_to_use_negative:
- The interaction table already contains KEGG reaction IDs (rn:Rxxxxx) — no conversion
  is needed
- The target pathway layer is not KEGG (transomics2cytoscape currently supports only
  KEGG ID and pathway file format; Reactome, Wikipathways, SBGN are not supported)
- The identifiers to convert are gene/protein IDs rather than EC numbers — the article's
  related workflow uses Ensembl BioMart to convert molecule IDs into KEGG gene IDs
  instead
edam_operation: http://edamontology.org/operation_0250
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0605
tools:
- name: transomics2cytoscape
  role: Provides the ec2reaction function that performs the EC-number to KEGG reaction-ID
    conversion on trans-omic interaction tables
  repo: https://github.com/ecell/transomics2cytoscape
- name: KEGGREST
  role: R client used via keggLink/keggFind to map EC numbers to KEGG reaction IDs
    in the 'rn' namespace
- name: R
  role: 'Implementation language: read.delim on the input TSV and write the converted
    allosteric_reactions.tsv'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41540-024-00342-8
    title: 'Transomics2cytoscape: an automated software for interpretable 2.5-dimensional
      visualization of trans-omic networks'
  source_content_hash: sha256:6769a6d8e84a03f5f2fd3837b5d1f99a01a676f804cc3e10c27718d3612389b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ecell/transomics2cytoscape
  tools:
  - KEGGREST
  - R
  - transomics2cytoscape
  edam_operation: http://edamontology.org/operation_0250
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# enzyme-identifier-conversion

## Summary

Converts EC-number annotations in a trans-omic interaction table (e.g. BRENDA-derived metabolite-protein interactions) into KEGG reaction IDs ('rn' namespace) so that heterogeneous inter-layer edges can attach to enzyme nodes in the KEGG global metabolic pathway map used by transomics2cytoscape.

## When to use

Use when an inter-layer interaction TSV carries EC numbers (e.g. 2.7.1.1 from BRENDA) but the target KEGG pathway layer indexes enzymes by KEGG reaction IDs, causing an ID namespace mismatch that must be bridged before visualization.

## When NOT to use

- The interaction table already contains KEGG reaction IDs (rn:Rxxxxx) — no conversion is needed
- The target pathway layer is not KEGG (transomics2cytoscape currently supports only KEGG ID and pathway file format; Reactome, Wikipathways, SBGN are not supported)
- The identifiers to convert are gene/protein IDs rather than EC numbers — the article's related workflow uses Ensembl BioMart to convert molecule IDs into KEGG gene IDs instead

## Inputs

- Allosteric metabolite-protein interaction TSV with an EC-number column (e.g. column 6, values like 2.7.1.1)
- KEGG REST API (via KEGGREST R package) as the EC-to-reaction reference mapping

## Outputs

- allosteric_reactions.tsv: tab-separated, UTF-8, same row count as input, with an added KEGG reaction-ID column alongside the original EC column

## How to apply

Load the interaction TSV in R with read.delim, preserving column order so that the EC-number column (e.g. column 6 of the allosteric metabolite-protein interaction table) is not reordered. Extract unique EC strings, strip any 'ec:' prefix, and query KEGGREST (keggLink/keggFind) to map each EC number to KEGG reaction ID(s) in the 'rn' namespace, concatenating IDs when the mapping is one-to-many. Rebuild the table with a new reaction-ID column alongside the original EC column, leaving all other columns unchanged, and write it out tab-separated (UTF-8) with the same row count. The rationale is that transomics2cytoscape supports only KEGG IDs and pathway file format, so BRENDA-style edges must be expressed in KEGG reaction space to attach to the same metabolic enzyme nodes as enzyme phosphorylation edges.

## Related tools

- **transomics2cytoscape** (Provides the ec2reaction function that performs the EC-number to KEGG reaction-ID conversion on trans-omic interaction tables) — https://github.com/ecell/transomics2cytoscape
- **KEGGREST** (R client used via keggLink/keggFind to map EC numbers to KEGG reaction IDs in the 'rn' namespace)
- **R** (Implementation language: read.delim on the input TSV and write the converted allosteric_reactions.tsv)

## Examples

```
library(KEGGREST); ecs <- sub('^ec:', '', t[,6]); rxns <- keggLink('reaction', paste0('ec:', unique(ecs)))
```

## Evaluation signals

- allosteric_reactions.tsv exists and its row count equals the row count of the input interaction TSV
- The output contains a reaction-ID field and all other original columns are unchanged
- Every converted reaction ID matches the KEGG reaction-ID format (rn:R followed by five digits)
- One-to-many EC-to-reaction mappings are handled by concatenating all mapped reaction IDs rather than dropping rows
- Downstream, both BRENDA-style metabolite-protein edges and enzyme phosphorylation edges attach to the same metabolic enzyme nodes in the KEGG global metabolic pathway map

## Limitations

- transomics2cytoscape currently supports only the KEGG ID and pathway file format, so converted IDs must land in KEGG namespaces
- Retrieval of inter-layer edges from databases such as BRENDA is not automated and must be prepared as input tables manually
- EC-to-reaction mappings can be one-to-many, so a single input row may correspond to multiple concatenated reaction IDs

## Evidence

- [other] transomics2cytoscape provides the ec2reaction function, which converts the EC-number column of a trans-omic interaction table into KEGG reaction IDs: [evidence span withheld — anchor status: not_found]
- [discussion] The IDs of molecules were converted into KEGG–compatible IDs such as KEGG gene ID and EC numbers by using Ensembl BioMart42: "The IDs of molecules were converted into KEGG–compatible IDs such as KEGG gene ID and EC numbers by using Ensembl BioMart42"
- [discussion] Transomics2cytoscape uses KEGGREST to retrieve KGML (KEGG pathway XML).: "Transomics2cytoscape uses KEGGREST to retrieve KGML (KEGG pathway XML)."
- [discussion] Currently, transomics2cytoscape supports only the KEGG ID and pathway file format.: "Currently, transomics2cytoscape supports only the KEGG ID and pathway file format."
