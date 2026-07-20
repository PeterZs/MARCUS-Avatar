# Third-Party Licenses

MARCUS-Avatar's own code is released under the MIT License (see [LICENSE](LICENSE)).
The `third_party/` directory vendors code from several external research
projects, which retain their original licenses. Each vendored subdirectory
has its own `LICENSE` file; this document is an index plus a summary of one
remaining open item that should be confirmed before public release.

## Vendored components

| Directory | Upstream project | License | Copyright |
|---|---|---|---|
| `third_party/face_box/` (excl. `retinaface/`) | [sicxu/Deep3DFaceRecon_pytorch](https://github.com/sicxu/Deep3DFaceRecon_pytorch) | MIT | (c) 2022 Sicheng Xu |
| `third_party/face_box/retinaface/` | [ternaus/retinaface](https://github.com/ternaus/retinaface) | MIT | (c) 2020 Vladimir Iglovikov |
| `third_party/landmark68/` | sicxu/Deep3DFaceRecon_pytorch | MIT | (c) 2022 Sicheng Xu |
| `third_party/mtcnn/` | sicxu/Deep3DFaceRecon_pytorch | MIT | (c) 2022 Sicheng Xu |
| `third_party/skin_mask/` | sicxu/Deep3DFaceRecon_pytorch | MIT | (c) 2022 Sicheng Xu |
| `third_party/face_parsing/` | [zllrunning/face-parsing.PyTorch](https://github.com/zllrunning/face-parsing.PyTorch) | MIT | (c) 2019 zll |
| `third_party/dml_csr/` | DML_CSR (CVPR 2022), vendored via [deepinsight/insightface](https://github.com/deepinsight/insightface/tree/master/parsing/dml_csr) | Apache-2.0 (per file header) | 2021 Qingping Zheng |
| `third_party/dml_csr/utils/encoding.py` (partial) | [vacancy/Synchronized-BatchNorm-PyTorch](https://github.com/vacancy/Synchronized-BatchNorm-PyTorch) | MIT | (c) 2018 Jiayuan Mao |
| `third_party/face_box/preprocess_sub.py` (`face_align` routine, partial) | [deepinsight/insightface](https://github.com/deepinsight/insightface) | MIT (per repo README) | not stated in upstream |

Data/model assets referenced by these projects (e.g. the Basel Face Model
used by Deep3DFaceRecon_pytorch, and insightface's own training data/models)
carry separate non-commercial-only terms from their original providers. That
restriction applies to those downloadable data/model files, **not** to the
code vendored here — MARCUS-Avatar does not redistribute BFM or insightface
model/data files, only code that operates on user-supplied images.

## Resolved item

`third_party/face_box/retinaface/box_utils.py`'s `decode` function carries a
comment crediting [Hakuyume/chainer-ssd](https://github.com/Hakuyume/chainer-ssd),
which has no license of its own. Direct byte-for-byte comparison confirms the
identical function (same code, docstring, and variable names) is also
published under MIT in
[amdegroot/ssd.pytorch](https://github.com/amdegroot/ssd.pytorch) and
[biubug6/Pytorch_Retinaface](https://github.com/biubug6/Pytorch_Retinaface).
This is a copy-paste lineage (chainer-ssd → ssd.pytorch → Pytorch_Retinaface),
not independent reimplementation, so provenance for this specific function is
traced through the MIT-licensed repos rather than relying on chainer-ssd's
unlicensed original. See `third_party/face_box/retinaface/LICENSE` for the
attribution note.

## Open items — needs a decision before public release

1. **`third_party/dml_csr/` license mismatch**: the vendored file
   `networks/dml_csr.py` states "Licensed under the Apache License,
   Version 2.0" in its own header, but the repo it currently lives in
   (deepinsight/insightface) has no root LICENSE file and its README
   claims a blanket MIT license for "the code of InsightFace" in general.
   This document treats the more specific, explicit Apache-2.0 header as
   authoritative for this subtree (folding code into a differently-licensed
   monorepo doesn't retroactively relicense the original author's grant),
   but this is worth confirming with the DML_CSR/insightface maintainers if
   MARCUS-Avatar's release will be scrutinized closely (e.g. by legal or a
   downstream commercial user).
