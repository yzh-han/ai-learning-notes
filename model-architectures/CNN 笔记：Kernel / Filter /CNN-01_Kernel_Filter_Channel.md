# CNN 笔记：Kernel / Filter / Channel（含融合与 bias）

## 0. 先记结论（最常用的口令）
- 输入: $H×W×C_{in}$
- 输出: $H'×W'×C_{out}$
- 权重: $W \in R^{C_{out}×C_{in}×k×k}$
- bias: $b \in \mathbb{R}^{C_{out}}$ （每个输出通道一个标量）
  
- 计算（对输出通道 o）：
  $y_o = \sum_{c=1}^{C_{in}} x_c * W_{o,c} + b_o$

---

## 1. Channel（通道）是什么？
- **Channel = 特征图的张数**
- 输入通道 `Cin`：
  - 灰度图：`Cin=1`
  - RGB：`Cin=3`
  - 深层网络：上一层输出的特征图张数
- 输出通道 `Cout`：这一层输出的特征图张数（由 `out_channels` 决定）

---

## 2. Filter 和 Kernel (可学习参数)
1 个 Filter 由 $C_{in}$ 个 Kernel 组成, Filter: $W_{filter}^{C_{in}, H, W}$ , Kernel: $W_{kernel}^{H, W}$ 

类似 神经网络 cin 是节点 -> cout 是节点, 全连接 连接的 Cin * Cout 个边是Cin * Cout个kernel (Cout 个filter). filter/kernel 是可学习参数。

### 2.1 Kernel
- kernel 2维, 是学习一个channel 的特征，输入图像和特征相似的会数值更高，根据这个提取出特征图
  
### 2.2 Filter
- filter 是3维, 学习所有channel 的融合特征, 一个Cout 对应一个filter 对应一种channels特征融合提取方式？

---

## 3. 跨通道“融合”是怎么发生的？
对某个输出通道 `o`：
1) 每个输入通道各自卷积（用自己的 `k×k` 切片核 $W_{o,c}$ ）得到响应
2) 把所有通道的响应 **相加**（融合）
3) 加上该输出通道的 bias `b_o`

公式：
$$y_o(i,j)=\sum_{c=1}^{C_{in}} (x_c * W_{o,c})(i,j) + b_o$$

直觉：
- 一个 filter 学到的是“跨通道综合起来的模式”
- 响应越大表示输入局部 patch（跨通道）越匹配这个模式 → 得到一张 feature map

---

## 4. bias 是怎么加的？是矩阵吗？
- **不是一整张 `H'×W'` 的矩阵参数**
- **bias 形状是 `Cout`，每个输出通道 1 个标量**
- 加法是广播：对通道 `o`，把 `b_o` 加到该通道的所有空间位置上

$$\
Y_o = (\text{conv result}) + b_o
\$$

在常见实现（N,C,H,W）里等价于：
- `bias.reshape(1, Cout, 1, 1)` 再加到输出上

---

## 5. 一句话终极总结
- **Channel**：特征图的张数
- **Filter**：产生一个输出通道的整块权重（`k×k×Cin`），学一种“跨通道融合后的特征”
- **Kernel**：有时指 filter（整体），有时指 filter 内某个输入通道的 `k×k` 切片（别被术语绕晕）
- **融合**：同一输出通道中，对所有输入通道卷积结果求和（再加 bias）
