# MethScope Data — models

Trained models for [**MethScope2**](https://github.com/zhou-lab/MethScope2). Each
model is a single self-contained bundle (`.ubjx` classifier or `.updecx` upscale
decoder) that already contains its MRMP feature definition and class labels, so a
query `.cg` can be run directly:

```sh
methscope2 predict query.cg models/hg38_celltype.ubjx   # cell-type / label
methscope2 upscale  models/hg38_10k1.updecx query.cg    # CpG-level upscaling
methscope2 inspect  models/hg38_sex.ubjx                # framework, labels, features
```

> The full MRMP definition sets (`*.cm`), pattern-definition tables (`*_def*`), and
> deconvolution references (`*_ref.rds`) are archived at tag **`v1`**
> (`git checkout v1`); `main` holds only the runnable models. MRMP construction
> recipes live in the MethScope lab journal (`20251216_methscope.org`).

## Models (`models/`)

| file | genome | task | framework | MRMP (features) | labels |
|------|--------|------|-----------|-----------------|--------|
| `hg38_celltype.ubjx` | hg38 | cell-type annotation | xgboost | `hg38_Zhou2025` (recurrence P1…P1000) | 62 human cell types (Alpha, ASC, AT1/AT2, B Mem/Naive/Plasma, Beta, … NK CD16/CD56, ODC, OPC, T subsets, …) |
| `mm10_celltype.ubjx` | mm10 | cell-type annotation | xgboost | `mm10_Liu2021` (recurrence P1…P1000) | 41 mouse-brain cell types (ASC, CA1, CA3, DG, ODC, OPC, MGC, IT-L2/3…L6, PT-L5, …) |
| `hg38_sex.ubjx` | hg38 | sex prediction | logistic | `hg38_Sex_20260612` — XCI markers, states `Xa_hi`/`Xa_lo` (not `P1…Pn`) | Female, Male |
| `hg38_10k1.updecx` | hg38 | CpG-level upscaling | MLP decoder | 101-pattern block MRMP | block 10k1 (10 000 CpGs) |

`methscope2 inspect <model>` prints the exact framework, full label list, and (for
linear models) the per-feature weights.

### Notes
- **Classifiers carry a required framework `kind` mark** (`xgboost` / `threshold` /
  `logistic`); `predict` rejects an unmarked bundle. Upscale decoders (`.updecx`)
  are run by `upscale` and need no mark.
- **hg38_sex** is the `logistic` model — on an independent cohort
  (2018_Zhou) it reaches ~95.8% (vs a manual β(Xa_hi)−β(Xa_lo) score); the misses
  are XCI-disrupted samples (leukemias, tumors, cell lines, PGCs, oocytes). The
  interpretable `threshold` variant is derivable by hand, so it is not shipped.

## Citation

Hongxiang Fu, Chin Nien Lee, Cameron Cloud, Hao Xu, Yanxiang Deng, Wanding Zhou,
MethScope: Ultra-fast Analysis of Sparse DNA Methylome via Recurrent Pattern Encoding.
