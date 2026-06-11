# 论文深度总结：What Kind of Visual Tokens Do We Need Training-Free Visual Token Pruning for Multi-Modal Large Language Models from the Perspective of Graph

## 1. 研究动机
- **现有背景**：当前多模态大语言模型(Multimodal Large Language Models, MLLMs)通常使用大量视觉token来弥补视觉能力不足，这导致了:
  - 过高的计算开销(如LLaVA-NeXT使用2880个视觉token，每次推理需要约18.52 TFLOPs)
  - 明显的视觉冗余问题
- **研究痛点**：
  - 现有MLLMs在视觉任务上存在严重冗余，随机丢弃部分视觉token对常见场景性能影响不大
  - 但在细粒度任务(如TextVQA)上，随机剪枝会导致性能急剧下降
  - 现有研究往往只关注前景信息(主要物体)，忽视了背景信息的重要性

## 2. 解决问题
论文致力于解决的核心科学问题：
- **视觉token冗余性**：如何识别和去除MLLMs中的视觉冗余token
- **token选择策略**：如何在前景和背景token间做出合理选择，因为两者对MLLMs都很重要
- **计算效率**：如何在不显著降低性能的情况下大幅减少计算开销
- **通用性**：如何开发一种适用于不同难度例子的token剪枝方法

## 3. 现象分析
作者观察到的重要现象和深入剖析：
- **冗余性验证**：通过实验证明MLLMs中的视觉token存在明显冗余，随机丢弃一定数量token对常见场景影响小
- **前景与背景同等重要**：通过计算l2-Norm频率分布直方图，发现前景和背景区域的分布有显著重叠，两者都对MLLMs任务至关重要
- **任务依赖性**：不同难度的例子需要保留不同类型的重要token，不能简单保留前景或背景
- **现有方法局限**：传统方法如ToMe、FastV等在细粒度任务上性能下降明显，特别是在高剪枝比例下

## 4. 主要方法
论文提出了一种基于图的无训练视觉token剪枝方法，称为**G-Prune**：

### 核心机制
1. **图构建**：
   - 将视觉token视为图节点
   - 根据token间的余弦相似性构建连接：$A_{ij} = \max(0, \cos(X_i, X_j) - s)$
   - 其中$s$是相似性阈值

2. **信息传播**：
   - 根据l2-Norm初始化token的信息量：$S_i^{(0)} = ||X_i||_2$
   - 通过迭代算法传播信息：$S^{(t)} = S^{(t-1)} \cdot (A')$
   - 其中$A'$是经过softmax归一化的邻接矩阵

3. **重要性评分**：
   - 计算每个token的度：$D_i = \sum_{j=1}^{N} \mathbb{1}(A_{ij} > 0)$
   - 归一化token分数：$S_i'^{(t)} = \frac{S_i^{(t)}}{D_i}$

4. **代表性token选择**：
   - 保留评分最高的$k$个token，这些可以是前景或背景token

## 5. 数据与实验
### 数据集
- **通用VQA任务**：VQA2.0、GQA
- **细粒度文本导向VQA**：TextVQA、DocVQA、ChartQA
- **MLLM专用基准**：POPE、MME、MMB

### 基线模型
- 随机剪枝(Random)
- 前景保留剪枝(Foreground-preserving pruning)
- 背景保留剪枝(Background-preserving pruning)
- ToMe (Token Merging)
- FastV

### 核心实验结果
- **计算效率提升**：
  - 在VQA2.0和TextVQA上，减少63.57% FLOPs
  - 在MME上，减少63.50%计算量而性能无下降

- **性能保持**：
  - VQA2.0上仅损失0.95%准确率
  - TextVQA上仅损失2.34%准确率
  - 在TextVQA上剪枝90% token时，准确率达59.31%，显著优于ToMe的39.02

- **不同任务表现**：
  - 在通用VQA任务上优于其他方法
  - 在细粒度任务上优势更明显
  - 在文本导向任务(如TextVQA、DocVQA)上表现最佳

## 6. 主要贡献
论文对领域的三大贡献：

1. **理论贡献**：
   - 首次系统研究不同类型视觉token对MLLMs的重要性
   - 揭示前景和背景token对MLLMs都至关重要，挑战了只关注前景的传统观点

2. **方法贡献**：
   - 提出G-Prune，一种基于图的训练-free视觉token剪枝方法
   - 通过迭代信息传播机制，能够智能选择最具代表性的视觉token
   - 方法无需额外训练，可作为即插即用模块集成到现有MLLMs中

3. **应用贡献**：
   - 在大幅减少视觉token的同时保持高性能
   - 在细粒度任务(如TextVQA、DocVQA)上表现尤为突出
   - 为MLLMs的高效部署提供了新思路，特别是在资源受限环境中的应用