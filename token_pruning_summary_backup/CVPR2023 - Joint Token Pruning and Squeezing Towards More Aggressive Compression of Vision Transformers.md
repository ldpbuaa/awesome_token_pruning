# 论文总结：Joint Token Pruning and Squeezing Towards More Aggressive Compression of Vision Transformers

## 1. 研究动机
- Vision Transformers (ViTs) 在各种计算机视觉任务中表现出色，但其高计算成本限制了实际应用
- 之前的修剪冗余token方法在性能与计算成本间取得了良好平衡，但修剪策略导致的错误会造成显著信息损失
- 定量实验表明，被修剪的token对模型性能的影响不容忽视
- 当保留token数量接近10以下时，性能会急剧下降，因为主体和辅助上下文信息都丢失严重
- 背景token丢失会导致错误分类（如缺少包含草地的背景token使割草机被误判为折叠椅）

## 2. 解决问题
- 致力于解决Vision Transformer在更激进压缩情况下的信息丢失问题
- 开发更高效的压缩方法，在减少计算成本的同时保持或提高模型性能
- 特别关注如何在保留token数量很少的情况下，尽可能保留被修剪token中的有用信息

## 3. 现象分析
- 发现被修剪的token仍包含有用信息，通过"反向修剪策略"实验证明（图3）
- 当交换保留和修剪的token（反向策略）时，被修剪的token仍能正确处理部分案例
- 随着token修剪强度增加，只有反向策略能正确预测的案例比例（奖励准确率）也随之上升
- 表明被修剪token中的独有信息在修剪强度增加时变得更重要
- 现有token重组织方法（如EViT和Evo-ViT）将修剪token聚合为一个，忽略了token间差异，导致特征坍塌

## 4. 主要方法
- 提出联合Token修剪与挤压（Token Pruning & Squeezing, TPS）模块
- TPS包含两个主要步骤：
  1. **Token修剪**：将输入token分为保留集合 $S^{[r]}$ 和修剪集合 $S^{[p]}$
  2. **Token挤压**：将修剪token信息挤压到保留token中，而非简单丢弃
- Token挤压过程包含两个子步骤：
  - **匹配**：使用单向最近邻匹配算法，将每个修剪token分配给相关保留token（宿主token）
  - **融合**：使用基于相似性的融合方法，将匹配修剪token特征融合到相应宿主token
- 提出两种灵活变体：
  - **dTPS**（块间变体）：采用dynamicViT的可学习token评分预测头
  - **eTPS**（块内变体）：使用类token注意力值衡量token重要性
- 设计减少了上下文信息损失，保持合理计算预算，实现硬件友好的恒定形状推理

## 5. 数据与实验
- **数据集**：
  - ImageNet1K：主要对比实验
  - iNaturalist 2019：细粒度视觉分类任务

- **对比基线模型**：
  - DynamicViT [25]：token修剪方法
  - EViT [16]：token重组织方法
  - 其他SOTA transformer模型，包括token修剪方法、vanilla ViTs和混合ViTs

- **核心实验结果**：
  - 在所有token修剪强度下，TPS均优于基线方法
  - 将DeiT-tiny和small计算预算压缩到35%时，ImageNet分类上比基线提高1%-6%准确率
  - TPS使DeiT-small吞吐量达1745图像/秒，超过DeiT-tiny的1686图像/秒，准确率高出4.78%
  - 各种transformers上的实验证明方法有效性
  - 分析实验证明TPS对token修剪策略错误具有更高鲁棒性
  - iNaturalist 2019上，dTPS比DynamicViT提高0.3%(DeiT-tiny)和0.2%(DeiT-small)准确率

## 6. 主要贡献
- 提出联合Token修剪与挤压（TPS）模块及其变体（dTPS和eTPS），保留被丢弃token信息，促进ViTs更激进压缩
- 实验证明与先前方法相比的更高性能，特别是在DeiT-small和tiny压缩到35% GFLOPs时，TPS比基线提高1%-6%准确率
- 广泛实验将方法应用于vanilla ViTs和混合ViTs，证明灵活性，分析实验证明比token修剪和token重组织更鲁棒
- 提出新颖token减少方法，具有更高效率、鲁棒性和灵活性，仅需微调预训练模型