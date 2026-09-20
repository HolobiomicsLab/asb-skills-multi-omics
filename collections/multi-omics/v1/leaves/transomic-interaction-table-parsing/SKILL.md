---
name: transomic-interaction-table-parsing
description: Use when a tabular file specifies trans-omic interactions between stacked
  pathway layers and you must convert each row into a resolved source node, target
  node, and interaction type for network edge creation.
when_to_use_negative:
- The network is a single 2D pathway layer with no stacked omic layers, so there are
  no inter-layer edges to specify
- Molecule IDs have not yet been converted into KEGG-compatible IDs (transomics2cytoscape
  currently supports only the KEGG ID format)
- You need transomics2cytoscape to fetch inter-layer interactions from databases such
  as BRENDA, OmniPath, SIGNOR, or IntAct — that retrieval is not automated and must
  be supplied as the table yourself
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0605
tools:
- name: transomics2cytoscape
  role: R Bioconductor package whose trans-omic interaction file contract this parsing
    implements; adds edges between stacked pathway layers
  repo: https://github.com/ecell/transomics2cytoscape
- name: RCy3
  role: R interface used to query node tables and add inter-layer edges in Cytoscape
    via Cytoscape Automation
- name: Cytoscape
  role: Hosts the layered 2.5D network whose node tables are matched against attribute
    values to resolve edge endpoints
  repo: https://github.com/cytoscape/cytoscape
- name: R
  role: Language in which the interaction table is loaded and per-row node resolution
    is scripted
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41540-024-00342-8
    title: 'Transomics2cytoscape: an automated software for interpretable 2.5-dimensional
      visualization of trans-omic networks'
  source_content_hash: sha256:41796cc8ceeaa0b445bf711a656533f6bf35a2cc7783c323fa9c2dd9f3d53f92
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ecell/transomics2cytoscape
  tools:
  - Cytoscape
  - R
  - RCy3
  - transomics2cytoscape
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# transomic-interaction-table-parsing

## Summary

Parses the 7-column trans-omic interaction TSV used by transomics2cytoscape to define inter-layer edges, resolving each row's source and target nodes by (layer index, node-table column name, attribute value) and extracting the interaction type for the Cytoscape edge table.

## When to use

Use when a tabular file specifies trans-omic interactions between stacked pathway layers and you must convert each row into a resolved source node, target node, and interaction type for network edge creation.

## When NOT to use

- The network is a single 2D pathway layer with no stacked omic layers, so there are no inter-layer edges to specify
- Molecule IDs have not yet been converted into KEGG-compatible IDs (transomics2cytoscape currently supports only the KEGG ID format)
- You need transomics2cytoscape to fetch inter-layer interactions from databases such as BRENDA, OmniPath, SIGNOR, or IntAct — that retrieval is not automated and must be supplied as the table yourself

## Inputs

- 7-column trans-omic interaction TSV (source layer index, source node-table column name, source attribute value, target layer index, target node-table column name, target attribute value, interaction type)
- Layered 2.5D network loaded in Cytoscape (node tables carrying KEGG-compatible IDs such as KEGG gene IDs and EC numbers)

## Outputs

- Inter-layer edges in the Cytoscape network connecting nodes across stacked layers, one per interaction-table row
- Edge table with the 'interaction' column populated from the TSV's seventh column
- Validated Cytoscape session file (e.g. transomic_edges.cys)

## How to apply

Read the trans-omic interaction file as a TSV with exactly seven columns: columns 1–3 identify the source node (layer index, node-table column name, attribute value), columns 4–6 identify the target node the same way, and column 7 gives the interaction type. For each row, load the layered 2.5D network in Cytoscape via RCy3, look up the node in the specified layer whose node-table attribute in the named column matches the given attribute value (e.g. a KEGG gene ID or EC number, since transomics2cytoscape expects KEGG-compatible IDs), and repeat for the target. Add the inter-layer edge between the resolved nodes using RCy3 network-editing commands, writing the seventh-column value into the edge table's 'interaction' column; insert edge-midpoint nodes where the interaction requires them. Validate that every input row yields exactly one correctly typed edge before saving the session.

## Related tools

- **transomics2cytoscape** (R Bioconductor package whose trans-omic interaction file contract this parsing implements; adds edges between stacked pathway layers) — https://github.com/ecell/transomics2cytoscape
- **RCy3** (R interface used to query node tables and add inter-layer edges in Cytoscape via Cytoscape Automation)
- **Cytoscape** (Hosts the layered 2.5D network whose node tables are matched against attribute values to resolve edge endpoints) — https://github.com/cytoscape/cytoscape
- **R** (Language in which the interaction table is loaded and per-row node resolution is scripted)

## Examples

```
Rscript -e 'txi <- read.delim("transomic_interactions.tsv", header=FALSE); createTransomicEdges(network.suid, txi)'
```

## Evaluation signals

- Every row of the trans-omic interaction TSV yields exactly one edge in the resulting Cytoscape network (edge count equals row count)
- Each created edge has the correct interaction type copied from the TSV's seventh column into the edge table's 'interaction' column
- Each edge's endpoints match the nodes resolved by (layer index, node-table column, attribute value) for both source and target
- Node resolution succeeds only when the specified node-table attribute matches the KEGG-compatible ID (KEGG gene ID or EC number) within the specified layer
- Session saves cleanly as transomic_edges.cys with no unresolved or orphaned inter-layer edges

## Limitations

- Currently only KEGG IDs and the KEGG pathway file format are supported; Reactome, Wikipathways, and SBGN are not
- Retrieval of inter-layer edges from databases (BRENDA, OmniPath, SIGNOR, IntAct) is not automated — the interaction table must be prepared externally
- Connectivity data of trans-omic networks are provided after reconstruction procedures that are not fully automated

## Evidence

- [full_text] The trans-omic interaction table defines edges connecting layers: its first three columns specify the source node (layer index, node-table column name, attribute value), the fourth to sixth columns specify the target node the same way: [evidence span withheld — anchor status: not_found]
- [discussion] The transomics2cytoscape package adds edges between layers: "The transomics2cytoscape package adds edges between layers to understand molecular interactions between pathway layers"
- [discussion] The IDs of molecules were converted into KEGG-compatible IDs: "The IDs of molecules were converted into KEGG–compatible IDs such as KEGG gene ID and EC numbers by using Ensembl BioMart42"
- [discussion] Only KEGG ID and pathway file format are supported: "Currently, transomics2cytoscape supports only the KEGG ID and pathway file format."
- [discussion] Inter-layer edge retrieval from databases is not automated: "transomics2cytoscape has not yet automated the retrieval processes for inter-layer edges."
