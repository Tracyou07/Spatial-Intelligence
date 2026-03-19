# Spatial Intelligence 论文简读索引

> 按发布时间排序 | 论文 PDF 在 `papers/` | 笔记在 `notes/`

## 演进脉络

```
DPT (2021) — ViT 用于 dense prediction
  ↓
DUSt3R (2023.12) — 图像对 → pointmap，无需相机参数
  ↓
MASt3R (2024.06) — + 特征匹配头，MASt3R-SfM
  ↓
Human3R (2024.10) — + 人体编码器，端到端人场重建
  ↓
JOSH (2025.01) — 联合优化 + 接触约束，非端到端
  ↓
CUT3R (2025.01) — Persistent State，视频流在线推理
  ↓
MUST3R (2025.03) — 多视图记忆机制，O(N) 规模扩展
  ↓
VGGT (2025.03) — 单模型四合一：位姿+深度+点云+轨迹，CVPR Best Paper
```

---

| # | 方法 | 时间 | 笔记 | GitHub |
|---|------|------|------|--------|
| 1 | DPT | ICCV 2021 | [notes/01_DPT_2021.md](notes/01_DPT_2021.md) | [isl-org/DPT](https://github.com/isl-org/DPT) |
| 2 | DUSt3R | 2023.12 | [notes/02_DUSt3R_2023.md](notes/02_DUSt3R_2023.md) | [naver/dust3r](https://github.com/naver/dust3r) |
| 3 | MASt3R | 2024.06 | [notes/03_MASt3R_2024.md](notes/03_MASt3R_2024.md) | [naver/mast3r](https://github.com/naver/mast3r) |
| 4 | Human3R | 2024.10 | [notes/04_Human3R_2024.md](notes/04_Human3R_2024.md) | [fanegg/Human3R](https://github.com/fanegg/Human3R) |
| 5 | JOSH | 2025.01 | [notes/05_JOSH_2025.md](notes/05_JOSH_2025.md) | [genforce/JOSH](https://github.com/genforce/JOSH) |
| 6 | CUT3R | 2025.01 | [notes/06_CUT3R_2025.md](notes/06_CUT3R_2025.md) | [CUT3R/CUT3R](https://github.com/CUT3R/CUT3R) |
| 7 | MUST3R | 2025.03 | [notes/07_MUST3R_2025.md](notes/07_MUST3R_2025.md) | [naver/must3r](https://github.com/naver/must3r) |
| 8 | VGGT 🏆 | 2025.03 | [notes/08_VGGT_2025.md](notes/08_VGGT_2025.md) | [facebookresearch/vggt](https://github.com/facebookresearch/vggt) |
