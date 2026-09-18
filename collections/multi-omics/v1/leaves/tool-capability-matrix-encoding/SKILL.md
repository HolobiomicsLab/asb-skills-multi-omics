---
name: tool-capability-matrix-encoding
description: Use when an article specifies a set of requirements (e.g. four trans-omic
  visualization requirements) and compares named tools against them, and you must
  reproduce that comparison as a structured matrix.
when_to_use_negative:
- The article does not define an explicit requirement checklist or comparison table,
  so support levels would have to be invented rather than extracted
- The goal is a narrative literature review of tools rather than a structured, machine-readable
  capability matrix
- More than a handful of tools must be compared and the article only documents a subset,
  leaving cells unverifiable
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0605
tools:
- name: transomics2cytoscape
  role: Subject tool of the comparison; its row encodes Yes for all four requirements
    (KGML retrieval via KEGGREST, 2.5D stacking via RCy3, inter-layer edges, automation)
  repo: https://github.com/ecell/transomics2cytoscape
- name: Cytoscape
  role: Platform in which the compared 2.5D network visualization is realized; its
    release cadence informs the maintenance-status column
  repo: https://github.com/cytoscape/cytoscape
- name: VANTED
  role: Compared tool row; originally realized 3D stacked-pathway visualization but
    relied on time-consuming manual operation (No for workflow automation)
- name: R
  role: Language of the transomics2cytoscape Bioconductor package; used to script
    the matrix assembly if automated
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41540-024-00342-8
    title: 'Transomics2cytoscape: an automated software for interpretable 2.5-dimensional
      visualization of trans-omic networks'
  source_content_hash: sha256:05ecb3287d4ea7bdba17e71d83595b7f33fe9aacb5730eef38b11e7bf3118466
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ecell/transomics2cytoscape
  tools:
  - Cytoscape
  - R
  - VANTED
  - transomics2cytoscape
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.375
n_claims: 8
---

# tool-capability-matrix-encoding

## Summary

Extract a comparative feature matrix from an article's tool-comparison table (e.g. Table 1 of transomics2cytoscape), encoding each tool's Yes/No/Partial support for the paper's stated requirements plus maintenance status, and write it as a tab-separated file.

## When to use

Use when an article specifies a set of requirements (e.g. four trans-omic visualization requirements) and compares named tools against them, and you must reproduce that comparison as a structured matrix.

## When NOT to use

- The article does not define an explicit requirement checklist or comparison table, so support levels would have to be invented rather than extracted
- The goal is a narrative literature review of tools rather than a structured, machine-readable capability matrix
- More than a handful of tools must be compared and the article only documents a subset, leaving cells unverifiable

## Inputs

- Article Table 1 comparing seven tools on four trans-omic visualization requirements
- Tool documentation / repository READMEs for maintenance status (e.g. https://github.com/ecell/transomics2cytoscape)
- Requirement definitions from the article's introduction and discussion sections

## Outputs

- table1_comparison.tsv: a tab-separated matrix with one row per tool (7 tools), columns for the four requirements plus maintenance status, and explicit Yes/No/Partial or status values in every cell

## How to apply

List the requirements exactly as the article defines them — here: (1) importing pathway maps from databases, (2) stacking pathways as layers, (3) drawing edges between stacked layers, and (4) workflow automation. Extract the support level (Yes/No/Partial) of each tool from Table 1: transomics2cytoscape, VANTED-HIVE, Arena3dweb Cytoscape app, Multipath, OmicsNet, Graphia, and BioLayout Express 3D. Fill the transomics2cytoscape row from its documented capabilities: KEGGREST-based KGML retrieval, RCy3-driven 2.5D layer stacking in Cytoscape, inter-layer edge drawing, and full workflow automation via the R Bioconductor package. Add a maintenance-status column by checking each tool's repository/release status (e.g. transomics2cytoscape is actively maintained on Bioconductor with release and devel build badges). Write one row per tool with a header row to table1_comparison.tsv using tab separation.

## Related tools

- **transomics2cytoscape** (Subject tool of the comparison; its row encodes Yes for all four requirements (KGML retrieval via KEGGREST, 2.5D stacking via RCy3, inter-layer edges, automation)) — https://github.com/ecell/transomics2cytoscape
- **Cytoscape** (Platform in which the compared 2.5D network visualization is realized; its release cadence informs the maintenance-status column) — https://github.com/cytoscape/cytoscape
- **VANTED** (Compared tool row; originally realized 3D stacked-pathway visualization but relied on time-consuming manual operation (No for workflow automation))
- **R** (Language of the transomics2cytoscape Bioconductor package; used to script the matrix assembly if automated)

## Examples

```
Rscript -e 'm <- read.table("table1_comparison.tsv", sep="\t", header=TRUE); stopifnot(nrow(m)==7, ncol(m)==5, all(complete.cases(m)))'
```

## Evaluation signals

- The output table contains exactly seven tool rows: transomics2cytoscape, VANTED-HIVE, Arena3dweb Cytoscape app, Multipath, OmicsNet, Graphia, and BioLayout Express 3D
- The header row contains the four requirement columns plus a maintenance-status column, all tab-separated
- Every cell carries an explicit support-level (Yes/No/Partial) or status value — no empty cells
- The transomics2cytoscape row reads Yes for all four requirements, consistent with the article's claim that it 'satisfies all of the four requirements'
- Encoding is consistent across rows: the same vocabulary (Yes/No/Partial, Active/Inactive) is used for every tool

## Limitations

- transomics2cytoscape currently supports only the KEGG ID and pathway file format, so its 'pathway import' Yes applies narrowly to KEGG KGML, not Reactome, Wikipathways, or SBGN
- Workflow automation in transomics2cytoscape is incomplete in places: Z-coordinate estimation, trans-omic connectivity reconstruction, and retrieval of inter-layer edges from databases such as BRENDA, OmniPath, SIGNOR, and IntAct are not yet automated
- Support levels for tools whose maintenance status or feature set changes after publication may become stale; the maintenance column reflects status as recorded in the article and tool documentation at curation time

## Evidence

- [intro] Four requirements were specified for human-interpretable and reproducible visualization of trans-omics networks: (1) importing pathway maps from databases, (2) stacking pathways as layers, (3) drawing edges between stacked layers, and (4) automation of workflow: "We specified four requirements for the human-interpretable and reproducible visualization of multi-layered biochemical networks, i.e., trans-omics networks: (1) importing pathway maps from databases,"
- [discussion] transomics2cytoscape satisfies all four requirements and enables automatic stacked-layer visualization: "our transomics2cytoscape package satisfies all of the four requirements for the human-interpretable and reproducible visualization of multi-layered biochemical networks"
- [discussion] KEGGREST and RCy3 implement the pathway-import and 2.5D-stacking capabilities encoded in the matrix: [evidence span withheld — anchor status: not_found]
- [abstract] The unique feature distinguishing transomics2cytoscape in Table 1 is automatic stacked-layer visualization after importing conventional pathway layouts: [evidence span withheld — anchor status: not_found]
- [readme] transomics2cytoscape maintenance status is supported by active Bioconductor release and devel build badges: [evidence span withheld — anchor status: not_found]
- [readme] VANTED's earlier 3D stacked-pathway visualization was manual, grounding its automation status in the matrix: "The 3D network visualization was realized by [VANTED](https://www.cls.uni-konstanz.de/software/vanted/).
However, the visualization relies on time-consuming manual operation."
