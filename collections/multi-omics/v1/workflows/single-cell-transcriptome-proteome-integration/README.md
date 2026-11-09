# single-cell-transcriptome-proteome-integration-workflow

**Status:** published as an **outline** — the structure is validated, the execution is not.
**Kind:** composite-workflow (P1 canonical set).
**Bound by:** perspicacite-semantic (text-embedding-3-large retrieval + EDAM-aware primary selection).

> Automatic grading of `workflow.yaml` (`asb solve-workflow`, checkpoint mode) is **not part of this release**: no released ASB version loads these files. Follow the stages yourself. The stage structure is validated by `validate_workflows.py` through `release_gate.py`.

## Stages

1. **preprocess** — paired RNA and protein counts to normalized modality matrices  →  `single-cell-multiomics-data-preprocessing`, `single-cell-count-matrix-ingestion`, `anndata-object-construction`
2. **neighbors** — normalized modality matrices to separate pruned cell-neighbor graphs  →  `neighborhood-graph-construction-with-neighbor-pruning`, `knn-neighborhood-graph-construction`, `weighted-nearest-neighbour-graph-construction`
3. **integrate** — paired expression matrices and graphs to a joint cell embedding  →  `single-cell-multiomics-integration-training`, `triplet-loss-self-supervised-training`, `multi-omics-embedding-fusion`, `graph-attention-autoencoder-training`, `leiden-clustering-metric-evaluation`
4. **cluster** — joint embedding to cell-cluster assignments across resolutions  →  `leiden-clustering-resolution-sweep`, `leiden-clustering-of-integrated-embeddings`, `single-cell-rna-seq-consensus-clustering`
5. **assess** — cell clusters and reference labels to a clustering-concordance report  →  `clustering-agreement-metric-computation--969d24d6c46ee9fc`, `clustering-agreement-metric-computation--89a468bebe16f1ba`, `adjusted-rand-index-evaluation`

`derived_from_workflows` in the frontmatter is a provenance record — the ASB per-paper workflows whose structure corroborated this pipeline. No ablation experiment consuming it is released.
