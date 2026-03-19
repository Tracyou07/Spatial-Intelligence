---
title: VGGT Token Compression — 研究思路
date: 2026-03-20
status: 探索阶段
---

# VGGT Token Compression 研究思路

## 动机

VGGT 处理多视图图像时，token 数量随帧数线性增长：
- N帧 × P patches（例：100帧 × 256 = 25,600 tokens）
- Aggregator 做全局跨帧注意力，复杂度 O(N²)，帧数多时极慢

## 核心挑战

**VGGT 与普通 MLLM token compression 的关键区别：**

```
N帧 × P patches
      ↓
  Aggregator（跨帧全局注意力）← 瓶颈
      ↓
  多个并行 Head（dense 输出）
  ├→ depth_head   → 每个 patch 都需要深度值
  ├→ point_head   → 每个 patch 都需要 3D 点
  ├→ camera_head  → 每帧输出位姿（非 dense）
  └→ track_head   → 跨帧轨迹
```

**约束**：depth_head / point_head 是 dense prediction，需要每个 patch token 都有对应输出，
不能像普通 MLLM 那样直接丢弃 token（否则输出分辨率下降）。

## 推荐方案

### 方案一：Token Merging（ToMe）【最推荐】

- **思路**：Bipartite soft matching，合并相似 token，decode 时 unmerge 恢复原始位置
- **优势**：merge/unmerge 机制保留 spatial identity，不破坏 dense head 输出分辨率
- **插入位置**：Aggregator 中间层（第 10-20 层之间）
- **参考**：[facebookresearch/ToMe](https://github.com/facebookresearch/ToMe)

```
Aggregator Layer i:
  tokens → [bipartite matching] → merged tokens → attention → ...
                                                        ↓
Head decode 前：unmerge → 恢复完整分辨率 → dense 输出
```

### 方案二：DART 思路 + 跨帧去重

- **思路**：VGGT 多视图场景天然存在跨帧冗余（同一物理点出现在多帧），用 cosine similarity 检测并移除重复 token
- **与 DART 不同**：重点去除**跨帧重复**而非帧内重复
- **插入位置**：Aggregator 中间层，不在输入层
- **dense head 恢复**：被压缩的 token 用最近邻插值或对应 pivot 的值填充
- **优势**：实现简单，不需要修改模型结构

### 方案三：帧级筛选 + Patch 级压缩（两阶段）

适用于帧数极多（>50 帧）场景：

1. **帧级**：基于帧间 feature similarity 选关键帧，去掉冗余帧
2. **Patch 级**：对保留帧用 ToMe/DART 进一步压缩

### 方案四：Aggregator 中间层直接剪枝

- 参考 DART 实验结论：在第 10-20 层剪枝效果最好，甚至能减少噪声
- 对 camera_head（非 dense）几乎无影响
- 对 depth/point head 有一定影响，需要评估

## 方案对比

| 方案 | 适用场景 | 实现难度 | 对 dense 精度影响 |
|---|---|---|---|
| ToMe in Aggregator | dense 输出场景 | 中 | 小（可 unmerge） |
| DART 跨帧去重 | 帧数很多时 | 低 | 中 |
| 帧级筛选 | >50 帧输入 | 低 | 视帧间相似度 |
| 中间层直接剪枝 | 只加速 attention | 低 | 几乎无 |

## 下一步

- [ ] 阅读 VGGT Aggregator 源码，确定 ToMe 插入位置
- [ ] 实现 ToMe + unmerge，在 depth 精度 vs 速度上做 ablation
- [ ] 测试跨帧 cosine similarity 的实际分布（验证跨帧冗余假设）
- [ ] Baseline：VGGT 原始推理速度 vs 压缩后速度

## 参考方法

见 `references/` 文件夹：
- `DART_StopLooking.md` — Arxiv 2502.11494，去重为主的 MLLM token 压缩
