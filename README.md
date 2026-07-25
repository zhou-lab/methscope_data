# MethScope Data — model catalog, test fixtures, reproducibility archive

Companion data repo for [**methscope-cli**](https://github.com/zhou-lab/methscope-cli).
The **runnable model bundles are hosted on HuggingFace** — too large for git —
at [`zhou-lab/methscope`](https://huggingface.co/zhou-lab/methscope). This repo
holds what stays in git: the model **catalog** (below), small **test fixtures**
(`test/`), and the MRMP-construction **reproducibility archive** (tag `v1`).

Each model is a single self-contained bundle (`.ubjx` classifier, `.updecx`
upscale decoder, or `.refx` deconvolution reference) that already contains its
MRMP feature definition (and labels / cell-type signature), so a query `.cg`
runs directly:

```sh
# fetch models from HuggingFace (--local-dir models to grab all)
hf download zhou-lab/methscope hg38_celltype.ubjx --local-dir models

methscope predict query.cg   models/hg38_celltype.ubjx     # cell-type / label
methscope deconv  mixture.cg models/hg38_65celltypes.refx   # cell-type proportions (NNLS)
methscope upscale -o out.cg  models/hg38_wg.updecx query.cg # genome-wide CpG upscaling
methscope inspect            models/hg38_sex.ubjx           # framework, labels, features
```

Small query `.cg` fixtures used by the
[methscope-cli](https://github.com/zhou-lab/methscope-cli) README/examples live
here in `test/` (4 typed cells, a simulated deconvolution mixture, and an
upscale input + truth), plus one small **reference** so `mrmp-build` is
runnable without the 2.3 GB atlas:

`test/human_hg38_40_celltypes_chr20.cg` — 40 Loyfer cell types, chr20 only
(773,477 CpGs, 40 MB). `mrmp-build` packs a pattern as a base-3 `uint64` and
3^40 < 2^64 < 3^41, so 40 samples is the hard ceiling; chr20 keeps it small
enough to ship. Rebuild it from the lab store with:

```sh
L=/mnt/isilon/zhou_lab/projects/20230727_all_public_WGBS/hg38/2023_Loyfer.cg
# the shipped .cg.idx *is* the sample list -- one per cell type, every other
# type across the atlas, 40 of Loyfer's 82
cut -f1 test/human_hg38_40_celltypes_chr20.cg.idx > pick40.txt
yame subset $L $(tr '\n' ' ' < pick40.txt) \
  | yame rowsub -B 15511118_16284595 - > human_hg38_40_celltypes_chr20.cg
yame index -s pick40.txt human_hg38_40_celltypes_chr20.cg
# the chr20 row range comes from ~/references/hg38/KYCGKB_hg38/cpg_nocontig.cr
```

It yields 116,450 distinct patterns over 773,477 CpGs (15.0% PNA).

`test/SHA256SUMS` records the digest of every fixture (`sha256sum -c
SHA256SUMS`). It is a convenience for anyone verifying by hand: `methscope
fetch` does not read it, because each digest is compiled into the tool's
registry, so a download is checked against something the binary already held
rather than against a file served by the same host.

> The full MRMP definition sets (`*.cm`), pattern-definition tables (`*_def*`),
> and deconvolution references (`*_ref.rds`) are archived at tag **`v1`**
> (`git checkout v1`). MRMP construction recipes live in the MethScope lab
> journal (`20251216_methscope.org`).

## Models — [huggingface.co/zhou-lab/methscope](https://huggingface.co/zhou-lab/methscope)

| file | task | framework | labels / cell types |
|------|------|-----------|---------------------|
| `hg38_wg.updecx` | whole-genome CpG upscaling | `UPDEC2` | all 29,401,795 hg38 CpGs; the primary upscaler (2.8 GB) |
| `hg38_10k1.updecx` | CpG upscaling (single block) | MLP decoder | block 10k1 (10,000 CpGs); small demo model |
| `hg38_celltype.ubjx` | cell-type annotation | xgboost | 62 human cell types (Alpha, ASC, AT1/AT2, B Mem/Naive/Plasma, Beta, … NK CD16/CD56, ODC, OPC, T subsets, …) |
| `mm10_celltype.ubjx` | cell-type annotation | xgboost | 41 mouse-brain cell types (ASC, CA1, CA3, DG, ODC, OPC, MGC, IT-L2/3…L6, PT-L5, …) |
| `hg38_sex.ubjx` | sex prediction | logistic | Female, Male (XCI `Xa_hi`/`Xa_lo` markers) |
| `hg38_65celltypes.refx` | cell-type deconvolution (NNLS) | refx | 65 cell types = 58 Zhou + 7 Loyfer organ/blood (Hepatocyte, Granulocyte, Adipocyte, Kidney_Tubular, Kidney_Podocyte, Erythrocyte_prog, Thyroid); **split MRMP, 15,300 patterns** |

`methscope inspect <model>` prints the exact framework, full label list, and (for
linear models) the per-feature weights.

### Notes
- **Classifiers carry a required framework `kind` mark** (`xgboost` / `threshold` /
  `logistic`); `predict` rejects an unmarked bundle. Upscale decoders (`.updecx`)
  are run by `upscale`, and deconvolution references (`.refx`, `kind=refx`) by
  `deconv`; both need no framework mark.
- **hg38_wg.updecx** is the whole-genome upscaler (unified `UPDEC2`, one
  processing unit per MRMP membership). It reconstructs dense methylation over
  all 29.4M hg38 CpGs from a sparse query; pure-C inference (~2 s/sample). The
  single-block `hg38_10k1.updecx` is retained as a small demo. Training + the
  external-cohort validation are recorded in `20251216_methscope.org`.
- **hg38_65celltypes.refx** is a whole-body deconvolution reference (cell-type ×
  pattern β signature + its MRMP), built on a **deterministic, reproducible**
  binstring MRMP (no random tie-break, ambiguity-filtered) over 58 Zhou single-cell
  types + 7 Loyfer bulk-WGBS organ/blood types (liver, kidney tubular, kidney
  podocyte, adipose, neutrophil, erythroid, thyroid), for whole-body / cfDNA
  deconvolution. The MRMP is **split**
  (each recurrence pattern's CpGs chunked into 1000-CpG genomic groups → 15,300
  patterns), which greatly improves **sparse / low-coverage** deconvolution: on 60
  simulated immune mixtures downsampled to 2¹⁶ (65k) binarized CpGs it reaches
  r² ≈ 0.87 (vs ≈ 0.6 for the unsplit MRMP), while full-coverage stays r² ≈ 0.997
  (Macrophage 0.997). `deconv` uses **all** patterns — there is no `-p` /
  `--var-threshold`, since a variance filter would drop cell-type-specific markers
  (which by construction have low cross-cell-type variance) and a leading-N cutoff
  discards real low-recurrence signal. Build recipe: `20251216_methscope.org` →
  `hg38_65celltypes.refx` (split step in `split_mrmp.sh`).
- **hg38_sex** is the `logistic` model — on an independent cohort
  (2018_Zhou) it reaches ~95.8% (vs a manual β(Xa_hi)−β(Xa_lo) score); the misses
  are XCI-disrupted samples (leukemias, tumors, cell lines, PGCs, oocytes). The
  interpretable `threshold` variant is derivable by hand, so it is not shipped.
