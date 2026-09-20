---
name: multiplex-network-gene-set-evaluation
description: Use when a curated gene set (e.g. shared MAPMAN or GO annotation) must
  be validated for functional/topological connectivity in a preassembled multiplex
  network before using it for ranking or discovery.
when_to_use_negative:
- The goal is scoring connectivity of a fixed pair or small set of named seed genes
  against all others rather than validating a set — use RWR_LOE instead of cross-validation.
- The input is a single monoplex network with one edge type; LOO/kfold CV here presumes
  a multiplex where layers carry distinct evidence (coexpression, PPI, regulation,
  metabolic, etc.).
- No curated gene set or annotation source (MAPMAN, GO) is available to define positives
  and non-member controls — cross-validation cannot be interpreted without them.
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3957
tools:
- name: RWRtoolkit
  role: Provides RWR_CV, RWR_make_multiplex, and RWR_LOE used to construct the multiplex
    and run k-fold/LOO cross-validation on gold-set genes
  repo: github.com/dkainer/RWRtoolkit
- name: RandomWalkRestartMH
  role: Underlying engine providing the random-walk-with-restart machinery on multiplex
    networks that RWR_CV wraps
  repo: github.com/alberto-valdeolivas/RandomWalkRestartMH
- name: R
  role: Runtime environment (R >= 4.1.0) in which the RWRtoolkit package and cross-validation
    workflow are executed
  repo: github.com/dkainer/RWRtoolkit
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaf028
    title: 'RWRtoolkit: multi-omic network analysis using random walks on multiplex
      networks in any species'
  source_content_hash: sha256:b7cbd872e4c0f0fd1a33d317da6093e541d0ad5e658ac36b0315cfd49b52d5ad
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - RWRtoolkit
  - RandomWalkRestartMH
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3957
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# multiplex-network-gene-set-evaluation

## Summary

Evaluate whether a curated gene set is topologically interconnected within a multiplex biological network by running Random Walk with Restart cross-validation (RWR_CV, k-fold or LOO) and inspecting per-gene mean ranks and AUROC against the 0.50 random baseline.

## When to use

Use when a curated gene set (e.g. shared MAPMAN or GO annotation) must be validated for functional/topological connectivity in a preassembled multiplex network before using it for ranking or discovery.

## When NOT to use

- The goal is scoring connectivity of a fixed pair or small set of named seed genes against all others rather than validating a set — use RWR_LOE instead of cross-validation.
- The input is a single monoplex network with one edge type; LOO/kfold CV here presumes a multiplex where layers carry distinct evidence (coexpression, PPI, regulation, metabolic, etc.).
- No curated gene set or annotation source (MAPMAN, GO) is available to define positives and non-member controls — cross-validation cannot be interpreted without them.

## Inputs

- Multiplex network .RData object built by RWR_make_multiplex from an flist of network layers (e.g. Comprehensive_Network_AT_d0.5_v02.RData, delta = 0.5)
- Gold-set gene list file in two-column set/gene format, mixing positively annotated genes and non-member control genes
- Choice of CV method ('kfold' with k = 5, or 'loo') and restart probability (0.7 in the demonstrated workflow)

## Outputs

- Full per-fold rank table for all genes in the multiplex
- Per-gene mean ranks across CV folds for each seed gene
- Summary metrics file including AUROC for the gene set
- Summary file reporting whether held-out genes were recovered in top ranks (e.g. top 100)

## How to apply

Load the multiplex (e.g. the GO_AT_d0.5_v01 Arabidopsis multiplex built with delta = 0.5) into an mpo object via RWRtoolkit::RWR_make_multiplex, which wraps create.multiplex, compute.adjacency.matrix, and compute.transition.matrix. Format the gold set as a two-column set/gene file — in the jasmonate case, 27 MAPMAN 'Jasmonate' genes plus 3 deliberately added non-jasmonate genes (AT1G08380, AT4G33360, AT5G23010) as negative controls. Run RWRtoolkit::RWR_CV with method = 'kfold', k = 5 (or method = 'loo' for small sets), and restart probability 0.7, which leaves out each gene (or fold) and reruns the walk to test whether held-out seeds are recovered by the remaining seeds. Parse the four output files (full ranks, mean ranks, metrics, summary) to extract per-gene mean ranks. Judge success by rank recovery of true members versus non-members and AUROC well above the 0.50 random baseline — e.g. 23 of 27 jasmonate genes in the top 100 mean ranks while non-jasmonate controls ranked near 2600/9425/9707.

## Related tools

- **RWRtoolkit** (Provides RWR_CV, RWR_make_multiplex, and RWR_LOE used to construct the multiplex and run k-fold/LOO cross-validation on gold-set genes) — github.com/dkainer/RWRtoolkit
- **RandomWalkRestartMH** (Underlying engine providing the random-walk-with-restart machinery on multiplex networks that RWR_CV wraps) — github.com/alberto-valdeolivas/RandomWalkRestartMH
- **R** (Runtime environment (R >= 4.1.0) in which the RWRtoolkit package and cross-validation workflow are executed) — github.com/dkainer/RWRtoolkit

## Examples

```
mpo <- RWRtoolkit::RWR_make_multiplex(flist='flist.tsv'); RWRtoolkit::RWR_CV(multiplex=mpo, geneset_path='jasmonate_goldset.tsv', method='kfold', k=5, r=0.7, outdir='cv_results/')
```

## Evaluation signals

- Functionally connected gene sets show AUROC well above the 0.50 random baseline in the RWR_CV metrics output
- Most positively annotated genes (e.g. 23 of 27 jasmonate genes) fall within the top 100 mean ranks, while non-member control genes rank far lower (e.g. near 2600/9425/9707)
- Four expected output files are produced: full ranks, mean ranks, metrics, and summary
- Gene set file parses correctly in two-column set/gene format and contains both annotated positives and non-member controls
- Mean-rank distribution separates positives from controls by orders of magnitude rather than overlapping them

## Limitations

- Results depend on the quality of the multiplex: the demonstration used a 9-layer comprehensive Arabidopsis multiplex with delta = 0.5, and layers with sparse or noisy edges will degrade rank recovery
- The RandomWalkRestartMH implementation historically mis-weighted seeds with tau equally across all layers rather than per layer, affecting walk-based rankings
- Set size matters: very small sets suit LOO (each of the 30 genes left out one at a time), while k-fold (k = 5) is used for the demonstrated 30-gene set; random gene sets will not separate from baseline and may look falsely poor
- No version number for RWRtoolkit or RandomWalkRestartMH is stated in the article, complicating exact reproduction

## Evidence

- [methods] RWR_CV cross-validation on curated gene sets: "we employed Random Walk with Restart Cross-Validation (RWR_CV) using a series of carefully curated gene sets"
- [discussion] Cross-validation on the Arabidopsis multiplex shows functional sets have strong topological similarity: "The example cross-validation experiment conducted on our 9-layer comprehensive Arabidopsis multiplex demonstrates that functionally related groups of genes have strong topological similarity"
- [discussion] LOO test count for a 30-gene set: "Given that there are 30 genes within the set, 30 individual tests of RWR were run, leaving only one gene out each time."
- [discussion] MAPMAN jasmonate set used in the RWR_CV example: "Using a set of genes sharing the jasmonate signalling term from the MAPMAN database, we see a small example of the power of RWR_CV’s LOO method."
- [readme] RWR_make_multiplex is the entry point for building the multiplex object: [evidence span withheld — anchor status: not_found]
