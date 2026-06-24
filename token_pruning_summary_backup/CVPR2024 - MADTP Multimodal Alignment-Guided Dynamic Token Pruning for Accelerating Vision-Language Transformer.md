# 论文总结：MADTP: Multimodal Alignment-Guided Dynamic Token Pruning for Accelerating Vision-Language Transformer

## 1. 研究动机
Vision-Language Transformers (VLTs) 在多模态学习领域取得了显著成功，但同时也带来了巨大的计算成本，主要原因是视觉和语言token数量庞大。现有VLT剪枝研究主要遵循单模态方案，忽略了不同模态对齐在引导token剪枝过程中的关键作用，导致在一个模态分支中重要的token在另一个模态分支中被错误剪除。此外，现有的VLT剪枝工作也缺乏根据不同输入样本动态压缩每层token的灵活性。

## 2. 解决问题
论文致力于解决两个核心问题：
1. **多模态对齐问题**：直接应用单模态剪枝方法而不考虑每个token的跨模态语义相关性，会错误删除在一个模态中不太重要但在另一个模态中至关重要的token，进一步加剧不同模态分支间的表征能力不平衡。
2. **动态压缩问题**：不同输入样本需要不同复杂度的计算，但现有动态剪枝工作主要关注单模态压缩，缺乏针对不同输入动态确定token跨模态重要性的方法。

## 3. 现象分析
作者观察到：
1. VLT模型通常包含多个特定于模态的子模块，导致不同模态间的参数和特征分布不平衡。
2. 不同模态分支在VLT中对同一语义概念产生具有不同表征能力的token。
3. 现有单模态剪枝方法直接应用于VLT时，会加剧压缩后不同模态分支间的表征能力不平衡，影响模型性能。

## 4. 主要方法
论文提出了MADTP（Multimodal Alignment-Guided Dynamic Token Pruning）框架，包含两个核心模块：

### 4.1 多模态对齐引导（MAG）模块
- 使用可学习token作为共同特征空间，建立视觉和语言模态间的关联
- 通过缩放点积注意力层计算可学习token与映射后的视觉token间的相关性
- 计算视觉特征和语言特征间的相似性，纳入损失约束
- 位于VLT架构中两个模态分支的transformer块之间

### 4.2 动态token剪枝（DTP）模块
- 位于每个Transformer块的自注意力层和前馈网络之间
- 计算Token Importance Score (TIS)：通过平均三种分数获得
  - 类注意力分数（S_cls）
  - 自注意力分数（S_self）
  - token注意力分数（S_token）
- 使用可学习阈值实现实例级自适应token剪枝
- 基于TIS和可学习阈值进行token剪枝，保留分数高于阈值的token

## 5. 数据与实验
### 5.1 数据集
- **NLVR2**：包含107,292对图像和文本描述
- **COCO**：约330,000张图像，每张图像有五个文本描述
- **Flickr30k**：31,783张图像，主要用于图像和文本检索
- **VQA v2.0**：关于图像的人类标注、开放式问答数据集

### 5.2 基线模型
- Upop [35]
- ELIP [17]
- CrossGET [36]
- 静态token剪枝（STP）作为对照

### 5.3 核心实验结果
- 在NLVR2数据集上，应用于BLIP模型时，MADTP减少80% GFLOPs，性能下降不到4%
- 在0.3的reduce ratio下，MADTP在dev集上比Upop提高2.17%准确率，test集上提高2.07%
- 在0.5的reduce ratio下，改进分别扩展到5.08%和5.24%
- 在Flickr30K数据集上，MADTP在0.5和0.75的reduce ratio下均优于Upop

## 6. 主要贡献
1. 揭示了多模态对齐在引导VLT压缩中的关键作用，提出了MADTP框架，有效加速各种Vision-Language Transformers
2. 提出MAG模块，明确对不同模态的联合表示进行对齐，在多模态token剪枝过程中提供指导
3. 提出DTP模块，根据输入实例的复杂度动态调整VLT模型每层的压缩比例
4. 通过多数据集和模型的广泛实验验证了MADTP的SOTA性能，特别是在BLIP模型上实现80% GFLOPs减少同时性能下降不到4%