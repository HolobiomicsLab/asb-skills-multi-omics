<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/brand/asb-logo-dark.svg">
    <source media="(prefers-color-scheme: light)" srcset="assets/brand/asb-logo.svg">
    <img alt="AgenticScienceBuilder" src="assets/brand/asb-logo.svg" width="320">
  </picture>
</p>

<h1 align="center">ASB Skills: Multi-omics</h1>

<p align="center"><em>Evidence-grounded skills and tool records for multi-omics integration: joint analysis of transcriptomic, proteomic, metabolomic and epigenomic layers.</em></p>

<p align="center">
  <img src="https://img.shields.io/badge/skills-787-00284b" alt="787 skills">
  <img src="https://img.shields.io/badge/tools-211-00284b" alt="211 tools">
  <img src="https://img.shields.io/badge/workflows-2-00284b" alt="2 workflows">
  <img src="https://img.shields.io/badge/papers-29-00284b" alt="29 source papers">
  <img src="https://img.shields.io/badge/content-CC--BY--4.0-04919a" alt="content CC-BY-4.0">
  <img src="https://img.shields.io/badge/code-Apache--2.0-04919a" alt="code Apache-2.0">
  <img src="https://img.shields.io/badge/status-released-04919a" alt="status released">
</p>

---

## TL;DR

This repository holds the multi-omics skill collection of AgenticScienceBuilder (ASB): 787 skills over 211 software tools and 2 composite workflows, distilled from 29 method papers and their code repositories. Each skill is a `SKILL.md` file that an agent can load. It names its source paper; 509 of the 787 also quote it or its repository README. The collection installs as a Claude Code plugin, and a script that needs only the Python standard library searches it offline.

## Collections

