# 🧠 AI 深度学习算子与架构符号速查表 (AI Operators & Symbols Cheat Sheet)

在阅读深度学习论文（如 Transformer, ResNet, LSTM）的架构图时，理解这些符号是瞬间看懂网络结构的关键。

---

## 1. “圆圈”家族：逐元素操作 (Element-wise Ops)
> **核心概念**：
> 这类操作通常**不改变张量形状**（Shape）。要求输入张量形状一致，或支持广播（Broadcasting）。

| 符号 | 数学名称 | 深度学习含义 | 典型应用场景 |
| :---: | :--- | :--- | :--- |
| $\oplus$ | **Direct Sum** | **逐元素相加** (Element-wise Add)<br>公式：$x + y$ | **ResNet (残差连接)**<br>保留原始信息，防止梯度消失。<br>输入绕过中间层直接加到输出。 |
| $\otimes$ | **Tensor Product** | **逐元素相乘** (Hadamard Product)<br>公式：$x \odot y$ | **LSTM / Attention (门控机制)**<br>像“水龙头”一样控制流量。<br>常配合 $\sigma$ 使用：保留或遗忘信息。 |
| $\odot$ | **Hadamard Product** | **逐元素相乘** (同上)<br>更严谨的数学写法 | **公式推导**<br>图示中常用 $\otimes$，公式中常用 $\odot$。 |
| $\ominus$ | **Minus** | **逐元素相减** | **Siamese Network (孪生网络)**<br>计算两个特征向量的距离/差异。 |
| $\oslash$ | **Division** | **逐元素相除** | **Normalization**<br>存在于 BatchNorm/LayerNorm 计算逻辑中。 |
| $\circledast$ | **Convolution** | **卷积** | **CNN 公式**<br>简写卷积操作：$Y = W \circledast X$。 |

---

## 2. 连接与变换：结构操作 (Structure Ops)
> **核心概念**：
> 这类操作通常会**改变张量的形状**（Shape）或组合方式。

| 符号 | 名称 | 含义 | 典型应用场景 |
| :---: | :--- | :--- | :--- |
| $\|$ 或 `Concat` | **Concatenation** | **拼接**<br>维度增加，不进行数值混合。 | **UNet / DenseNet / Transformer**<br>特征融合。例：`[32, 10]` 拼上 `[32, 20]` $\rightarrow$ `[32, 30]`。 |
| $\times$ 或 $\cdot$ | **Matrix Mul** | **矩阵乘法** (Dot Product)<br>核心计算算子。 | **Linear / Attention**<br>$Q \cdot K^T$ 是注意力机制的核心计算。 |
| $T$ | **Transpose** | **转置**<br>交换维度。 | **Attention**<br>常作 $X^T$，调整矩阵方向以便相乘。 |

---

## 3. 希腊字母：函数与统计 (Functions & Stats)
> **核心概念**：
> 代表具体的数学变换函数或统计量，常作为节点（Node）出现。

| 符号 | 名称 | 作用 / 区间 | 说明 |
| :---: | :--- | :--- | :--- |
| $\sigma$ | **Sigmoid** | $(0, 1)$ | **概率 / 门控开关**。<br>输出接近 1 代表“开”，接近 0 代表“关”。 |
| $\tanh$ | **Tanh** | $(-1, 1)$ | **双曲正切**。<br>LSTM 内部常用，处理正负特征信息。 |
| $\text{ReLU}$ | **ReLU** | $[0, \infty)$ | **非线性激活**。<br>$\max(0, x)$，目前最通用的激活函数。 |
| $\mu$ | **Mu** | 均值 (Mean) | VAE (变分自编码器) 或 BN/LN 层统计量。 |
| $\sigma^2$ | **Sigma Sq** | 方差 (Variance) | 描述分布的离散程度。 |

---

## 4. ⚡️ 快速识图口诀 (Patterns)

在复杂架构图（如 LSTM, Transformer）中，寻找以下组合模式：

* **看到 $\oplus$ (圈加)** $\rightarrow$ **ResNet / 残差**
    * *含义*：把旧的信息加回来，“我全都要”。
* **看到 $\sigma \rightarrow \otimes$ (Sigmoid 连着 圈乘)** $\rightarrow$ **Gating / 门控**
    * *含义*：用 Sigmoid 生成 0~1 的信号，乘到原始数据上，用来“筛选/控制”信息流（如 LSTM 的遗忘门）。
* **看到 $\|$ (两条竖线)** $\rightarrow$ **Concat / 特征融合**
    * *含义*：把不同来源的信息拼在一起，增加特征丰富度。

---

> *Note: 数学符号在不同文献中可能略有差异，但上述定义涵盖了 95% 的深度学习论文场景。*
