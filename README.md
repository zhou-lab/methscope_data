# MethScope Data — test fixtures, reproducibility archive

Companion data repo for [**methscope-cli**](https://github.com/zhou-lab/methscope-cli).
The runnable model bundles are hosted on HuggingFace — too large for git — at
[`zhou-lab/methscope`](https://huggingface.co/zhou-lab/methscope); model
descriptions live with the models (the HuggingFace model card and the
[methscope-cli documentation](https://github.com/zhou-lab/methscope-cli)), and
their published digests in the YAME registry, so nothing here needs updating
when a model does. This repo holds what stays in git: small **test fixtures**
(`test/`) and the MRMP-construction **reproducibility archive** (tag `v1`).

## Test fixtures (`test/`)

Small query `.cg` fixtures used by the methscope-cli README/examples: typed
cells (`human_hg38_celltypes.cg`), simulated deconvolution mixtures, an
upscale input + truth (`human_hg38_test.cg`, `human_hg38_test.truth.cg`), and
one small reference so `mrmp-build` is runnable without the 2.3 GB atlas.

`test/human_hg38_immune_mixture.cg` — simulated deconvolution mixtures with
**exact known truths**, blended from a deconvolution reference's own class
pools. Record names encode composition and sparsity (e.g.
`mac70_mono30_2pow22` = Macrophage 0.70 / Mono 0.30 at 2^22 binarized CpGs,
1 read each), so each record checks an answer rather than merely producing
one. The sparsest rung sits deliberately at the depth where single draws
become seed-dependent.

`test/human_hg38_40_celltypes_chr20.cg` — 40 Loyfer cell types, chr20 only
(773,477 CpGs, 40 MB). `mrmp-build` packs a pattern as a base-3 `uint64` and
3^40 < 2^64 < 3^41, so 40 samples is the hard ceiling; chr20 keeps it small
enough to ship. The shipped `.cg.idx` *is* the sample list; rebuild from the
lab store by subsetting those samples from `2023_Loyfer.cg` and taking the
chr20 row block (recipe in the MethScope lab journal).

`test/SHA256SUMS` records the digest of every fixture (`sha256sum -c
SHA256SUMS`). It is a convenience for anyone verifying by hand: `methscope
fetch` does not read it, because each digest is compiled into the tool's
registry, so a download is checked against something the binary already held
rather than against a file served by the same host.

## Reproducibility archive (tag `v1`)

The full MRMP definition sets (`*.cm`), pattern-definition tables (`*_def*`),
and legacy deconvolution references (`*_ref.rds`) are archived at tag **`v1`**
(`git checkout v1`). MRMP construction recipes live in the MethScope lab
journal (`20251216_methscope.org`).

## License

The data in this repository, at every tag (`v1` through `v4`) and on `main`,
are released under the
[Creative Commons Attribution 4.0 International license (CC BY 4.0)](LICENSE).
Files derived from third-party datasets (see Sources) remain subject to the
original datasets' terms as well; cite the original studies when you use them.
The pretrained models on HuggingFace are under the CHOP academic BSD-2 license
(see the [model card](https://huggingface.co/zhou-lab/methscope)), and the
software under the
[methscope-cli license](https://github.com/zhou-lab/methscope-cli/blob/main/LICENSE).

## Sources

The fixtures, the archived MRMP sets and the pretrained models derive from:

- **Loyfer et al. 2023**, *Nature* 613:355-364, human cell-type WGBS atlas
  (GEO GSE186458): `human_hg38_celltypes.cg`,
  `human_hg38_40_celltypes_chr20.cg`, the `hg38_Loyfer*` and
  `hg38_LoyferPlusZhou` sets at `v1`.
- **Liu et al. 2021**, *Nature* 598:120-128, mouse brain single-nucleus
  methylomes (GEO GSE132489; NeMO): `mm10_Liu2021` at `v1`.
- **1000 Genomes Project**: `hg38_1KG` at `v1`.
- **Zhou et al. 2025**, human single-cell methylation atlas: `hg38_Zhou2025`
  and `hg38_LoyferPlusZhou` at `v1`, `human_hg38_immune_mixture.cg`.
  TODO: publication and accession.
- **BLUEPRINT**: immune pools in the `hg38_33celltypes` reference. TODO:
  confirm the release and its access terms.
- `hg38_Sex_20260612` at `v1`. TODO: record the training data.