| Path | Title | Skills | Tools | Released | DOI |
|---|---|---|---|---|---|
| `collections/multi-omics/v1` | ASB Multi-omics Skill Collection v1 | 787 | 211 | 2026-11-15 | [10.5281/zenodo.24255470](https://doi.org/10.5281/zenodo.24255470) |

## Status

Collection `v1`, built from corpus cut `dd3209129626`, is released on `main` at tag `multi-omics-v1.1.1` and archived on Zenodo under the version DOI [10.5281/zenodo.24255470](https://doi.org/10.5281/zenodo.24255470).

| Item | State |
|---|---|
| Release gate | `receipts/dd3209129626/gate_report.json` reports `pass`: 15 of 16 checks pass and 1 is not applicable (`cut_manifest`). [`receipts/dd3209129626/VERIFY.md`](receipts/dd3209129626/VERIFY.md) gives the command that re-verifies it. |
| Plugin manifests | `claude plugin validate` passes for `.claude-plugin/marketplace.json` and for `collections/multi-omics/v1` |
| Collection IRI | `https://w3id.org/holobiomicslab/asb-skill/collection/multi-omics/v1` |
| IRI resolution (2026-11-14) | Registered at w3id.org. It redirects (HTTP 302) to the collection landing page in the asb-skill-collections registry (HTTP 200), which names this repository and the Zenodo record. |
| DOI | [10.5281/zenodo.24255470](https://doi.org/10.5281/zenodo.24255470) (this version). The Zenodo deposit metadata is in [`.zenodo.json`](.zenodo.json). |
| Concept DOI | [10.5281/zenodo.24093090](https://doi.org/10.5281/zenodo.24093090) (all versions) |

## What is inside

| Item | Count | Path under `collections/multi-omics/v1/` |
|---|--:|---|
| Leaf skills (`SKILL.md`) | 787 | `leaves/<slug>/SKILL.md` |
| Skill router | 1 | `skills/_router/` |
| Composite workflows | 2 | `workflows/<slug>/` |
| Workflow router | 1 | `workflows/_workflow_router/` |
| Software-tool records | 211 | `tools/` |
| Source papers | 29 | `corpus.yaml` |
| Indexes | 4 | `skills_index.json`, `tools_index.json`, `workflows/workflows_index.json`, `kb_bundle.json` |
| Offline search script | 1 | `bin/search_skills.py` |

The build and gate receipts are in `receipts/dd3209129626/` at the repository root.

Composite workflows chain leaf skills into a typed pipeline:

| Workflow | Use when | Stages |
|---|---|--:|
| `single-cell-transcriptome-proteome-integration` | paired single-cell transcriptome and protein count matrices with reference cell labels need a joint embedding, a clustering resolution sweep, and a report of agreement with the supplied labels | 5 |
| `spatial-transcriptomics-niche-analysis` | image-based spatial transcriptomics includes transcript coordinates, staining channels and an aligned histology image, and needs resolved cell boundaries, per-cell measurements and image-derived spatial niches | 6 |

## Licence tiers

Each skill carries a `license_tier` that answers one question: what may you do with the underlying tool?
The tiers are defined in the registry's [licence-tier policy](https://github.com/HolobiomicsLab/asb-skill-collections/blob/main/governance/LICENSE_TIERS.md).
The tier is advice about the tool. It does not change the licence of this collection's content.

| Tier | Skills |
|---|--:|
| `open` | 462 |
| `noncommercial` | 325 |
| `restricted` | 0 |
| `unknown` | 0 |

## Install

Claude Code, from the directory that holds your clone of this repository:

```bash
claude plugin marketplace add ./asb-skills-multi-omics
claude plugin install multi-omics@asb-skills-multi-omics
```

Other agents: every skill is one Markdown file with YAML front matter. Point the agent at `collections/multi-omics/v1/leaves/`, or let it call the search script below.

### Only a few skills

Each leaf directory holds a single `SKILL.md`, which does not refer to other files of the collection. Copy the leaves you need, from the directory that holds your clone. Claude Code reads the skills of a project from `<project>/.claude/skills/`:

```bash
mkdir -p <project>/.claude/skills
cp -R asb-skills-multi-omics/collections/multi-omics/v1/leaves/multimodal-factor-analysis <project>/.claude/skills/
```

The search script below prints the slug of each match.

## Search offline

The search script needs only the Python standard library.

```bash
cd asb-skills-multi-omics
python3 collections/multi-omics/v1/bin/search_skills.py --query "multi-omics factor analysis" -k 3
python3 collections/multi-omics/v1/bin/search_skills.py --target workflows --query "joint embedding of RNA and protein" -k 2
```

Each result prints its rank and slug, then its description, tools, licence tier and the path of the file to read. The ranked slugs:

```text
1. multimodal-factor-analysis
2. single-cell-multiome-count-preprocessing
3. multi-omics-power-analysis

1. single-cell-transcriptome-proteome-integration
```

## How it was built

The AgenticScienceBuilder pipeline read each paper and its code repository, wrote the skills, and passed the collection through a strict release gate. `receipts/dd3209129626/source.json` names the pipeline commit `da8abd15`, the collections-tooling commit `b8c22a6bc` and the corpus commit `afa4911`.

The gate checks that every included paper has an allowed access type, caps the length and similarity of verbatim quotes, scans quote spans for personal data and dual-use content, requires provenance and a licence tag on every skill, resolves each composite workflow to real leaves, and binds the receipt to the hashes of the collection files.

All 29 included papers are open access according to `corpus.yaml` (`access.is_oa`). 4 of the 29 are bioRxiv preprints.

Each leaf names its source paper in `provenance.source_papers` (787 of 787 leaves). 509 of the 787 quote it, or its repository README, in their `## Evidence` section. In the other 278, the gate parsed no quote span (`gate_report.json`, `strip_verbatim_similarity`, `coverage.files_without_spans`). In 274 of them the section is empty or its quotes are withheld because they could not be anchored in the source. In 4 the section holds a quote in a form the gate parser does not read.

## Related repositories

- [ASB skill collections](https://github.com/HolobiomicsLab/asb-skill-collections): the registry of every domain, the shared tooling, the governance documents, the contribution loop and the earlier metabolomics v2 tree. The current metabolomics collection (v3) is in the `asb-skills-metabolomics` repository.
- AgenticScienceBuilder: the pipeline that generated this collection. Repository follows.
- [`indicium-adapters-multi-omics`](https://github.com/HolobiomicsLab/indicium-adapters-multi-omics): the Indicium domain adapter for the same domain.

## Citation

Cite the collection with [`CITATION.cff`](CITATION.cff). Cite also the source paper of each skill you use: its DOI is in `provenance.source_papers` in the skill's front matter. The collection's version DOI is [10.5281/zenodo.24255470](https://doi.org/10.5281/zenodo.24255470); its concept DOI, which covers all versions, is [10.5281/zenodo.24093090](https://doi.org/10.5281/zenodo.24093090).

## Licence

Code (the search script and other scripts): Apache-2.0, see [`LICENSE`](LICENSE). Collection content (`SKILL.md` files, tool records, indexes, metadata): CC-BY-4.0, see [`LICENSING.md`](LICENSING.md). Short verbatim quotes from source papers are reproduced for attribution under quotation right. A rights holder can ask for removal through an issue.
