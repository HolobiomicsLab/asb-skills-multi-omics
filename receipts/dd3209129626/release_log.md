# Release log: multi-omics v1

| Stage | Done at | Repository commit | Digests |
|---|---|---|---|
| gate | 2026-11-09T14:24:15Z | dc62f362fa92ca1879d396123117088f12e7c0b5 | `{"candidate_tree": "sha256:a777e74b9601194290b9ce44d11d2427083569051813e2b0bada950dd8ae459f", "gate_report": "66c579a11da3846f1138105d671bfd14aae12aee5c05592c72756d417541cf58"}` |
| deposit-plan | 2026-11-09T14:24:15Z | dc62f362fa92ca1879d396123117088f12e7c0b5 | `{"archive_sha256": "256794fb9d3d6b5ed0a4458a53053b5cc22d3b7b0642956137413a3de5c02c45"}` |
| deposit-draft | 2026-11-09T21:05:50Z | dc62f362fa92ca1879d396123117088f12e7c0b5 | `{"archive_sha256": "256794fb9d3d6b5ed0a4458a53053b5cc22d3b7b0642956137413a3de5c02c45"}` |
| record-doi | 2026-11-10T10:47:11Z | dc62f362fa92ca1879d396123117088f12e7c0b5 | `{"CITATION.cff": "461f2dc6bbc9ca36553ac5203fee781bf52b69cef615fba68c7069a771dafb01", "README.md": "17e4712701be778c807cd6282ccee938aa5cc771f34bc0a70aea66d68e573e19", "RELEASE.yaml": "033c62545751868d8d5ef171280ecdd15c9680c5c1a999c3a79ebfb955129d1b", "marketplace.json": "618a5b95179b581e4aa742c1bb90fd72b5c003c8fbf3fe7165936cd5c30152d2", "plugin.json": "c20c796f1fdc84a3933f8e36f44d959f3a9e339976b4a52498567b925e87da71"}` |
| receipt | 2026-11-10T10:47:40Z | dc62f362fa92ca1879d396123117088f12e7c0b5 | `{"receipt": "32d4dedb9e5195781968d630f86f3142e3c98a6dcd358df6e5ad772b461c34fe"}` |
| manifest | 2026-11-10T10:51:11Z | 75b57e9a34cf87a936958c395b2206d2e711a89f | `{"catalogue_jsonld": "341ff530c9464b9b95da82dd1a9f8dcf86eb4afaf0597d8c7cada6485e4c36f1", "collections_yaml": "28aab1a50c7c1e676dd04eda36296728bb43c1318a7b42005da0e491b581168e"}` |
| tag | 2026-11-10T11:06:52Z | 75b57e9a34cf87a936958c395b2206d2e711a89f | `{}` |

## Owner actions

- OWNER: publish the reviewed Zenodo draft at https://zenodo.org/deposit/24093092; publishing mints both the version DOI and the concept DOI
- OWNER: run record-doi --doi <version DOI>, then manifest --concept-doi <concept DOI>
- OWNER: git tag multi-omics-v1.0.0 75b57e9a34cf87a936958c395b2206d2e711a89f
- OWNER: git push origin multi-omics-v1.0.0
- OWNER: git tag multi-omics-v1.0.0 75b57e9a34cf87a936958c395b2206d2e711a89f
- OWNER: git push origin multi-omics-v1.0.0
