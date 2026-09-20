---
name: software-maintenance-status-assessment
description: Use when a paper's Table 1 (or equivalent) compares several tools against
  explicit capability requirements and the matrix must be reproduced with consistent
  support encoding and a tool maintenance-status assessment from repositories and
  build badges.
when_to_use_negative:
- The goal is a performance or accuracy benchmark (runtime, layout quality) rather
  than a discrete feature-support and maintenance assessment
- The comparison target is a single tool's capability documentation rather than a
  multi-tool requirement matrix (e.g., a Table 1)
- The source material does not state requirement-support levels explicitly, so every
  cell would have to be guessed rather than extracted and cited
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: transomics2cytoscape
  role: Subject tool assessed against all four requirements (Yes on pathway import
    via KEGGREST KGML, 2.5D layer stacking via RCy3 in Cytoscape, inter-layer edges,
    automation); its repo/build badges ground the maintenance-status cell
  repo: https://github.com/ecell/transomics2cytoscape
- name: Cytoscape
  role: Target platform whose automation support (RCy3/Cytoscape Automation) is one
    assessed requirement dimension; its repository informs the maintenance-status
    column
  repo: https://github.com/cytoscape/cytoscape
- name: VANTED
  role: Compared tool; its earlier 3D stacked-layer visualization relied on time-consuming
    manual operation, informing its Partial/No encoding on the automation requirement
- name: KEGGREST
  role: Mechanism cited in the matrix row for the pathway-import requirement (retrieves
    KEGG KGML pathway XML)
- name: RCy3
  role: Mechanism cited in the matrix row for the layer-stacking and automation requirements
    (automates importing multiple pathway maps into a 2.5D network in Cytoscape)
- name: R
  role: Language ecosystem (R Bioconductor package) whose release/dev build status
    grounds the maintenance assessment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41540-024-00342-8
    title: 'Transomics2cytoscape: an automated software for interpretable 2.5-dimensional
      visualization of trans-omic networks'
  source_content_hash: sha256:9d4ef16e0e7b075b9998862363cf84a407ce5ba41bc5f4b74956dc8c39b16cbe
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ecell/transomics2cytoscape
  tools:
  - Cytoscape
  - KEGGREST
  - R
  - RCy3
  - VANTED
  - transomics2cytoscape
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# software-maintenance-status-assessment

## Summary

Assembles a feature-vs-maintenance comparison matrix for scientific software (here, seven trans-omic visualization tools judged against four requirements: pathway map import, layer stacking, inter-layer edges, and workflow automation), encoding support as Yes/No/Partial plus a maintenance-status column.

## When to use

Use when a paper's Table 1 (or equivalent) compares several tools against explicit capability requirements and the matrix must be reproduced with consistent support encoding and a tool maintenance-status assessment from repositories and build badges.

## When NOT to use

- The goal is a performance or accuracy benchmark (runtime, layout quality) rather than a discrete feature-support and maintenance assessment
- The comparison target is a single tool's capability documentation rather than a multi-tool requirement matrix (e.g., a Table 1)
- The source material does not state requirement-support levels explicitly, so every cell would have to be guessed rather than extracted and cited

## Inputs

- Article Table 1: tool-vs-requirement support matrix for seven trans-omic visualization tools
- Tool repository state (e.g., github.com/ecell/transomics2cytoscape with Bioconductor release/dev build badges)
- Tool documentation (Bioconductor package pages, README files, Cytoscape app pages)
- Article discussion text on which capabilities are automated vs. manual (KEGGREST KGML retrieval, RCy3 integration)

## Outputs

- table1_comparison.tsv: headered tab-separated matrix with one row per tool and columns for the four requirements plus a maintenance-status column
- Validated tool-by-requirement support encoding (Yes/No/Partial) for all seven tools

## How to apply

