>ACL 2024
# 1 Motivation
- **对文本提及词的过度依赖**：现有的MEL方法通常依赖于文本中的提及词（mention words）作为实体链接的主要线索。然而，在实际应用中，提及词可能缺失或不准确，导致这些方法无法有效利用图像和文本信息进行准确的实体链接。
- **图像模态的噪声问题**：图像中可能包含与实体链接无关的噪声信息，而现有方法往往难以**区分图像中的相关区域和噪声区域**，从而影响链接结果。
- **应用场景的局限性**：传统MEL方法在没有提及词的情况下表现不佳，限制了其在多模态场景中的应用范围，例如在用户不熟悉文本语言或更依赖图像信息的情况下。
为了解决这些问题，作者提出了一个新的任务——**视觉提示引导的多模态实体链接（VP-MEL）**，通过在图像中标注视觉提示（visual prompts）直接指示实体的位置，从而减少对文本提及词的依赖，并提高模型对图像信息的利用效率。

# 2 Contribution
- **提出VP-MEL任务**：首次定义了一个新的多模态实体链接任务，使用**视觉提示代替传统的提及词来标注图像中的实体位置**，从而扩展了MEL的应用场景。![](https://aquazone.oss-cn-guangzhou.aliyuncs.com/20250216214939.png)
- **构建VPWiki数据集**：为了支持VP-MEL任务的研究，作者构建了一个高质量的标注数据集VPWiki，包含大量图像-文本对和对应的**视觉提示标注**。此外，还提出了一种**自动化标注流程**，提高了数据集的构建效率。
- **提出FBMEL框架**：设计了一个名为**FBMEL（Feature Balanced Multimodal Entity Linking）** 的框架，通过**视觉提示增强图像特征提取**，并利用**预训练的Detective-VLM模型生成补充文本信息**，平衡了图像和文本模态的贡献，减少了对单一模态的依赖。
- **实验验证**：在VPWiki数据集上，FBMEL显著优于现有的MEL和VLM基线方法，证明了其在VP-MEL任务中的有效性。此外，FBMEL在传统MEL任务上也表现出竞争力。所以是**自建数据集+传统数据集**

# 3 Methodology
![image.png](https://aquazone.oss-cn-guangzhou.aliyuncs.com/20250216220748.png)
## 3.1 **多模态编码**
- **文本编码**：输入文本 T 通过BERT提取上下文特征：
    
    $Htext=BERT(T)∈RL×d$
    其中 L 为序列长度，d 为特征维度。
- **视觉编码与提示生成**：  
    图像 I 通过ViT提取全局特征 
    
    $Vglobal=ViT(I)∈Rd$，
    
    并生成 KK 个局部视觉提示符：
    
	$P=[p1,p2,…,pK]=Wp⋅Vglobal+bp∈RK×d$
    其中 $Wp∈RK×d、bp​∈RK×d$ 为可学习参数，每个提示符 pi 对应图像中与实体相关的局部区域（如物体、场景）。

## 3.2 **视觉提示引导的跨模态融合**
- **交叉注意力机制**：将视觉提示符作为键（Key）和值（Value），文本特征作为查询（Query），计算注意力权重：
	
	$Attention(Qtext,KP,VP)=softmax(QtextKP⊤d)VP$
    其中 $Qtext=Htext​、KP=VP=P$，输出融合特征 $Hfuse∈RL×d$。

- **对比学习对齐**：对匹配的文本-图像对 (T,I)和不匹配对 (T,I−)，计算InfoNCE损失：
    
    $Lcontrast=−log⁡exp⁡(s(T,I)/τ)exp⁡(s(T,I)/τ)+∑I−exp⁡(s(T,I−)/τ)$
    其中 s(⋅)为相似度得分，τ 为温度系数。
    

## 3.3 **实体链接决策**
- **候选实体排序**：对知识库中的候选实体 ei​，计算其与融合特征 Hfuse​ 的相似度得分：
    
    $s(ei)=MLP(Hfuse)⋅Embed(ei)$
    
    使用交叉熵损失优化排序：
    
    $Lrank=−log⁡exp⁡(s(e+))∑eiexp⁡(s(ei))$
    其中 e+ 为目标实体。

## 3.4 **总损失函数**

联合优化对比学习与排序损失：

$Ltotal=Lcontrast+λLrank$
其中 λ 为平衡系数。

# 4 Quota
![image.png](https://aquazone.oss-cn-guangzhou.aliyuncs.com/20250216220838.png)



# 5 Q&A
+ Visual Prompt


# 6 Revelation
+ 多模态特征平衡
