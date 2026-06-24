# DivPrune: Diversity-based Visual Token Pruning for Large Multimodal Models 论文总结

## 1. 研究动机
- 大型多模态模型(Large Multimodal Models, LMMs)能够处理文本、图像和视频等多种数据模态，但视觉tokens的加入显著增加了总token数量(通常增加数千个)。
- LLM的输入长度增加导致推理复杂度大幅提高，造成高延迟，难以满足低延迟应用需求，特别是在资源受限环境中。
- 现有token剪枝方法存在两个主要痛点：(1)需要大量校准(calibration)和微调(fine-tuning)，成本高且耗时；(2)依赖次优的重要性指标(importance metrics)，导致保留的token间冗余度高。

## 2. 解决问题
- 解决LMMs中视觉tokens冗余度高导致的推理延迟和内存消耗问题。
- 解决现有token剪枝方法在高压缩比(≥80%)下性能显著下降的问题。
- 消除对校准数据和模型微调的依赖，提供即插即用的解决方案。

## 3. 现象分析
- 视觉信息处理中存在高度冗余，研究表明可减少50%到95%的视觉tokens而不显著影响模型性能。
- 基于注意力分数(attention scores)的剪枝方法在高压缩比下表现不佳，倾向于保留彼此相似的tokens，增加冗余。
- 当压缩比例很高时，这种冗余阻碍了选择足够数量的独特token来准确表示原始token集，导致性能大幅下降。

## 4. 主要方法
- 提出DivPrune，将token剪枝问题表述为最大最小多样性问题(Max-Min Diversity Problem, MMDP)。
- MMDP目标：选择一个子集，使得所选元素之间的多样性最大化，即最小化元素间的最小距离。
- 核心机制：
  - 使用余弦距离(cosine distance)测量token间相似性
  - 两阶段算法实现：
    1. 第一阶段：基于候选列表中token之间的成对距离选择第一个token
    2. 第二阶段：迭代添加后续token，确保新添加token与已选token集中所有token之间的最小距离最大化
  - 通过一次矩阵乘法计算距离矩阵，避免重复计算，选择过程开销相对于LLM内计算可忽略

## 5. 数据与实验
- **数据集**：16个数据集，包括11个图像语言数据集(COCO, Flickr, GQA, MMBench, MME, MMMU, Nocaps, OKVQA, POPE, ScienceQA, SeedBench)和5个视频语言数据集。
- **基线模型**：与5种基线方法比较，包括FastV、PruMerge、VTW、FitPrune和M[3]。
- **核心实验结果**：
  - 在各种LMMs(LLaVA 1.5-7B, LLaVA 1.5-13B, LLaVA 1.6-7B, LLaVA-NeXT-Video-7B)上实现了最先进的准确性
  - 在极端剪枝比例(≥80%)下表现出显著优势，例如在LLaVA 1.5-7B上，当TFLOP比例降至约15%时，DivPrune在COCO数据集上的CIDEr分数仅下降12.7%，而VTW和FastV分别下降约95%
  - 减少了GPU内存使用和推理延迟，同时保持了与原始模型相当的准确性
  - 在无需微调或校准的情况下，在大多数数据集上实现了与需要校准或微调的基线相当或更好的性能

## 6. 主要贡献
- 提出DivPrune，一种基于MMDP的token剪枝方法，最大化视觉token间的多样性，有效减少冗余并确保高度代表性的子集。
- 提出一种无需训练、无需校准数据、即插即用的解决方案，可无缝集成到现成的LMMs中。
- 在16个数据集上对图像和视频语言模型进行了全面评估，DivPrune实现了最先进的性能，在极端剪枝(比例≥80%)下有显著提升。
- 实证表明DivPrune减少了GPU内存使用和推理延迟，同时在大多数数据集上保持了与原始模型相当的准确性。