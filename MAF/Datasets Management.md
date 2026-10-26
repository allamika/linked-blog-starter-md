# Dataset & Model Management Conventions

Conventions for storing datasets, training results, and deployed models for the fruit detection deep learning pipeline (fruit / stem / calyx detection, multi-fruit: peach, orange, plum, kaki, apple, …).

## Core principles

1. **Raw data is immutable.** Captured images are never renamed, moved, or deleted. Everything derives from them.
2. **Labels know their image.** The labelling format keep a relative link the images. Annotation folder mimic the structure of raw.       
3. **Trainings only use frozen, versioned dataset releases.** Never train on "the current state of a folder". If data or labels change, cut a new release version.
4. **Splits are text files, not folder copies.** `train.txt` / `val.txt` / `test.txt` list image paths.
5. **Every training run records the exact dataset version it used.**
6. **Only promoted models are deployed**, from a registry with a model card and checksums.

## Directory layout

```
datasets/
  _raw/                              # immutable captures, by acquisition campaign
    2026-05-12_peach_lineA_realsense/
      images/
      capture_info.yaml              # camera, site, operator, conditions
  _annotations/                      # live labeling workbench (mutable)
    peach/
      2026-05-12_lineA/              # mirrors the _raw campaign name
        cvat_export.xml              # labeling tool exports
      labeling_status.md             # who's on what, review status
  releases/                          # frozen training-ready snapshots
    peach_det_v3/
      dataset.yaml                   # manifest (see below)
      images/                        # hardlinks/copies from _raw
      labels/                        # one YOLO .txt per image, same basename
      splits/
        train.txt  val.txt  test.txt

trainings/
  2026-07-15_peach_det_v3_yolov8m_baseline/
    config.yaml                      # full hyperparameters, model, augmentations
    weights/best.pt  last.pt
    metrics.json                     # per-class metrics, per split
    notes.md                         # who, why, conclusion

models/                              # registry: deployed models only
  peach_det/
    v5/
      model.onnx                     # deployable format
      model.pt                       # source checkpoint (for finetuning)
      model_card.yaml
      checksum.sha256
    LATEST.txt                       # contains "v5" (pointer, humans/dev only)

classes.yaml                         # single shared class list, all fruits
```

## Naming conventions

| Thing           | Pattern                                       | Example                                    |
| --------------- | --------------------------------------------- | ------------------------------------------ |
| Raw campaign    | `{date}_{fruit}_{location}_{sensor}`          | `2026-05-12_peach_lineA_realsense`         |
| Dataset release | `{fruit}_{task}_v{N}`                         | `peach_det_v3`, `multi_det_v1`             |
| Training run    | `{date}_{dataset}_{model}_{tag}`              | `2026-07-15_peach_det_v3_yolov8m_baseline` |
| Registry model  | `{fruit}_{task}/v{N}` (independent numbering) | `peach_det/v5`                             |

`task` vocabulary: `det` (detection), `seg` (segmentation), `kpt` (keypoints).

## Classes

One shared class list for all fruits, defined once in `classes.yaml`:

```yaml
classes: [fruit, stem, calyx]
# decision: mucron is merged with calyx (see notes)
```

Same class IDs everywhere — this keeps multi-fruit datasets and transfer experiments painless.

## Dataset release contents

- `images/` — the image files (hardlinks into `_raw/` to avoid duplication, or not working just duplicate it).
- `labels/` — one `.csv` for the release,   (`class_id x_center y_center width height`, normalized to [0,1]).
- `splits/` — `train.txt` / `val.txt` / `test.txt` listing image paths.
- `dataset.yaml` — the manifest:

```yaml
name: peach_det_v3
created: 2026-07-10
created_by: mathieu
parent: peach_det_v2
changes: "added 450 images from 2026-06 campaign, fixed 32 calyx labels"
classes: [fruit, stem, calyx]
sources: [2026-05-12_peach_lineA_realsense, 2026-06-20_peach_lineB]
counts: {images: 2140, train: 1712, val: 214, test: 214}
```

## Workflow

```
capture ──> _raw/  (immutable)
                │
label/fix ──> _annotations/  (mutable workbench, labeling tool)
                │  cut release (export + splits + manifest)
                ▼
            releases/{fruit}_{task}_vN  (frozen forever)
                │  train
                ▼
            trainings/{date}_{dataset}_{model}_{tag}
                │  promote (only validated models)
                ▼
            models/{fruit}_{task}/vN  (registry → deployment)
```

Label corrections go to `_annotations/`, then a **new** release is cut. Existing releases are never edited — old trainings stay reproducible.

## Model registry

Only models that ship get promoted (copied, not moved) from `trainings/` into `models/`. Each version folder contains the deployable format, the source checkpoint, and:

`model_card.yaml`:

```yaml
name: peach_det_v5
promoted_from: trainings/2026-07-15_peach_det_v3_yolov8m_baseline
dataset: peach_det_v3
metrics: {mAP50: 0.94, calyx_recall: 0.88}   # on the frozen test split
exported: {format: onnx, opset: 17, input: 640x640}
deployed_to: [FastPackDouble >= 2.3]
promoted_by: mathieu
date: 2026-07-16
```

`checksum.sha256` (standard `sha256sum` format, machine-generated, never hand-edited):

```
a1b2c3d4...  model.onnx
7c8d9e0f...  model.pt
```

Generate at promotion:

```powershell
Get-ChildItem models/peach_det/v5 -Include *.onnx,*.pt -Recurse |
  Get-FileHash -Algorithm SHA256 |
  ForEach-Object { "$($_.Hash.ToLower())  $(Split-Path $_.Path -Leaf)" } |
  Set-Content models/peach_det/v5/checksum.sha256 -Encoding ascii
```

### Verifying what a machine runs

The application computes the SHA256 of the model file at load and logs it:

```csharp
using var stream = File.OpenRead(modelPath);
var hash = Convert.ToHexString(SHA256.HashData(stream));
Log.Info($"Loaded model {Path.GetFileName(modelPath)} sha256={hash}");
```

Compare the logged hash against the registry's `checksum.sha256` to prove a machine runs the exact validated bytes.

### Deployment rules

- Machines pin an **explicit version** (`peach_det/v5`) in their config — never `LATEST.txt`.
- `LATEST.txt` is convenience for humans and dev tooling only.
- TensorRT engines are GPU/version-specific: name them accordingly
  (`model_rtx4060_trt8.6.engine`) or build on-device from ONNX.

## File format conventions

| File | Written by | Format |
|---|---|---|
| Manifests, model cards, configs | humans | YAML |
| Metrics | training code | JSON |
| Notes | humans | Markdown |
| Pointers, checksums | tools | plain text |

## Storage & retention

- Datasets and weights live on the shared NAS / object storage — **never in git**
  (DVC is the natural upgrade path: git-versioned manifests + hashes, bytes on the NAS).
- Promoted models: kept forever.
- Failed/superseded experiment weights: may be pruned after a few months —
  always keep `config.yaml`, `metrics.json`, `notes.md` (tiny, stay useful).

## The one rule that matters most

> **Nobody trains on an unversioned folder.** Need different data? Cut a new release first.
> Two minutes with the manifest template saves the "which data was this model trained on?"
> archaeology entirely.
