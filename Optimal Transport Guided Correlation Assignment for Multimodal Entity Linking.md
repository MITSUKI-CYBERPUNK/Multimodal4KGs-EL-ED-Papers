# 1 Motivation
- **局部相关性偏差**：传统方法（如注意力机制）依赖自动学习的局部权重分配，容易过度聚焦于表面信息（如名称重复的实体“Robert”），忽略深层多模态线索（如文本中的“Actor”与图像中人脸区域的关联）。这种局部优化导致全局语义匹配不充分。
- **模态间语义鸿沟**：文本和图像的特征分布差异大，现有方法难以有效对齐跨模态的细粒度元素（如文本词与图像补丁），影响实体链接的准确性。

# 2 Contribution
**理论创新：首次引入最优传输（OT）到多模态实体链接（MEL）**
- **问题适配**：提出将多模态相关性分配（如文本词与图像补丁、提及与实体间的匹配）建模为OT问题，通过全局优化避免传统注意力机制的局部偏差。
- **数学形式化**：设计基于**Sinkhorn-Knopp算法的OT相关性分配模块**（式11），显式建模多对多关系（如一个文本词对应多个图像区域），弥补注意力机制的不足

**方法创新：OT-MEL框架**
- **多模态与单模态联合优化**：
    - **跨模态对齐**：通过OT实现文本-图像特征的全局交互（图2中的CA模块）。
    - **单模态对齐**：对提及与实体的文本/视觉特征分别进行OT匹配（式19-24），增强细粒度语义捕捉。
- **高效实现**：提出**知识蒸馏**策略（式28），将OT学到的全局相关性迁移到轻量级注意力机制中，在仅损失0.62%性能的情况下（表2），推理速度提升显著（表4）。

# 3 Methodology
![image.png](https://aquazone.oss-cn-guangzhou.aliyuncs.com/20250126221440.png)

## 3.1 问题表述
![image.png](https://aquazone.oss-cn-guangzhou.aliyuncs.com/20250126221917.png)
其中θ* 表示最终模型，我们使用KG中的所有实体作为候选实体。

## 3.2 OT 线性规划
![image.png](https://aquazone.oss-cn-guangzhou.aliyuncs.com/20250126221849.png)
其中 Π* 为最优运输计划，其中 Πij 为从 xi 到 yj 的信息量。1n和1m分别是维数n和m的全一向量。这样，运输计划保持了两组之间的多元素相关性，并
赋予了适当的权重。

为了加速计算，Cuturi(2013)提出用**熵正则化项**来平滑问题:![image.png](https://aquazone.oss-cn-guangzhou.aliyuncs.com/20250126222539.png)
其中H(Π)为Π的熵，λ为正则化系数。在此基础上，**Sinkhorn-Knopp算法**可以有效地求解OT问题

## 3.3 实操
### 3.3.1 多模态特征提取
![image.png](https://aquazone.oss-cn-guangzhou.aliyuncs.com/20250126223243.png)
CLIP:BERT+ViT
对于实体和提及，我们使用**参数共享编码器**来确保表示空间的一致性。

### 3.3.2 多模态特征交互
目标：通过相关性分配模块（CA）实现文本与图像特征的全局对齐。 
+ 注意力机制（Attention-based Correlation Assignment）：![image.png](https://aquazone.oss-cn-guangzhou.aliyuncs.com/20250126223703.png)
	+ 使用**标准注意力机制**计算图像到文本的局部相关性权重（式6-8）。
	+ 局限性：仅关注局部相似性，缺乏全局优化。    

+ OT相关性分配（OT-based Correlation Assignment）： ![](https://aquazone.oss-cn-guangzhou.aliyuncs.com/20250126223731.png)
	+ 将文本-图像对齐建模为**OT问题**，最小化全局传输成本（式9-10）
	+ 通过**Sinkhorn-Knopp算法求解最优传输矩阵**（式11），确保全局相关性分配。
	+ 输出：文本-图像交互特征（式12）

### 3.3.3 多模态实体链接
特征匹配：softpool：max-pooling的软版本![image.png](https://aquazone.oss-cn-guangzhou.aliyuncs.com/20250126224006.png)
![image.png](https://aquazone.oss-cn-guangzhou.aliyuncs.com/20250126224422.png)
......
# 4 Quota
![image.png](https://aquazone.oss-cn-guangzhou.aliyuncs.com/20250126224523.png)


# 5 Q&A
哈达玛积
OT
SK
CLIP
# 6 Revelation
>利用最优传输（Optimal Transport, OT）来指导多模态实体链接中的相关性分配，挺有意思的，也可以多从这个点下手
