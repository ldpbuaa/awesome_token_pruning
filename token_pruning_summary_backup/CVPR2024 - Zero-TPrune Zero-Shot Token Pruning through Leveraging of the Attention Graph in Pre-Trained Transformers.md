# 论文总结：Zero-TPrune: Zero-Shot Token Pruning through Leveraging of the Attention Graph in Pre-Trained Transformers

## 1. 研究动机
- Transformer模型在边缘设备上的部署面临挑战，推理成本随输入序列中token数量呈二次方增长。
- 现有Token Pruning方法虽易于部署在各种Transformer骨干网络上，但大多需要计算昂贵的微调(fine-tuning)。
- 例如，最先进的DynamicViT方法需要150小时A100 GPU微调时间来修剪DeiT-S模型，对更大模型(DeiT-B和DeiT-L)则需要数千小时。
- 边缘设备和用户计算资源有限，反复微调不同修剪配置的模型不切实际。
- 需要一种无需微调的零样本token修剪方法，解决现有方法的高计算开销问题。

## 2. 解决问题
- 如何在不进行微调的情况下有效执行token修剪，减少Transformer模型的推理成本。
- 如何利用预训练Transformer模型的注意力图(attention graph)识别重要与可修剪的token。
- 如何同时考虑token的重要性和相似性，实现更有效的修剪。
- 设计一种计算效率高的算法，适用于各种规模的Transformer模型，并支持轻松切换不同修剪配置。

## 3. 现象分析
- 注意力矩阵可视为有向图的邻接矩阵，token为节点，注意力值为边权重。
- 这种注意力图是推断重要token和可修剪token的丰富信息源。
- 实验表明token经常学习相似抽象，可修剪相同特征副本而不会丢失信息。
- 背景token有时可能比主要对象token获得更高的重要性分数，导致不正确的token排序。
- 不同注意力头通常关注输入图像的不同部分，某些token在一个或两个头中非常重要，而在其他头中不是。
- 某些注意力头中，重要性分数可能收敛到不需要的分布：图像边缘token可能获得过高分数，或分布变得几乎均匀。

## 4. 主要方法
### Zero-TPrune框架
- 零样本token修剪方法，同时考虑token的重要性和相似性。
- 利用预训练Transformer模型的注意力图生成token重要性分布。
- 可插入Transformer块之间，由I-stage和S-stage组成。

### I-stage（基于重要性的修剪）
- **Weighted Page Rank (WPR)算法**：将注意力矩阵视为有向图的邻接矩阵，通过迭代分配相对重要性识别重要token。
- **Emphasizing Informative Region (EIR)聚合**：通过平方和的均方根聚合不同头的重要性分数，有效区分信息区域。
- **Variance-based Head Filter (VHF)**：基于方差过滤掉提供误导性信息的注意力头，设置最小和最大方差阈值。

### S-stage（基于相似性的修剪）
- 基于重要性分数的token分区，将token分成大致相等大小的组。
- 在不同组间识别最相似的token对，并修剪其中一个token。
- 使用Key矩阵中的向量表示每个token，采用余弦相似度计算相似性。

### 整体流程
- 采用I'_stage→S-stage→I-stage顺序，减少相似性对I-stage的负面影响。
- I'_stage仅分配重要性分数而不修剪token，为S-stage提供指导。

## 5. 数据与实验
### 数据集
- ImageNet：评估各种视觉Transformer骨干网络性能。
- 下游任务数据集：检查模型迁移学习能力（具体数据集在补充材料中）。

### 基线模型
1. 需要微调的方法：
   - DynamicViT：在Transformer块间插入预测模块预测和丢弃信息量较少的token。
   - A-ViT：引入随机修剪过程，使用自适应停止模块计算每个token的停止概率。

2. 无需微调的方法：
   - ATS：使用逆变换实现基于重要性分数分布的自适应token采样。
   - ToMe：专注于合并token而非修剪，基于相似性 progressively 合并token。

### 核心实验结果
- ImageNet上，Zero-TPrune将DeiT-S的FLOPs成本降低34.7%，吞吐量提高45.3%，仅0.4%准确率损失，无需任何微调。
- 与需要微调的最先进方法相比，Zero-TPrune消除修剪后微调需求，仅造成0.1%准确率损失。
- 与最先进的无需微调方法相比，在相似FLOPs预算下将准确率损失降低最多49%。
- 在DeiT-S上，与最先进的无需微调方法相比，准确率损失降低33%。
- 在DeiT、LV-ViT、AugReg等多种Transformer骨干网络上验证了方法有效性。

## 6. 主要贡献
1. 提出Zero-TPrune，第一个同时考虑token重要性和相似性的零样本token修剪方法。
2. 利用预训练Transformer注意力图，通过加权PageRank (WPR)算法生成token重要性分布。
3. 提出强调信息区域(EIR)聚合和基于方差的头过滤(VHF)技术，提高重要性估计准确性。
4. 设计基于重要性的token分区策略，用于高效相似性修剪，维护与专业骨干网络的兼容性。
5. 实验证明Zero-TPrune在各种视觉Transformer骨干网络上性能优于现有方法，显著减少准确率损失并提高吞吐量。
6. 消除修剪后微调开销，使大模型修剪、不同修剪配置间切换及高效超参数调整成为可能。