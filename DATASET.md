# DocHTML Dataset

Large-scale HTML/CSS document-generation dataset accompanying the CVPR 2026
paper *"AnyDoc: Enhancing Document Generation via Large-Scale HTML/CSS Data
Synthesis and Height-Aware Reinforcement Optimization."* DocHTML is the dataset;
AnyDoc is the model trained on it (released predictions and scores are tagged
with the `anydoc*` model identifiers).

> **Access.** DocHTML and the AnyDoc checkpoints are released under the
> [Adobe Research License](LICENSE.txt) and provided on request. See
> [Accessing the DocHTML Dataset and Model Checkpoints](README.md#accessing-the-dochtml-dataset-and-model-checkpoints)
> in the main README for how to request access. The image assets carry an
> additional restriction — see [Image-asset restrictions](#license-and-image-asset-restrictions).

## What you receive

Upon access, the dataset is delivered as Apache Parquet tables plus sharded
tarballs of the rendered images and per-instance assets. The layout is:

```
data/
  train/*.parquet
  val/*.parquet
  val_sample/*.parquet
  test/*.parquet
  test_1000_intention/*.parquet
  test_1000_screenshot/*.parquet
renders/{split}/{split}-NNNN-of-NNNN.tar     # one PNG render per variation
assets/{split}/{split}-NNNN-of-NNNN.tar      # per-instance asset PNGs + HTML
predictions/{task}/{model}.parquet           # baseline outputs on the benchmarks
scores/vlm_judge.parquet
scores/derendering.parquet
```

Each row in the `data` tables represents one *variation* of a synthesized page.
For every row we publish three task framings (intention, screenshot, element)
and two HTML style formats (normal, abs), for six HTML columns in total (see the
schema below).

## Splits

| split | rows | source |
|---|---|---|
| `train`, `val`, `test` | ~166k / ~16k / ~16k | 80/10/10 page-level split (seed 42) |
| `val_sample` | 1,000 | small subsample of `val` |
| `test_1000_intention` | 1,000 | published benchmark for the intention task |
| `test_1000_screenshot` | 1,000 | published benchmark for the screenshot task |

`val_sample`, `test_1000_intention`, and `test_1000_screenshot` are subsets of
`val`/`test` rematerialized as their own splits for download convenience.
`test_1000_intention` and `test_1000_screenshot` are independent samples that
overlap by 592 idxs.

> **Note on counts.** The row counts above describe the *released* Parquet splits
> (one row per page variation). The headline figure reported in the paper —
> 265,206 samples across 111 categories and 32 styles — counts synthesized
> documents at the sample level.

## Row schema

| column | description |
|---|---|
| `idx` | `{page_uuid}-{variation}` |
| `page_id` | `{page_uuid}` (shared across same-page variations) |
| `variation` | int |
| `width`, `height` | rectified page dimensions (px) |
| `category`, `styles`, `moods`, `topics` | page metadata |
| `intention`, `description`, `score` | page-level descriptors |
| `html` | normal CSS, picsum URL srcs. Target for intention and screenshot tasks. |
| `html_with_assets` | normal CSS, `image_N_HxW.png` srcs (N = filename index on disk) — renderable locally. |
| `html_with_assets_permuted` | normal CSS, `image_K_HxW.png` srcs (K = model-input position) — element task training target. |
| `html_abs`, `html_abs_with_assets`, `html_abs_with_assets_permuted` | absolute-positioned CSS variants of the above. May be null for a small number of idxs that lack abs coverage. |
| `num_images` | count of `<img>` tags |
| `image_filenames` | per-row asset filenames, in N-order (natural sort) |
| `element_image_order` | `K -> N` permutation; the file at model-input position `K` is `image_filenames[element_image_order[K]]` |
| `intention_input` | JSON-encoded prompt for the intention task |
| `screenshot_input` | prompt for the screenshot task (contains `<image>` placeholder) |
| `element_input` | prompt for the element task (lists per-image dimensions in N-order) |

## Media

Renders (one PNG per variation) and assets (per-instance asset PNGs and HTML
files) are shipped as sharded tarballs alongside the Parquet:

```
renders/{split}/{split}-NNNN-of-NNNN.tar
assets/{split}/{split}-NNNN-of-NNNN.tar
```

`assets/{idx}/` contains: `image_N_HxW.png` for each image, plus `raw.html`,
`processed_suffix.html`, and `processed_rename.html` (legacy formats kept for
audit).

## Predictions and scores

```
predictions/{task}/{model}.parquet
scores/vlm_judge.parquet
scores/derendering.parquet
```

`predictions/` contains the model predictions used in the paper's experiments —
the outputs of AnyDoc and the baselines on the `test_1000_*` benchmark splits,
organized by task. `scores/` contains the **VLM-as-judge** scores
(`vlm_judge.parquet`) and the derendering scores (`derendering.parquet`) computed
for those predictions, i.e., the numbers behind the paper's quantitative tables.

## Tasks and targets

To assemble supervised fine-tuning data for the element task (normal CSS), the
input is `element_input` and the target is `html_with_assets_permuted`; the
images are ordered by `element_image_order`:

```
images[K] = assets/{idx}/{image_filenames[element_image_order[K]]}
```

The intention and screenshot tasks use `intention_input` / `screenshot_input` as
input and `html` as the target. The `test_1000_intention` and
`test_1000_screenshot` splits are the published 1k-row evaluation benchmarks.

## License and image-asset restrictions

This dataset is released under the [Adobe Research License](LICENSE.txt). In
addition, the per-instance image assets under `assets/*/image_*.png` were
generated with
[FLUX.1-dev](https://huggingface.co/black-forest-labs/FLUX.1-dev) under the
FLUX.1 [dev] Non-Commercial License v1.1.1. Per that license:

> "You may not use the Output to train, fine-tune or distill a model that is
> competitive with the FLUX.1 [dev] Model or the FLUX.1 Kontext [dev] Model."

You are therefore expressly prohibited from using the distributed image assets to
train, fine-tune, or distill a model that is competitive with the FLUX.1 [dev]
Model or the FLUX.1 Kontext [dev] Model. See
[`FLUX_LICENSE_NOTES.md`](FLUX_LICENSE_NOTES.md) for the full notice.

## Citation

```bibtex
@InProceedings{Lin_2026_CVPR,
    author    = {Lin, Jiawei and Zhu, Wanrong and I Morariu, Vlad and Tensmeyer, Christopher},
    title     = {AnyDoc: Enhancing Document Generation via Large-Scale HTML/CSS Data Synthesis and Height-Aware Reinforcement Optimization},
    booktitle = {Proceedings of the IEEE/CVF Conference on Computer Vision and Pattern Recognition (CVPR)},
    month     = {June},
    year      = {2026},
    pages     = {626-635}
}
```
