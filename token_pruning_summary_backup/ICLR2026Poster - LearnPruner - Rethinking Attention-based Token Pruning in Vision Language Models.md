# 论文总结：LearnPruner: Rethinking Attention Based Token Pruning in Vision Language Models

## 1. 研究动机
- Vision-Language Models (VLMs) 在视觉理解和推理方面展现出卓越能力，但对长视觉序列输入造成显著计算负担
- 高分辨率图像或长视频输入场景下，大量视觉tokens给VLM推理带来严峻挑战
- 计算负担严重限制了VLM在资源受限环境和实时应用中的实际部署
- 现有token剪枝方法主要依赖视觉编码器或LLMs的注意力分数来确定token重要性，但这些方法存在局限性

## 2. 解决问题
- 如何有效减少VLM中的视觉token数量，同时保持模型性能
- 如何更准确地评估视觉token的重要性，提高剪枝效率
- 如何解决现有注意力机制在视觉编码器和LLMs中用于token剪枝时的局限性

## 3. 现象分析
- 视觉编码器中的[CLS] token无法充分关注显著前景区域，导致在有限token预算下剪枝结果次优
- 视觉编码器存在"注意力汇点"(attention sink)问题，对信息丰富的前景区域关注不足
- LLMs中的注意力存在"注意力偏移"(attention shift)现象，视觉token索引越高，接收的注意力分数也越高
- 文本到视觉的注意力(text-to-vision attention)表现出对注意力偏移的抵抗力，在中间层能提供可靠的token选择指导
- LLM中间层的文本到视觉注意力能更好地区分与查询相关的视觉区域

## 4. 主要方法
提出了**LearnPruner**，一个两阶段token剪枝框架：

### 第一阶段：移除视觉冗余
- 使用轻量级可学习剪枝模块(LPM)直接预测视觉token重要性分数
- LPM是一个多层感知机(MLP)，执行二元分类确定每个token的保留或剪枝
- 采用直通估计器(STE)实现二进制决策的可微分训练
- 引入基于多样性的token选择，保留少量多样性token提供互补视觉信息

### 第二阶段：移除文本不相关内容
- 在LLM中间层执行查询感知的token选择
- 利用文本到视觉注意力分数指导剪枝：$A_{text} = \frac{1}{N_q}\sum_{k=1}^{N_q} \text{Attention}(X_q^{(k)}, X_v^{(k)})$
- 仅保留注意力分数最高的前k个token参与LLM内进一步交互

## 5. 数据与实验
### 数据集
- 训练：10%的LLaVA-665K数据集
- 评估：GQA、POPE、MME、TextVQA、VQAv2、MMB、MMBCN、TGIF-QA、MSVD-QA、MSRVTT-QA等多个基准

### 基线模型
- FastV、SparseVLM、DivPrune、DART、VisPruner、VisionZip、TwigVLM

### 核心实验结果
- 保留约5.5%视觉tokens时，LearnPruner保留约95%原始性能
- 实现3.2倍推理加速
- 在保留5.6% tokens时，保留94.8%原始性能，实现2.3倍(prefill)和1.5倍(总时间)加速
- 高分辨率场景下(LLaVA-NeXT)，保留5.6% tokens时性能达原始模型的99.3%
- 视频理解任务中同样表现优越

## 6. 主要贡献
1. **新的分析视角**：深入分析视觉编码器和LLMs中注意力机制在token剪枝中的有效性，发现[CLS] token局限性和文本到视觉注意力的优越性
2. **新的框架**：提出LearnPruner两阶段剪枝框架，结合可学习剪枝模块和基于文本注意力的查询感知选择
3. **新的性能记录**：在多个基准测试中实现最先进的性能-效率权衡，极低token保留率下保持高准确性
4. **新的训练方法**：引入轻量级可学习剪枝模块，使用STE训练，保持原始VLM权重不变