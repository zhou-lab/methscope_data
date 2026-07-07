# MethScope Data — models

Trained models for [**methscope-cli**](https://github.com/zhou-lab/methscope-cli). Each
model is a single self-contained bundle (`.ubjx` classifier, `.updecx` upscale
decoder, or `.refx` deconvolution reference) that already contains its MRMP feature
definition (and labels / cell-type signature), so a query `.cg` can be run directly:

```sh
methscope predict query.cg  models/hg38_celltype.ubjx      # cell-type / label
methscope deconv  mixture.cg models/hg38_65celltypes.refx   # cell-type proportions (NNLS)
methscope upscale  models/hg38_10k1.updecx query.cg        # CpG-level upscaling
methscope inspect  models/hg38_sex.ubjx                    # framework, labels, features
```

Small query `.cg` fixtures used by the [methscope-cli](https://github.com/zhou-lab/methscope-cli)
README tests live in `test/` (4 typed cells, a simulated deconvolution mixture, and
an upscale input + truth).

> The full MRMP definition sets (`*.cm`), pattern-definition tables (`*_def*`), and
> deconvolution references (`*_ref.rds`) are archived at tag **`v1`**
> (`git checkout v1`); `main` holds only the runnable models. MRMP construction
> recipes live in the MethScope lab journal (`20251216_methscope.org`).

## Models (`models/`)

| file | task | framework | labels / cell types |
|------|------|-----------|---------------------|
| `hg38_celltype.ubjx` | cell-type annotation | xgboost | 62 human cell types (Alpha, ASC, AT1/AT2, B Mem/Naive/Plasma, Beta, … NK CD16/CD56, ODC, OPC, T subsets, …) |
| `mm10_celltype.ubjx` | cell-type annotation | xgboost | 41 mouse-brain cell types (ASC, CA1, CA3, DG, ODC, OPC, MGC, IT-L2/3…L6, PT-L5, …) |
| `hg38_sex.ubjx` | sex prediction | logistic | Female, Male (XCI `Xa_hi`/`Xa_lo` markers) |
| `hg38_10k1.updecx` | CpG-level upscaling | MLP decoder | block 10k1 (10 000 CpGs) |
| `hg38_65celltypes.refx` | cell-type deconvolution (NNLS) | refx | 65 cell types = 58 Zhou + 7 Loyfer organ/blood (Hepatocyte, Granulocyte, Adipocyte, Kidney_Tubular, Kidney_Podocyte, Erythrocyte_prog, Thyroid); **split MRMP, 15,300 patterns** |

`methscope inspect <model>` prints the exact framework, full label list, and (for
linear models) the per-feature weights.

### Notes
- **Classifiers carry a required framework `kind` mark** (`xgboost` / `threshold` /
  `logistic`); `predict` rejects an unmarked bundle. Upscale decoders (`.updecx`)
  are run by `upscale`, and deconvolution references (`.refx`, `kind=refx`) by
  `deconv`; both need no framework mark.
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

## Citation

Hongxiang Fu, Chin Nien Lee, Cameron Cloud, Hao Xu, Yanxiang Deng, Wanding Zhou,
MethScope: Ultra-fast Analysis of Sparse DNA Methylome via Recurrent Pattern Encoding.
