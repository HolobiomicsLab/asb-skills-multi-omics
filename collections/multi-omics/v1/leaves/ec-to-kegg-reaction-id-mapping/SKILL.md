---
name: ec-to-kegg-reaction-id-mapping
description: Use when an inter-layer interaction TSV references enzymes by EC numbers
  (e.g. 2.7.1.1 in column 6) but the downstream pathway layer uses KEGG reaction IDs
  (rn:R#####), and both edge types must attach to the same KEGG enzyme layer.
when_to_use_negative:
- When the interaction table already uses KEGG reaction IDs rather than EC numbers
  — no namespace conversion is needed.
- 'When the pathway layer is not KEGG-based: the article states ''Currently, transomics2cytoscape
  supports only the KEGG ID and pathway file format'' (Reactome, Wikipathways, SBGN
  are not supported yet).'
- 'When mapping non-enzyme molecule identifiers (e.g. gene/protein IDs): the article
  converts those ''into KEGG-compatible IDs such as KEGG gene ID and EC numbers by
  using Ensembl BioMart'', which is a different conversion path.'
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: transomics2cytoscape
  role: Provides the ec2reaction function that converts the EC-number column of a
    trans-omic interaction table into KEGG reaction IDs
  repo: https://github.com/ecell/transomics2cytoscape
- name: KEGGREST
  role: R client used to map EC numbers to KEGG reaction IDs in the 'rn' namespace
    (keggLink/keggFind); the package also uses KEGGREST to retrieve KGML
- name: R
  role: Runtime environment for reading the TSV (read.delim), performing the ID conversion,
    and writing allosteric_reactions.tsv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41540-024-00342-8
    title: 'Transomics2cytoscape: an automated software for interpretable 2.5-dimensional
      visualization of trans-omic networks'
  source_content_hash: sha256:f2e73a4d6d04c6653731a8b9b9f602da75cdb06519c3f4e5cf3e7aa3779cb322
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
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# ec-to-kegg-reaction-id-mapping

## Summary

Converts the EC-number column of a trans-omic interaction table (e.g. BRENDA-derived metabolite-protein interactions) into KEGG reaction IDs using the ec2reaction function of transomics2cytoscape with KEGGREST, so BRENDA-style edges and enzyme phosphorylation edges can both attach to the same metabolic enzyme nodes in the KEGG global metabolic pathway map.

## When to use

Use when an inter-layer interaction TSV references enzymes by EC numbers (e.g. 2.7.1.1 in column 6) but the downstream pathway layer uses KEGG reaction IDs (rn:R#####), and both edge types must attach to the same KEGG enzyme layer.

## When NOT to use

- When the interaction table already uses KEGG reaction IDs rather than EC numbers — no namespace conversion is needed.
- When the pathway layer is not KEGG-based: the article states 'Currently, transomics2cytoscape supports only the KEGG ID and pathway file format' (Reactome, Wikipathways, SBGN are not supported yet).
- When mapping non-enzyme molecule identifiers (e.g. gene/protein IDs): the article converts those 'into KEGG-compatible IDs such as KEGG gene ID and EC numbers by using Ensembl BioMart', which is a different conversion path.

## Inputs

- Tab-separated trans-omic interaction TSV (e.g. BRENDA-derived allosteric metabolite-protein interactions) with EC numbers in column 6
- KEGG reaction ID namespace ('rn') accessed via KEGGREST

## Outputs

- allosteric_reactions.tsv — tab-separated, UTF-8 interaction table with a KEGG reaction-ID column added, same row count as the input

## How to apply

Load the allosteric metabolite-protein interaction TSV in R with read.delim, preserving column order so column 6 contains the EC numbers (e.g. 2.7.1.1). Extract the unique EC strings, strip any 'ec:' prefix, and use KEGGREST (keggLink/keggFind) to map each EC number to its KEGG reaction ID(s) in the 'rn' namespace, handling one-to-many mappings by concatenating reaction IDs. Rebuild the TSV with a new reaction-ID column alongside the original EC column, leaving all other columns unchanged, and write it as a tab-separated, UTF-8 file (e.g. allosteric_reactions.tsv). Rationale: transomics2cytoscape only supports KEGG IDs and pathway file formats, and it relies on KEGGREST for KEGG data retrieval, so EC numbers from BRENDA must be converted into the KEGG reaction namespace before edges can be joined to the global metabolic pathway layer.

## Related tools

- **transomics2cytoscape** (Provides the ec2reaction function that converts the EC-number column of a trans-omic interaction table into KEGG reaction IDs) — https://github.com/ecell/transomics2cytoscape
- **KEGGREST** (R client used to map EC numbers to KEGG reaction IDs in the 'rn' namespace (keggLink/keggFind); the package also uses KEGGREST to retrieve KGML)
- **R** (Runtime environment for reading the TSV (read.delim), performing the ID conversion, and writing allosteric_reactions.tsv)

## Examples

```
library(KEGGREST); ecs <- gsub('^ec:', '', as.character(tsv$EC)); rxn <- keggLink('reaction', paste0('ec:', ecs))
```

## Evaluation signals

- allosteric_reactions.tsv exists after the run
- Output file has the same number of rows as the input TSV (no rows dropped or added by the conversion)
- Output contains a reaction-ID field, and reaction IDs match the KEGG reaction-ID format 'rn:R' followed by five digits
- One-to-many EC mappings are handled by concatenating multiple reaction IDs for a single EC number
- All columns other than the new reaction-ID column are unchanged from the input TSV (original EC column preserved)

## Limitations

- transomics2cytoscape supports only KEGG IDs and the KEGG pathway file format; other pathway databases such as Reactome and Wikipathways are not supported yet.
- Retrieval of inter-layer edges from databases such as BRENDA is not automated — the interaction TSV must be prepared manually before conversion.
- EC-to-reaction mappings can be one-to-many, so a single interaction row may map to multiple concatenated reaction IDs.

## Evidence

- [other] transomics2cytoscape provides the ec2reaction function, which converts the EC-number column of a trans-omic interaction table into KEGG reaction IDs: [evidence span withheld — anchor status: not_found]
- [discussion] Interactions between omic layers can also be obtained from databases such as BRENDA as well as pathway data from KEGG.: "Interactions between omic layers can also be obtained from databases such as BRENDA as well as pathway data from KEGG."
- [discussion] Currently, transomics2cytoscape supports only the KEGG ID and pathway file format.: "Currently, transomics2cytoscape supports only the KEGG ID and pathway file format."
- [discussion] Transomics2cytoscape uses KEGGREST to retrieve KGML (KEGG pathway XML).: "Transomics2cytoscape uses KEGGREST to retrieve KGML (KEGG pathway XML)."
