# 1 motivation
社交平台文本的实体消歧（Named Entity Disambiguation）有以下3点特征：

- 文本不规范（inconsistent or incomplete notations）；
- 上下文有限；
- [[zero-shot]]情况（有merging entities often unseen during training）。

想到可以**利用社交平台上文字所附照片来提供visual context辅助消歧**，从而提出**Multimodal NED**任务。

# 2 Contribution
1. 提出**MNED任务**和对应数据集**SnapCaptionsKB（未开源）**，12k Snapchat的图片+配文，KB是Freebase![v2-a0b5bf24e71beaab317912e94d3ab4b2_1440w.png](https://aquazone.oss-cn-guangzhou.aliyuncs.com/v2-a0b5bf24e71beaab317912e94d3ab4b2_1440w.png)
2. 提出**Deep Zero-shot MNED Network**（code未开源）
	+ 对两种模态的上下文的处理：用**Bi-LSTM**提取textual feature，用**CNN**提取visual feature ；（首次将视觉上下文用于命名实体消歧任务）
	- 用**Deep Levenshtein**模型来预测两字符串的approximate Levenshtein score，该模型训练后得到的Bi-Char-LSTM最终用于得到mention和entity的lexical embedding；计算mention和entity之间的字面值相似度
	- 通过KB中的relations among entities学到KB embeddings，这里用的Holographic KB implementation by (Nickel et al., 2016)；
	- Predicted Label Embeddings = Modality Attention(textual feature, visual feature, lexical embedding)
	- 让Predicted Label Embeddings接近KB embeddings，让mention和golden entity的lexical embedding接近。
![v2-9b5bf4fb4213856d923d59e0f438f3ca_1440w.png](https://aquazone.oss-cn-guangzhou.aliyuncs.com/v2-9b5bf4fb4213856d923d59e0f438f3ca_1440w.png)

+ Deep Levenshtein：传统的 NED 任务假定提及及其相关实体之间的词汇完全匹配，而在我们的任务中，必须**考虑到与每个实体相对应的提及的各种表面形式（昵称、拼写错误、记号不一致等）**![image.png](https://aquazone.oss-cn-guangzhou.aliyuncs.com/20241223205309.png)
一旦学会了从上下文信息到标签嵌入的高层映射，基于知识图谱的zeroshot方法就能提高训练数据中未见的模糊实体的实体链接性能。
# 3 Quota 
![image.png](https://aquazone.oss-cn-guangzhou.aliyuncs.com/20241223204716.png)

# 4 Q&A
+ Bi-LSTM, Inception Net等常见 MM encoder
+ Deep Levenshtein
+ golden entity的lexical embedding