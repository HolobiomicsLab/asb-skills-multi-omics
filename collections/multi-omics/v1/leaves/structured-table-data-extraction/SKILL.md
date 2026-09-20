---
name: structured-table-data-extraction
description: Use when an article presents a qualitative feature-comparison matrix
  (e.g., tool-vs-requirement support levels in Table 1) that must be turned into a
  validated, structured file like table1_comparison.tsv with consistent encodings.
when_to_use_negative:
- The table is already available in machine-readable form (e.g., as supplementary
  CSV/XLSX) — re-extraction adds no value
- The target table contains quantitative measurements requiring numeric parsing or
  units rather than qualitative support-level encoding
- The task concerns a single tool's capabilities rather than a cross-tool comparison
  matrix
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: transomics2cytoscape
  role: 'Subject tool whose row must reflect its documented capabilities: KEGGREST
    KGML retrieval, RCy3-driven 2.5D stacking in Cytoscape, and inter-layer edge addition'
  repo: https://github.com/ecell/transomics2cytoscape
- name: Cytoscape
  role: Rendering platform into which the 2.5D stacked network is imported via RCy3;
    grounds the automation requirement of the subject tool's row
  repo: https://github.com/cytoscape/cytoscape
- name: VANTED
  role: Compared tool in Table 1; the article notes its 3D stacked visualization relied
    on time-consuming manual operation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41540-024-00342-8
    title: 'Transomics2cytoscape: an automated software for interpretable 2.5-dimensional
      visualization of trans-omic networks'
  source_content_hash: sha256:f8348f3b04341d8571890cbaad9abbbbcae9f1c2b3c3e5a3f66322703004601e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ecell/transomics2cytoscape
  tools:
  - Cytoscape
  - R
  - VANTED
  - transomics2cytoscape
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# structured-table-data-extraction

## Summary

Converts a qualitative, cross-tool comparison table embedded in a published article (here, Table 1 of the transomics2cytoscape paper comparing seven tools on four trans-omic visualization requirements) into a machine-readable TSV matrix with explicit Yes/No/Partial support-level encodings.

## When to use

Use when an article presents a qualitative feature-comparison matrix (e.g., tool-vs-requirement support levels in Table 1) that must be turned into a validated, structured file like table1_comparison.tsv with consistent encodings.

## When NOT to use

- The table is already available in machine-readable form (e.g., as supplementary CSV/XLSX) — re-extraction adds no value
- The target table contains quantitative measurements requiring numeric parsing or units rather than qualitative support-level encoding
- The task concerns a single tool's capabilities rather than a cross-tool comparison matrix

## Inputs

- Article full text or PDF containing the comparison table (Table 1 of DOI 10.1038/s41540-024-00342-8)
- Article-enriched index of tools, workflow steps, and findings supporting cell-level decisions
- Tool documentation or repository READMEs for maintenance-status assessment (e.g., github.com/ecell/transomics2cytoscape)

## Outputs

- table1_comparison.tsv: header row, one row per compared tool, four requirement columns plus a maintenance-status column with explicit Yes/No/Partial values
- Validation confirmation that all seven tools and six columns are present with no empty cells

## How to apply

Locate the comparison table in the source article and enumerate its axes: for the transomics2cytoscape paper these are seven tools (transomics2cytoscape, VANTED-HIVE, Arena3Dweb Cytoscape app, Multipath, OmicsNet, Graphia, BioLayout Express 3D) crossed with four requirements — (1) importing pathway maps from databases, (2) stacking pathways as layers, (3) drawing edges between stacked layers, and (4) workflow automation. Encode every tool-requirement cell with a consistent Yes/No/Partial vocabulary rather than copying mixed qualitative labels, and add a maintenance-status column informed by the article and tool documentation. Fill the subject tool's row from its described workflow: KEGGREST-based KGML (KEGG pathway XML) retrieval and RCy3-driven stacking into a 2.5D network in Cytoscape support requirements 1, 2, and 4, while inter-layer edge drawing supports requirement 3. Write the result as a tab-separated file named table1_comparison.tsv with a header row, one row per tool. Validate by checking that all seven tools appear as rows, the four requirement columns plus maintenance status are present, and every cell carries an explicit support-level or status value.

## Related tools

- **transomics2cytoscape** (Subject tool whose row must reflect its documented capabilities: KEGGREST KGML retrieval, RCy3-driven 2.5D stacking in Cytoscape, and inter-layer edge addition) — https://github.com/ecell/transomics2cytoscape
- **Cytoscape** (Rendering platform into which the 2.5D stacked network is imported via RCy3; grounds the automation requirement of the subject tool's row) — https://github.com/cytoscape/cytoscape
- **VANTED** (Compared tool in Table 1; the article notes its 3D stacked visualization relied on time-consuming manual operation)

## Examples

```
printf 'tool\tpathway_import\tlayer_stacking\tinterlayer_edges\tworkflow_automation\tmaintenance\n' > table1_comparison.tsv
```

## Evaluation signals

- table1_comparison.tsv parses as valid TSV with exactly 8 lines: 1 header plus 7 tool rows
- Header contains the tool column, the four requirement columns (pathway-map import, layer stacking, inter-layer edges, workflow automation), and maintenance status
- Every cell holds an explicit Yes/No/Partial or maintenance-status value — no blanks, no mixed free-text encodings
- The transomics2cytoscape row encodes Yes for all four requirements, consistent with the article's claim that it satisfies all four
- Partial/No encodings for transomics2cytoscape caveats (e.g., only KEGG ID/pathway file format supported) do not leak into the requirement columns as Yes

## Limitations

- transomics2cytoscape supports only the KEGG ID and pathway file format; Reactome, Wikipathways, and SBGN are not supported, which bounds how its pathway-import capability should be scored
- Z-coordinate estimation in the paper's Case Studies I and II is not automated, and retrieval of inter-layer edges from databases such as BRENDA, OmniPath, SIGNOR, and IntAct is not automated — the workflow-automation cell may warrant Partial depending on scoring rules
- Maintenance status is judged from the article and tool documentation; the qualitative Yes/No/Partial encoding is a curation judgment, not a verbatim copy of Table 1 wording

## Evidence

- [intro] Four requirements frame the comparison matrix: "We specified four requirements for the human-interpretable and reproducible visualization of multi-layered biochemical networks"
- [discussion] Subject tool satisfies all four requirements: "our transomics2cytoscape package satisfies all of the four requirements for the human-interpretable and reproducible visualization of multi-layered biochemical networks"
- [discussion] KEGGREST grounds the pathway-import requirement cell: "Transomics2cytoscape uses KEGGREST to retrieve KGML (KEGG pathway XML)."
- [readme] VANTED comparison basis for manual vs automated scoring: [evidence span withheld — anchor status: not_found]
