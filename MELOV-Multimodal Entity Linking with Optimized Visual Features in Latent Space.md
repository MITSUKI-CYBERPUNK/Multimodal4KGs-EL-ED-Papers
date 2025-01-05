https://aclanthology.org/2024.findings-acl.46/
# 1 Motivation
+ 视觉信息经常存在质量问题[[A Dual-Way Enhanced Framework from Text Matching Point of View for Multimodal Entity Linking#^8f3bcf]] 如图像模糊、不清晰或与文本提及的实体相关性低
	+ 但先前方法总是假设每个提及都与高质量、高价值、高相关性的视觉图像相关联
	+ 因此需要开发一种新方法来处理和利用差图像，而不是简单丢弃

# 2 Cotribution & Methodology
![image.png](https://aquazone.oss-cn-guangzhou.aliyuncs.com/20241224094328.png)
+ 首次系统性地分析了多模态实体链接中**差图像**（包括低质量、低价值和低相关性图像）的问题
+ 并提出了在潜在空间中优化视觉特征的方法：消除差图像的负面影响，同时保留原始图像中的隐含视觉线索
+ 利用**变分自编码器（VAE）** 挖掘异构文本特征中的共享语义信息，并生成**基于文本的视觉特定特征**：在潜在空间中生成与文本信息相一致的视觉特征
+ **内模态聚合**：构建提及图并使用图卷积网络（GCN）聚合语义相似邻居的视觉信息
+ **信息自适应融合机制**：通过注意力机制评估原始视觉特征、跨模态生成的视觉特征和内模态聚合的视觉特征对MEL任务的贡献，并在潜在空间中以不同比例融合这些特征


# 3 Quota
![image.png](https://aquazone.oss-cn-guangzhou.aliyuncs.com/20241224095047.png)

# 4 Q&A
+ 跨模态(VAE)+模态内(GCN) 优化 原理补全