Extract from the source article's Table 1 the support level of each of the seven tools (transomics2cytoscape, VANTED-HIVE, Arena3Dweb Cytoscape app, Multipath, OmicsNet, Graphia, BioLayout Express 3D) for the four requirements: (1) importing pathway maps from databases, (2) stacking pathways as layers, (3) drawing edges between stacked layers, and (4) workflow automation. Encode support with a single consistent Yes/No/Partial vocabulary across all cells. Assess maintenance status per tool from its repository and documentation (e.g., the ecell/transomics2cytoscape GitHub repo with Bioconductor Release and Dev build badges, and the Bioconductor package page). Fill the transomics2cytoscape row with Yes on all four requirements, grounded in its KEGGREST-based KGML retrieval and RCy3-driven 2.5D stacking in Cytoscape, while noting that inter-layer edge retrieval from databases such as BRENDA, OmniPath, SIGNOR, and IntAct is not automated. Write the result as a headered tab-separated file named table1_comparison.tsv, one row per tool, with columns for the four requirements plus maintenance status.

## Related tools

- **transomics2cytoscape** (Subject tool assessed against all four requirements (Yes on pathway import via KEGGREST KGML, 2.5D layer stacking via RCy3 in Cytoscape, inter-layer edges, automation); its repo/build badges ground the maintenance-status cell) — https://github.com/ecell/transomics2cytoscape
- **Cytoscape** (Target platform whose automation support (RCy3/Cytoscape Automation) is one assessed requirement dimension; its repository informs the maintenance-status column) — https://github.com/cytoscape/cytoscape
- **VANTED** (Compared tool; its earlier 3D stacked-layer visualization relied on time-consuming manual operation, informing its Partial/No encoding on the automation requirement)
- **KEGGREST** (Mechanism cited in the matrix row for the pathway-import requirement (retrieves KEGG KGML pathway XML))
- **RCy3** (Mechanism cited in the matrix row for the layer-stacking and automation requirements (automates importing multiple pathway maps into a 2.5D network in Cytoscape))
- **R** (Language ecosystem (R Bioconductor package) whose release/dev build status grounds the maintenance assessment)

## Examples

```
BiocManager::install("transomics2cytoscape")
```

## Evaluation signals

- table1_comparison.tsv contains exactly seven data rows, one for each of transomics2cytoscape, VANTED-HIVE, Arena3Dweb Cytoscape app, Multipath, OmicsNet, Graphia, and BioLayout Express 3D
- Header row names all four requirement columns (pathway import, layer stacking, inter-layer edges, workflow automation) plus a maintenance-status column
- Every cell carries an explicit support level (Yes/No/Partial) or an explicit maintenance status — no empty or ambiguous cells
- The transomics2cytoscape row is Yes on all four requirements, consistent with the article's claim that it satisfies all four, with KEGGREST KGML retrieval and RCy3-driven 2.5D stacking noted
- Maintenance statuses are traceable to repository/build-badge evidence (e.g., Bioconductor Release and Dev build shields in the transomics2cytoscape README)

## Limitations

- Feature-support levels come from a single publication snapshot (Table 1 of the 2024 article); maintenance status is time-sensitive and must be re-verified against live repositories
- transomics2cytoscape currently supports only the KEGG ID and pathway file format, so its pathway-import Yes is narrower than general pathway-database support
- Some transomics2cytoscape capabilities are partial: Z-coordinate estimation in Case Study I/II, inter-layer edge retrieval from databases (BRENDA, OmniPath, SIGNOR, IntAct), and trans-omic connectivity reconstruction are not fully automated, which must be reflected honestly in Partial encodings

## Evidence

- [intro] four requirements for trans-omic visualization: "We specified four requirements for the human-interpretable and reproducible visualization of multi-layered biochemical networks, i.e., trans-omics networks:"
- [discussion] transomics2cytoscape satisfies all four requirements: "our transomics2cytoscape package satisfies all of the four requirements for the human-interpretable and reproducible visualization of multi-layered biochemical networks"
- [discussion] KEGGREST-based pathway import mechanism: "Transomics2cytoscape uses KEGGREST to retrieve KGML (KEGG pathway XML)."
- [discussion] non-automated inter-layer edge retrieval limitation: "transomics2cytoscape has not yet automated the retrieval processes for inter-layer edges."
- [readme] manual-operation rationale for automation requirement: "However, the visualization relies on time-consuming manual operation."
