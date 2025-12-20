# Normalization Layers 笔记（BN / LN / GN / IN / RMSNorm）

## 0. 统一记号
- CNN 常用输入：`N × C × H × W` (batch size, channel, h, w)
- Transformer 常用输入：`N × T × D` (batch size, token, dimension)
- 多数 Norm 都有可学习仿射参数：
  - `γ`（scale，缩放）
  - `β`（shift，平移；RMSNorm 常常没有 β）
- 归一化一般形式：
  - 标准化（LN/BN/GN/IN）：
    \[
    \hat x = \frac{x-\mu}{\sqrt{\sigma^2+\epsilon}},\quad y=\gamma \hat x+\beta
    \]
  - RMSNorm：
    \[
    y=\gamma\cdot \frac{x}{\mathrm{RMS}(x)+\epsilon}
    \]

---

## 1) BatchNorm（BN）
**典型输入**：`N × C × H × W`  
**统计维度**：对每个通道 `C`，跨 `N、H、W` 统计  
- `μ_c = mean_{n,h,w}(x[n,c,h,w])`
- `σ_c^2 = var_{n,h,w}(x[n,c,h,w])`

**训练 / 推理**
- 训练：用当前 mini-batch 的均值/方差；同时更新 `running_mean / running_var`
- 推理：用 `running_mean / running_var`（固定），因此 batch=1 推理没问题

**优点**
- CNN 大 batch 训练通常更快更稳
- 常有一定正则化效果（batch 统计带噪声）

**缺点 / 坑**
- 小 batch 不稳（统计噪声大）
- 推理必须 `eval()`（否则会用当前 batch 统计，batch=1 很容易出问题）
- 数据分布漂移时，running stats 可能 mismatch

**常用场景**
- ImageNet 风格 CNN 分类、大 batch 训练

**小 batch 解决**
- SyncBN / 冻结 BN / 换 GN

---

## 2) LayerNorm（LN）
**典型输入**：`N × T × D`  
**统计维度**：对每个样本 `n`、每个 token `t`，在 `D`（特征维）上统计  
- `μ_{n,t} = mean_d(x[n,t,d])`
- `σ_{n,t}^2 = var_d(x[n,t,d])`

**训练 / 推理**
- 一致（不依赖 batch，不用 running stats）

**优点**
- 对 batch size 不敏感（batch=1 也稳定）
- Transformer/LLM 训练更常见

**缺点**
- 在部分大 batch CNN 分类任务上不一定优于 BN

**常用场景**
- Transformer/LLM（Pre-LN / Post-LN 等结构）

---

## 3) GroupNorm（GN）
**典型输入**：`N × C × H × W`  
**做法**：把通道 `C` 分成 `G` 组；对每个样本 `n`、每个组 `g` 在 `(组内通道 + H + W)` 上统计均值/方差

**两种极端**
- `G = C`  → InstanceNorm（每个通道一组）
- `G = 1`  → 对每个样本在 `C × H × W` 上统一归一化（不依赖 batch）

**训练 / 推理**
- 一致（不需要 running stats）

**优点**
- 小 batch 更稳定（检测/分割常用）

**缺点**
- 大 batch CNN 分类上有时不如 BN

**常用场景**
- 目标检测 / 分割（batch 常很小）
- 常见经验：`G=32`（不是硬规定）

---

## 4) InstanceNorm（IN）
**典型输入**：`N × C × H × W`  
**等价关系**：`IN = GN with G=C`  
**统计维度**：对每个样本 `n`、每个通道 `c`，在 `H × W` 上统计均值/方差

**特点**
- 强“去风格/去对比度”（每张图每通道自己归一化）
- 可能抹掉对分类有用的全局强度信息

**常用场景**
- 风格迁移、部分生成任务

---

## 5) RMSNorm
**典型输入**：`N × T × D`（或任意向量的最后一维）  
**统计量：RMS（均方根）**
\[
\mathrm{RMS}(x)=\sqrt{\frac{1}{D}\sum_{d=1}^D x_d^2}
\]

**归一化**
\[
y=\gamma\cdot \frac{x}{\mathrm{RMS}(x)+\epsilon}
\]

**与 LayerNorm 的区别**
- LN：减均值 + 除标准差（强制“居中”）
- RMSNorm：不减均值，只控制“幅度/能量”

**优点**
- 计算更省、实现更简单
- LLM 中常很稳定、效果好

**常用场景**
- 现代 Transformer/LLM（RMSNorm 很常见）

---

## 速记：怎么选
- CNN 分类 + 大 batch：优先 BN
- 检测/分割 + 小 batch：GN（或 SyncBN / 冻结 BN）
- Transformer/LLM：LN 或 RMSNorm（现在 RMSNorm 很常见）
- 风格迁移/去风格：IN

---

## 关键区别
- BN：跨 batch 统计 → 训练/推理不一致，小 batch 易不稳
- LN/GN/IN/RMSNorm：不依赖 batch → 训练/推理一致，更适合小 batch / 序列模型
- RMSNorm vs LN：RMSNorm 不居中，只控幅度；LN 居中 + 控方差
