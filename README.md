# MARCUS-Avatar

Official implementation of **"Monocular Avatar Reconstruction via Cascaded Diffusion Priors and UV-Space Differentiable Shading"** (ECCV 2026).

[[Project Page](https://luh1124.github.io/MARCUS-Avatar-Projectpage/)] · [[Paper](https://arxiv.org/abs/2606.28144)] · [[HF Weights](https://huggingface.co/luh0502/MARCUS-Avatar)] · [[HF Data](https://huggingface.co/datasets/luh0502/Marcus-avatar-data)]

## Overview

MARCUS reconstructs high-fidelity, relightable 3D face avatars from a single in-the-wild portrait image. The released inference code generates UV-space PBR assets and exportable 3D avatar files from either a single image or a folder of images.

![MARCUS-Avatar teaser](assets/teaser.png)

## Installation

### Option 1: pip / conda

```bash
git clone https://github.com/luh1124/MARCUS-Avatar.git
cd MARCUS-Avatar

conda create -n marcus python=3.10
conda activate marcus

# Choose the PyTorch build that matches your CUDA runtime.
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu121

pip install -r requirements.txt

# basicsr 1.4.2 (pulled in by realesrgan) is incompatible with torchvision>=0.18;
# patch the installed file once (also noted at the bottom of requirements.txt):
sed -i 's|torchvision\.transforms\.functional_tensor|torchvision.transforms.functional|' \
  "$(python -c 'import basicsr, os; print(os.path.dirname(basicsr.__file__))')/data/degradations.py"
```

### Option 2: pixi

If you use [pixi](https://pixi.sh/):

```bash
pixi install
pixi run download-weights
pixi run app
```

## Download Weights

The repository does not store model weights or topology assets in git. Download them from Hugging Face:

```bash
python download_weights.py
```

This restores:

```text
ckpts/
assets/topo/
```

By default, the runtime loads the base diffusion model `meituan-longcat/LongCat-Image-Edit` directly from Hugging Face. `batch_infer.py` additionally loads `fancyfeast/llama-joycaption-beta-one-hf-llava` for captioning (the Gradio demo uses editable default prompts instead).

If you keep local copies or run in an offline environment, override the paths with environment variables.

## Training Data

The relit renders and UV unwraps used to train MARCUS are released as
[`luh0502/Marcus-avatar-data`](https://huggingface.co/datasets/luh0502/Marcus-avatar-data).
Everything is stored as uncompressed [WebDataset](https://github.com/webdataset/webdataset)
tar shards, keyed by the source dataset's image id:

| config | source | samples | shards | size |
|---|---|---|---|---|
| `ffhq1024_rendered` | FFHQ-1024 | 69,999 | 140 | ~755 GB |
| `ffhq1024_unwrap_texture` | FFHQ-1024 | 69,999 | 35 | ~194 GB |
| `celebamask_hq_rendered` | CelebAMask-HQ | 29,994 | 60 | ~330 GB |
| `celebamask_hq_unwrap_texture` | CelebAMask-HQ | 29,994 | 15 | ~83 GB |

`rendered` holds three HDRI relightings per sample (`render_hdri_{0,1,2}.png`) with their
baked textures and environment maps, plus an evenly lit bake. `unwrap_texture` holds the
fitted meshes, the unwrapped UV texture, and the 3DMM coefficients / landmarks / transform
tensors. Per-key details are in the dataset card.

```python
from datasets import load_dataset

ds = load_dataset("luh0502/Marcus-avatar-data", "ffhq1024_rendered",
                  split="train", streaming=True)
sample = next(iter(ds))
print(sample["__key__"], sample["render_hdri_0.png"].size)
```

Shard-to-id ranges are listed in each subset's `shard_index.json`, so a single id range can
be fetched without pulling the whole config. The source datasets are non-commercial research
licenses (FFHQ is CC BY-NC-SA 4.0 with per-photograph Flickr rights; CelebAMask-HQ is
research-only) and those terms carry over to this derived data.

## Runtime Paths

Optional environment variables:

```bash
export HF_REPO_ID="luh0502/MARCUS-Avatar"
export CKP_DIR="./ckpts"
export TOPO_DIR="./assets/topo"
export BASE_MODEL_PATH="meituan-longcat/LongCat-Image-Edit"
export JOY_CAPTION_MODEL="fancyfeast/llama-joycaption-beta-one-hf-llava"
export BLENDER_PATH="/path/to/blender"
```

## Usage

### Gradio Demo

```bash
python app.py
```

The demo opens a local Gradio interface for single-image avatar reconstruction.

### Batch Inference

```bash
python batch_infer.py ./examples -o ./outputs
```

The input can be either a single image or a folder. See all options with:

```bash
python batch_infer.py --help
```

Typical outputs include reconstructed meshes, UV textures, PBR material maps, and optional `.glb` / `.blend` files.

### Blender (optional, only for `.blend` export)

Blender is **not installed automatically**. When exporting `.blend`, the app looks for an executable in this order: `$BLENDER_PATH`, `blender` on `PATH`, `/usr/bin/blender`. If none is found, the export fails and the status box reports `Blender executable was not found; set the BLENDER_PATH environment variable`. `.glb` export does not require Blender.

Install options:

```bash
# Option A: via pixi / conda-forge (available on PATH inside `pixi shell` / `pixi run`)
pixi add blender

# Option B: download from https://www.blender.org/download/ and point the app at it
export BLENDER_PATH="/path/to/blender"
```

## Repository Structure

```
MARCUS-Avatar/
├── app.py                  # Gradio demo entry point
├── batch_infer.py          # Single-image / folder batch inference
├── download_weights.py     # Hugging Face weight downloader
├── runtime_paths.py        # Shared runtime path configuration
├── pipeline.py             # Pipeline utilities and programmatic inference helpers
├── adjust_mask.py          # CLI utility to erode/dilate/open/close masks
├── inplace_abn.py          # Pure-Python fallback for the optional inplace_abn extension
├── longcat_image/          # Core model, preprocessing, reconstruction, texture, and render code
├── third_party/            # Vendored face detection / parsing helpers
├── utils/                  # Image I/O plus GLB / Blender export helpers
├── styles/                 # Gradio UI styling
├── examples/               # Example input images
├── requirements.txt        # pip dependencies for inference
└── pixi.toml               # optional pixi environment
```

Large runtime files are downloaded separately and ignored by git:

```text
ckpts/
assets/topo/
outputs/
```

Local maintenance, evaluation, and experimental helper scripts are intentionally excluded from the public inference release.

## License

This project is released under the MIT License (see [LICENSE](LICENSE)).
Code vendored under `third_party/` retains its own upstream licenses — see
[THIRD_PARTY_LICENSES.md](THIRD_PARTY_LICENSES.md) for details.

## Citation

```bibtex
@inproceedings{li2026marcus,
  title={Monocular Avatar Reconstruction via Cascaded Diffusion Priors and UV-Space Differentiable Shading},
  author={Li, Hong and Meng, Minqi and Liang, Yanjun and Ye, Chongjie and Chen, Houyuan and Xiao, Weiqing and Guo, Xianda and Lei, Guojun and Liu, Xuhui and Yang, Chaojie and Peng, Yanlun and Zhao, Hao and Zhang, Baochang},
  booktitle={European Conference on Computer Vision (ECCV)},
  year={2026}
}
```
