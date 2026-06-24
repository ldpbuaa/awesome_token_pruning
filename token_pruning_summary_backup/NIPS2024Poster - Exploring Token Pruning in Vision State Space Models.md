# 论文总结：Exploring Token Pruning in Vision State Space Models

## 1. 研究动机
- 状态空间模型(State Space Models, SSMs)相比Transformer中的注意力模块具有线性计算复杂度的优势，已被应用于视觉任务作为一种新型强大的视觉基础模型
- 现有针对Vision Transformer (ViT)的token剪枝技术无法直接有效应用于SSMs-based视觉模型
- 直接应用ViT token剪枝技术会导致显著的性能下降，且这种性能损失无法通过大量微调恢复
- 提高SSMs的计算效率对促进实时应用具有重要意义

## 2. 解决问题
- 论文致力于解决的核心问题：如何在保持SSMs计算效率的同时，设计一种有效的token剪枝方法，避免直接应用ViT token剪枝技术导致的显著性能下降
- 具体技术难题：SSMs基于扫描路径处理输入，每个token对相邻token敏感，而ViTs的注意力机制计算的是目标token与图像中所有其他token之间的相关性，消除了对相邻token的敏感性
- 直接应用token剪枝会破坏SSM扫描中的原始token位置和邻域关系，导致显著精度下降

## 3. 现象分析
- 观察到直接将ViT的token剪枝技术应用于SSMs-based视觉模型时，会出现以下现象：
  - 零样本性能显著下降（在ViM-S上超过68%的准确率下降）
  - 即使经过大量微调，性能也无法从token剪枝中恢复（与原始准确率仍有5.7%的差距）
  - 这种性能下降是永久性的，无法通过微调完全恢复
- 深入分析发现SSMs的计算模式具有独特的扫描路径特性，每个token对相邻token高度敏感
- 传统ViT token剪枝中，剩余token会被重新标记，破坏了原始邻域关系，而SSMs的扫描机制对这种位置变化极为敏感

## 4. 主要方法
- **剪枝感知的隐藏状态对齐方法(Pruning-Aware Hidden State Alignment)**：
  - 对剩余token的隐藏状态：保持当前token和之前所有剩余token的依赖关系
  - 对被剪枝token的隐藏状态：不简单移除，而是使用前一个未剪枝邻居的状态，保持原始token的顺序位置和邻域关系
  - 这种方法避免了传统token剪枝中剩余token重新标记导致的邻域关系破坏

- **基于重要性评估的token剪枝(Token Pruning based on Importance Evaluation)**：
  - 利用SSM的输出来评估token的重要性
  - 通过聚合所有通道的裁剪值作为token重要性指标：$S = \sum_{d=1}^{D'} [\max(0, y_{m,d})]$
  - 根据重要性排序，剪除不太重要的token

- **高效实现和实际加速方法**：
  - 提出了"剪枝感知的隐藏状态对齐"内核，利用位置图指导SSM操作
  - 在扫描过程中，根据token的剩余/剪枝状态切换计算模式
  - 其他模型部分可以直接通过减少token数量来加速

## 5. 数据与实验
- **数据集**：
  - ImageNet-1K：用于图像分类任务
  - COCO 2017：用于目标检测和实例分割任务
  - ADE20K：用于语义分割任务

- **对比的基线模型**：
  - EViT token pruning方法
  - 原始的SSMs-based模型（ViM-T, ViM-S, PlainMamba-L1, PlainMamba-L2, PlainMamba-L3）

- **核心实验结果**：
  - 在ImageNet-1K上，我们的方法在ViM-S上比EViT方法高4.0%的准确率
  - 在PlainMamba系列上，我们的方法比EViT高2.4%-2.8%的准确率
  - 对于剪枝后的PlainMamba-L3，在ImageNet上达到81.7%的准确率，同时FLOPs减少41.6%
  - 在目标检测和实例分割任务上，我们的方法也显示出优越的性能

## 6. 主要贡献
- **理论贡献**：
  - 首次分析了直接应用ViT token剪枝技术到SSMs-based视觉模型失败的根本原因
  - 揭示了SSMs中扫描机制对token邻域关系的敏感性
  - 为理解SSMs在视觉任务中的行为提供了更深入的见解

- **方法贡献**：
  - 提出了第一个专门针对SSMs-based视觉模型的通用token剪枝方法
  - 设计了剪枝感知的隐藏状态对齐方法，解决了传统token剪枝导致的邻域关系破坏问题
  - 提出了适应SSMs模型的token重要性评估方法
  - 提供了高效实现和实际加速方法

- **实践贡献**：
  - 首次实现了基于token剪枝加速视觉SSMs模型
  - 在图像分类、目标检测和实例分割任务上进行了全面实验，证明了方法的有效性
  - 在保持高准确率的同时实现了显著的计算量减少