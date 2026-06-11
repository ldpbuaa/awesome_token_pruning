# 论文总结：PACT: Pruning and Clustering-Based Token Reduction for Faster Visual Language Models

## 1. 研究动机
- 视觉语言模型(VLMs)需要大量计算资源进行推理，主要是因为需要额外的输入token来表示视觉信息
- 现有VLM架构(如LLaVA、Qwen-VL等)虽然表现出色，但由于视觉token数量庞大，导致计算成本高昂
- 视觉token中常包含冗余和不重要信息，造成不必要的计算资源浪费
- 现有方法存在局限性：要么与FlashAttention不兼容，要么存在位置偏差，要么仅处理冗余或不相关token中的一种问题

## 2. 解决问题
- 如何减少视觉语言模型推理时间和内存使用量
- 如何在不显著降低模型性能的情况下减少视觉token数量
- 如何同时处理两种类型的视觉token问题：不相关的(irrelevant)和冗余的(redundant)
- 如何开发一种与FlashAttention兼容的token减少方法，避免注意力分数计算带来的开销

## 3. 现象分析
- 现有视觉token减少方法存在多种局限：
  - 基于注意力分数的方法(如FastV)与FlashAttention不兼容
  - 使用平均注意力分数作为修剪指标会导致位置偏差，倾向于修剪序列开头或结尾的token
  - 大多数方法仅处理不相关token或冗余token中的一种，而非同时处理
  - 一些方法(如LLaVA-PruMerge、HiRED)仅适用于特定架构(使用[CLS]token的模型)
  - 一些方法(如VTW)在深层layer才减少token，限制了计算成本减少的效果
- 视觉token的隐藏状态范数存在高方差，表明某些token携带更多信息，可能更适合保留

## 4. 主要方法
- **PACT (Pruning and Clustering-Based Token Reduction)**：结合剪枝和聚类的token减少方法
- **EUTI (Efficient Unimportant Tokens Identification)**：新的重要性度量方法，不依赖注意力分数
  - 计算全局查询向量作为所有查询向量的平均值
  - 结合key向量和全局查询向量的点积，以及隐藏状态范数计算重要性分数
  - 避免位置偏差，确保修剪仅基于token信息内容而非位置
- **DBDPC (Distance Bounded Density Peak Clustering)**：新的聚类算法
  - 保证每个向量到其聚类中心的距离小于预设阈值dc
  - 确保聚类中心间距离有下界，集群内元素间距离有上界
  - 高效且并行化，适合GPU计算
- PACT完整流程：使用EUTI识别重要和不重要token → 使用DBDPC聚类剩余token → 重新引入接近聚类的不重要token → 合并聚类内token

## 5. 数据与实验
### 数据集
- 文档理解：AI2D、TextVQA、ChartQA、DocVQA、InfographicVQA
- 多学科推理：MME、MMBench、MMVet、MathVerse、MathVista、MMMU、MMStar、ScienceQA
- 真实场景评估：Vibe-Eval、MM-LiveBench、LLaVA-BenchWilder
- 图像间推理：LLaVA-Interleave Bench、MuirBench
- 视频理解：ActivityNet-QA、MLVU、VideoMME、EgoSchema、PerceptionTest
- 对话式视频交互：Video-ChatGPT

### 对比基线模型
- FastV：基于平均注意力分数评估token重要性
- VTW：在深层layer移除token
- ToME：迭代合并相似token
- 其他方法：LaVIT、LLaVA-PruMerge、HiRED、TRIM等

### 核心实验结果
- 在LLaVA-OneVision-7B上实现50%视觉token减少，性能损失可忽略不计
- 在71.3%视觉token减少率下，仅损失1.4%性能，而之前最先进方法在同减少率下最多损失4.4%
- 显著提升吞吐量：225%（对比基线100%）
- 降低GPU内存消耗：从27.4GB降至19.05GB
- 在多种模型架构(LLaVA-1.6-Mistral-7B、Qwen2-VL-7B-Instruct、InternVL2-8B)上均表现优异
- 在各种任务和数据集上保持鲁棒性，包括文档理解、多图像推理和视频理解

## 6. 主要贡献
1. 提出EUTI：一种不依赖注意力分数的视觉token剪枝度量，确保与FlashAttention兼容
2. 引入DBDPC：一种新的聚类算法，有效减少视觉冗余，性能优于其他聚类算法
3. 证明剪枝与聚类结合优于单独使用任一技术，实现更有效的视觉token减少
4. 提出PACT方法：集成剪枝和聚类算法，超越之前和同期工作
5. 展示PACT在多种任务和数据集上的有效性，包括单图像、多图像和视频理解场景
6. 方法完全在推理时应用，无需额外训练，便于实际部署