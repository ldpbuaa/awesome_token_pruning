# SPViT: Enabling Faster Vision Transformers via Latency-aware Soft Token Pruning - 论文总结

## 1. 研究动机

- **ViT部署瓶颈**：Vision Transformer在计算机视觉领域持续取得突破，但其高计算和内存成本使其在工业生产中部署困难
- **现有方法局限性**：
  - 注意力头剪枝(attention head pruning)仅减少部分ViT计算(MSA)，效率低下
  - 静态token剪枝(static token pruning)对所有图像使用固定比例，忽略不同图像高级信息的区域和位置差异
  - 动态token剪枝(dynamic token pruning)搜索空间大，且直接丢弃不重要的token导致信息损失
- **边缘设备需求**：需要一种能同时优化ViT准确率并最大化每图像动态剪枝率，同时保持边缘设备实际计算约束的方法

## 2. 解决问题

论文专注于解决以下核心科学问题和技术难题：
- 如何有效减少Vision Transformer的计算复杂度，使其能在边缘设备高效部署
- 如何实现每图像自适应的token剪枝，而非对所有图像使用固定剪枝率
- 如何在剪枝过程中保留被丢弃token中的有用信息，避免信息损失
- 如何根据目标边缘设备的延迟规格约束模型剪枝，确保模型满足硬件性能要求

## 3. 现象分析

论文观察到的重要现象和深入剖析包括：
- **多头特性**：ViT中每个头独立编码视觉感受野，每个token在不同头中具有不同影响
- **层间差异**：早期和中间层的token表示编码不足，使得在这些层进行token剪枝较为困难
- **背景信息重要性**：完全移除背景(负面)token会削弱自注意力机制捕获关键信息的能力
- **特征相似度分析**：使用CKA分析表明，早期层的token特征与最终CLS token特征差异较大，证明早期层token表示编码不充分
- **计算效率分析**：在相同剪枝率下，剪枝token(减少N)比剪枝通道(减少Dch或Dattn)能减少更多整体计算量

## 4. 主要方法

### SPViT框架
一个延迟感知的软剪枝框架，包含两个核心组件：

1. **基于注意力的多头token选择器**：
   - 评估每个token在所有头中的重要性分数
   - 为每个头生成token分数图，通过MLP提取局部和全局特征
   - 通过注意力分支合并各头分数，获得最终token分数
   - 使用Gumbel-Softmax技术生成可微分的token保留/剪枝决策，避免Top-k排序

2. **Token打包技术**：
   - 将被认为不重要的token压缩成一个打包token(package token)
   - 将打包token与保留的token连接，参与后续计算
   - 允许模型纠正评分错误，并保留背景特征
   - 计算成本小(小于总模型GFLOPs的1%)

### 延迟感知训练策略
1. **延迟感知稀疏损失**：将token剪枝率与目标边缘设备的延迟规格关联
2. **层到阶段渐进训练**：
   - 从后到前渐进插入token选择器
   - 将具有相似剪枝率的相邻块组合为一个阶段
   - 仅保留每个阶段的第一个选择器
   - 如果最终计算低于目标延迟，则降低第一个选择器的剪枝率

## 5. 数据与实验

### 数据集
- ImageNet-1K数据集，图像分辨率为224×224

### 对比的基线模型
- DeiT-T, DeiT-S, LV-ViT-S, LV-ViT-M, PiT-T, PiT-XS, PiT-S, Swin-T, Swin-S
- 与DynamicViT, IA-RED[2], RegNetY, CrossViT, VTP, ATS, CvT, PVT, T2T-ViT, UP-DeiT, PS-ViT, EvoViT, TNT, HVT, Swin, CoaT, CPVT, EViT, UVC, MD-DeiT, S[2] ViTE等方法进行比较

### 核心实验结果
1. **延迟优化**：
   - 在Samsung Galaxy S20手机上，SPViT将DeiT-T的延迟降低到26ms，比现有方法快26%~41%
   - 在Xilinx ZCU102 FPGA上，SPViT使DeiT-S的延迟为13.2ms
   - 实现了DeiT-T在移动设备上的实时推理(26ms < 33ms)

2. **准确率与计算量权衡**：
   - DeiT-T和DeiT-S在减少40%-60%推理延迟的情况下，仅损失0.5%的准确率
   - SPViT在ImageNet上实现了更好的计算量-准确率权衡
   - 例如，SPViT在减少延迟的同时，在ImageNet上实现了0.25%~4%更高的top-1准确率

3. **模型适用性**：
   - SPViT适用于扁平结构(如DeiT)和层次结构(如Swin)的Transformer
   - 能够生成更高效的PiT和Swin模型，性能损失可忽略

## 6. 主要贡献

这篇论文对该领域的主要贡献包括：

1. **理论贡献**：
   - 提供了对ViT计算复杂度的详细分析，证明token剪枝相比其他维度的压缩能带来更大的计算量减少
   - 分析了不同ViT压缩策略的优缺点

2. **方法贡献**：
   - 提出了SPViT，一个新颖的延迟感知软剪枝框架
   - 设计了基于注意力的多头token选择器，实现每图像自适应剪枝
   - 引入了token打包技术，保留被剪枝token中的有用信息
   - 提出了延迟感知训练策略，有效探索SPViT设计空间，最大化每图像的剪枝率而不降低准确率

3. **性能贡献**：
   - SPViT实现了比现有方法更高的剪枝率，同时保持可比的准确率
   - 对于轻量级模型，SPViT显著减少了推理延迟(40%-60%)，仅损失0.5%准确率
   - 首次实现了ViT模型在边缘设备上超越实时性能的推理

4. **应用贡献**：
   - 代码已公开：https://github.com/PeiyanFlying/SPViT
   - SPViT在移动设备和FPGA上展示了出色的性能，为ViT在边缘设备上的部署提供了实用解决方案