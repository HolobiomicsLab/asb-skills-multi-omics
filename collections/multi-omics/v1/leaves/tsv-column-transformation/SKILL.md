---
name: tsv-column-transformation
description: Use when a trans-omic interaction TSV carries identifiers in one namespace
  (e.g. BRENDA EC numbers) that must match the identifier namespace of a pathway map
  (e.g. KEGG reaction IDs) before integrating layers in transomics2cytoscape.
when_to_use_negative:
- Interaction edges are already in the KEGG-compatible namespace the pathway layer
  uses (e.g. KEGG gene IDs), so no ID conversion is needed
- The pathway layer is not a KEGG map — the article states 'Currently, transomics2cytoscape
  supports only the KEGG ID and pathway file format', so EC-to-reaction conversion
  for other map formats is out of scope
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: transomics2cytoscape
  role: Provides the ec2reaction function that performs the EC-number-to-KEGG-reaction-ID
    column conversion on trans-omic interaction tables
  repo: https://github.com/ecell/transomics2cytoscape
- name: KEGGREST
  role: R client used via keggLink/keggFind to map EC numbers to KEGG reaction IDs
    in the 'rn' namespace; the package also uses KEGGREST to retrieve KGML
- name: R
  role: Runtime language in which the TSV is read with read.delim, transformed, and
    written back as allosteric_reactions.tsv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41540-024-00342-8
    title: 'Transomics2cytoscape: an automated software for interpretable 2.5-dimensional
      visualization of trans-omic networks'
  source_content_hash: sha256:38868deb5d51f5cd0beb0cc3a0a9fa8799deb739b99273e49a03d17c5d6e77fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ecell/transomics2cytoscape
  tools:
  - KEGGREST
  - R
  - transomics2cytoscape
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# tsv-column-transformation

## Summary

Transforms a column of a tab-separated trans-omic interaction table from one ID namespace to another — concretely, transomics2cytoscape's ec2reaction converts EC numbers (e.g. 2.7.1.1) in column 6 of a BRENDA allosteric metabolite-protein interaction TSV into KEGG reaction IDs (rn:R#####) via KEGGREST, so heterogeneous edge types can attach to the same metabolic enzyme nodes in the KEGG global metabolic pathway map.

## When to use

Use when a trans-omic interaction TSV carries identifiers in one namespace (e.g. BRENDA EC numbers) that must match the identifier namespace of a pathway map (e.g. KEGG reaction IDs) before integrating layers in transomics2cytoscape.

## When NOT to use

- Interaction edges are already in the KEGG-compatible namespace the pathway layer uses (e.g. KEGG gene IDs), so no ID conversion is needed
- The pathway layer is not a KEGG map — the article states 'Currently, transomics2cytoscape supports only the KEGG ID and pathway file format', so EC-to-reaction conversion for other map formats is out of scope

## Inputs

- Allosteric metabolite-protein interaction TSV (tab-separated) whose column 6 contains EC numbers, e.g. 2.7.1.1
- EC-number strings, optionally with an 'ec:' prefix

## Outputs

- allosteric_reactions.tsv — tab-separated, UTF-8, same row count as input, with an added KEGG reaction-ID column (rn:R followed by five digits) alongside the original EC column

## How to apply

Load the interaction TSV in R with read.delim, preserving column order so the EC-number column (column 6 in the allosteric metabolite-protein table) stays fixed. Extract unique EC strings from that column and strip any 'ec:' prefix, then use KEGGREST (keggLink/keggFind) to map each EC number to KEGG reaction ID(s) in the 'rn' namespace, concatenating IDs when one EC number maps to several reactions (one-to-many mappings). Rebuild the TSV with a new reaction-ID column alongside the original EC column, keeping all other columns unchanged, and write it as tab-separated UTF-8 (e.g. allosteric_reactions.tsv) with the same row count as the input. The rationale is that BRENDA-style metabolite-protein edges and enzyme phosphorylation edges use different ID namespaces, and only reaction IDs can connect both edge types to the same enzyme nodes in the 2.5D KEGG global metabolic pathway map built by transomics2cytoscape.

## Related tools

- **transomics2cytoscape** (Provides the ec2reaction function that performs the EC-number-to-KEGG-reaction-ID column conversion on trans-omic interaction tables) — https://github.com/ecell/transomics2cytoscape
- **KEGGREST** (R client used via keggLink/keggFind to map EC numbers to KEGG reaction IDs in the 'rn' namespace; the package also uses KEGGREST to retrieve KGML)
- **R** (Runtime language in which the TSV is read with read.delim, transformed, and written back as allosteric_reactions.tsv)

## Examples

```
library(KEGGREST); rn <- keggLink('reaction', paste0('ec:', ecs)); allosteric_reactions <- cbind(allosteric, reaction_id = rn); write.table(allosteric_reactions, 'allosteric_reactions.tsv', sep = '\t', quote = FALSE)
```

## Evaluation signals

- allosteric_reactions.tsv exists after the run
- Output row count equals the input TSV row count (no rows dropped or added)
- Output contains a reaction-ID field distinct from the original EC column, with all other columns unchanged
- Every reaction ID matches the KEGG reaction-ID format: rn:R followed by five digits
- Each mapped reaction ID corresponds to the source EC number when checked through KEGGREST (one-to-many EC mappings yield concatenated reaction IDs)

## Limitations

- EC-to-reaction mappings can be one-to-many, requiring concatenation of multiple reaction IDs per row
- transomics2cytoscape currently supports only KEGG IDs and the KEGG pathway file format; Reactome, Wikipathways, and SBGN are not supported, so this conversion only feeds KEGG-based maps
- The article notes retrieval of inter-layer edges from databases such as BRENDA is not automated — the input TSV must be prepared manually before this transformation

## Evidence

- [other] ec2reaction converts the EC-number column of a trans-omic interaction table into KEGG reaction IDs, enabling BRENDA-style metabolite-protein interactions and enzyme phosphorylation interactions with different ID namespaces to connect to the same metabolic enzyme nodes: [evidence span withheld — anchor status: not_found]
- [other] KEGGREST used to map EC numbers to reaction IDs in the rn namespace: [evidence span withheld — anchor status: not_found]
- [discussion] Article discussion on ID conversion into KEGG-compatible IDs: "The IDs of molecules were converted into KEGG–compatible IDs such as KEGG gene ID and EC numbers by using Ensembl BioMart42"
- [discussion] Article discussion confirming KEGGREST is the KEGG interface used by the package: "Transomics2cytoscape uses KEGGREST to retrieve KGML (KEGG pathway XML)."
- [discussion] Article discussion on scope limitation to KEGG formats: "Currently, transomics2cytoscape supports only the KEGG ID and pathway file format."
