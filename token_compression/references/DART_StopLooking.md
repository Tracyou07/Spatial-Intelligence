---
title: Stop Looking for Important Tokens — DART
arxiv: 2502.11494
venue: Arxiv 2025-02
github: https://github.com/EasonAI-5589/paper-reading/tree/main/MLLM-Token-Compression/%5BArxiv%202502.11494%5D%20StopLooking
---

# DART — Duplication-Aware Reduction of Tokens

## 核心观点

**反直觉结论**：不要找"重要" token 来保留，而是找"重复" token 来删除。

传统 importance-based 方法的四个致命缺陷：
1. **静态 importance score**：移除 token 后其他 token 的重要性会变化，原始分数失效
2. **FlashAttention 不兼容**：需要暴露 attention score，必须关闭硬件加速，得不偿失
3. **位置偏差**：序列末尾 token 的 attention score 虚高，与实际信息量无关
4. **不如随机**：部分 importance 方法效果还不如随机 token 保留

## DART 方法

两步走：

```
Step 1: Pivot 选取（约 2% 的 token）
        → norm-based / random / attention-based 均可，差异仅 0.2%

Step 2: 去重检测
        → 计算所有 token 与 pivot 的 cosine similarity
        → 保留：R = P ∪ {tokens where min_similarity_to_pivots ≤ ε}
        → 删除：与任一 pivot 高度相似的 token（冗余）
```

额外开销 < 0.08 秒，无需访问 attention score，完全兼容 FlashAttention。

## 实验结果

| 模型 | Token 压缩率 | 性能保留 | 推理加速 |
|---|---|---|---|
| LLaVA-1.5-7B | 88.9% | 93.7% | 1.99× |
| LLaVA-1.5-13B | 88.9% | 94.7% | — |

- 超越 SparseVLM（第二名）2.2%（激进压缩下）
- Prefilling 加速 2.99×

## 关键发现

- **Pivot 选取几乎不影响结果**："重要" vs "不重要" pivot 差异仅 0.2% → 证明冗余比重要性更关键
- **最优剪枝层**：第 10-20 层剪枝效果最好，过深层剪枝（去掉所有视觉 token）仅下降 0.1-1.6%
- **跨模态 pivot**：同时用视觉 + 文本 token 做 pivot 效果优于单模态
- **减少幻觉**：在某些层剪枝后 POPE 指标甚至超过原始模型

## 理论保证

在 Lipschitz 连续假设下，输出误差上界：

```
‖f(X) - f(R)‖ ≤ K√(2(1-ε))B
```

误差只与压缩阈值 ε 相关，与具体选哪些 token 无关 → 验证了方法的鲁棒性。

## 局限性

- 需要访问模型中间层（不适用于 GPT/Claude 等闭源模型）
- 对 OCR / 文档类内容效果较弱（token 本身重复度低）

## 对 VGGT 的启发

- 多视图场景跨帧冗余大 → DART 的去重逻辑天然契合
- 但 VGGT 有 dense prediction head，需配合 unmerge 机制使用
- 剪枝位置建议放在 Aggregator 第 10-20 层
