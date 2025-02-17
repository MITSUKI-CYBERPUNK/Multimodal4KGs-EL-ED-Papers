>COLING 2024
# 1 Motivation
论文的主要动机是解决多模态实体链接（MEL）任务中的两个关键问题：
1. **随机负样本采样的局限性**：
    - 现有的MEL方法通常使用对比学习（contrastive learning），通过随机采样负样本进行训练。然而，这种方法可能会引入“容易”的特征（easy features），导致模型忽略实体的独特细节。例如，在知识库中存在多个高度相似的实体（如不同国家的“最高法院”）时，随机采样这些相似实体作为负样本会阻碍模型正确链接输入句子和目标实体。
2. **视觉模态的多样性问题**：
    - 现有的MEL方法忽略了提及（mention）和目标实体之间视觉模态的差异。例如，输入图像可能包含多个实体，而目标实体的图像可能与输入图像中的某个实体不完全匹配。这种视觉模态的多样性会干扰实体链接任务。

为了解决这些问题，作者提出了两个主要方法：
- **基于Jaccard距离的条件对比学习（JD-CCL）**：通过利用元信息（meta-information）选择具有相似属性的负样本，增加训练的难度和鲁棒性。
- **上下文视觉辅助可控变换（CVaCPT）**：通过合成图像和上下文文本信息增强视觉表示，解决视觉模态的多样性问题。

# 2 Contribution
1. **提出JD-CCL方法**：
    - 利用**Jaccard距离**选择具有相似属性的负样本，使模型在训练时不能依赖简单的属性（如“人类”、“建筑”或“动物”），而是需要识别更复杂的特征来区分正负样本。
    - 通过预处理阶段计算知识库中实体之间的**Jaccard相似度**，选择最难的负样本进行训练。
2. **提出CVaCPT模块**：
    - 使用**文本到图像扩散模型（text-to-image diffusion）** 生成合成图像，帮助模型根据具体提及（mention）控制输入图像的关键特征。
    - 通过**多视图合成图像**和**上下文文本信息**增强视觉表示，减少噪声和不相关特征的影响。
3. **实验验证**：
    - 在三个基准MEL数据集（**WikiDiverse、RichpediaMEL和WikiMEL**）上验证了方法的有效性，证明了其优于现有最先进方法的性能。

# 3 Methodology
![image.png](https://aquazone.oss-cn-guangzhou.aliyuncs.com/20250217110251.png)

## **3.1 总体架构**

- **视觉编码器（Visual Encoder）**：
    - 使用预训练的Vision Transformer（ViT）**作为视觉编码器**，将图像分割成2D patches，并通过多层Transformer处理，生成全局特征（global feature）和局部特征（local feature）。
    - 公式：
        $VEi​G​=VISenc(Ei​),VEi​L​=VISenc(Ei​)$
- **文本编码器（Textual Encoder）**：
    - 使用**BERT**作为文本编码器，将提及（mention）和上下文句子拼接后输入模型，生成全局和局部文本特征。
    - 公式：
        $TMj​G​=TEXTenc(Mj​),TMj​L​=TEXTenc(Mj​)$
- **匹配模块（Matcher）**：
    - 计算提及和实体之间的相似性分数，使用对比学习损失函数进行训练。
    - 公式：
        $L=−logexp(F(M,Epos​))+∑i​exp(F(M,Eneg,i​))exp(F(M,Epos​))​$

## **3.2 基于Jaccard距离的条件对比学习（JD-CCL）**
- **条件采样（Conditional Sampling）**：
    - 计算知识库中实体之间的Jaccard相似度，选择最相似的实体作为负样本。
    - 公式：
        $J(i,j)=∣eai​​∪eaj​​∣∣eai​​∩eaj​​∣​$
    - 选择前k个最相似的实体作为负样本：
        $J^i​=topk(Ji​)$
- **JD-CCL损失函数**：
    - 使用条件对比学习损失函数，使模型在训练时面对更复杂的负样本。
    - 公式：
$LJD-CCL​=E(M,Epos​)∼PM,E​,{Eneg,j​}∼J^i​​[logexp(F(M,Epos​))+∑j=1k​exp(F(M,Eneg,j​))exp(F(M,Epos​))​]$

## **3.3 上下文视觉辅助可控变换（CVaCPT）**
- **可控特征变换（Controllable Feature Transformation, CFT）**：
    - 使用合成图像和上下文文本信息调整输入图像的特征表示。
    - 公式：
$$
V^Ej​G​=VEj​G​+w(αG⊙VEj​G​+βG)V^Ej​L​=VEj​L​+w(αL⊙VEj​L​+βL)\alpha^G, \beta^G = \text{AG}(\langle V^G_{E_j}, V^T_{E_s_j} \rangle)\alpha^L, \beta^L = \text{AL}(\langle V^L_{E_j}, V^T_{E_s_j} \rangle)V^T_{E_s_j} = \text{CE}(\langle V^G_{E_s_j}, T^G_{E_j} \rangle)
$$
- **多视图合成图像（Multi-view Synthetic Images）**：
    - 使用多个合成图像进行特征融合，通过最大池化（max-pooling）选择最重要的特征。
    - 公式：
        $$\hat{V}^T_{E_s_j} = [\hat{V}^T_{E_s_j,1}, \ldots, \hat{V}^T_{E_s_j,n_s}]V^T_{E_s_j,h} = \text{CE}(\langle V^G_{E_s_j,h}, T^G_{E_j} \rangle)V^T_{E_s_j} = \text{max\_pooling}(\hat{V}^T_{E_s_j})$$

# 4 Quota
![image.png](https://aquazone.oss-cn-guangzhou.aliyuncs.com/20250217110316.png)


# 5 Q&A
+ 负样本，零样本，少样本
+ 距离淆
+ 相似度淆

# 6 Revelation
+ **条件对比学习的重要性**
	- 通过选择具有相似属性的负样本，JD-CCL方法显著提高了模型对实体独特特征的识别能力，这对于处理复杂的多模态数据非常有效。
	- 这种方法可以应用于其他对比学习任务，特别是在需要区分高度相似样本的场景中。
+ **可控特征变换的潜力**
	- CVaCPT模块通过合成图像和上下文信息调整视觉特征，展示了可控特征变换在多模态任务中的潜力。
	- 这种方法可以扩展到其他多模态任务，如图像描述生成、视觉问答等，通过合成图像增强模型的视觉理解能力。
