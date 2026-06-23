---
counter: True
---

# Matrix-Understanding

### 向量点积

两个向量的点积定义为对应分量相乘后求和：

$$
\mathbf{a} \cdot \mathbf{b} = \sum_{i} a_i b_i
$$

例如：

$$
\mathbf{a} = \begin{bmatrix} 2 \\ 3 \end{bmatrix}, \quad
\mathbf{b} = \begin{bmatrix} 4 \\ 1 \end{bmatrix}, \quad
\mathbf{a} \cdot \mathbf{b} = 2 \times 4 + 3 \times 1 = 11
$$

**几何意义**：点积衡量两个向量的**方向相似程度**。等价地，$\mathbf{a} \cdot \mathbf{b} = \|\mathbf{a}\| \|\mathbf{b}\| \cos\theta$，方向越一致（$\theta$ 越小），点积越大；两向量正交时点积为零。

### 矩阵乘向量（列向量：$M = Ax$）

设：

$$
A = \begin{bmatrix} 2 & 3 \\ 1 & 0 \\ 5 & 2 \end{bmatrix}, \quad
x = \begin{bmatrix} 4 \\ 1 \end{bmatrix}
$$

#### 视角一：每行与 $x$ 做点积

结果向量的第 $i$ 个元素，等于 $A$ 的第 $i$ 行与 $x$ 的点积：

$$
Ax = \begin{bmatrix} [2,\ 3] \cdot [4,\ 1] \\ [1,\ 0] \cdot [4,\ 1] \\ [5,\ 2] \cdot [4,\ 1] \end{bmatrix}
= \begin{bmatrix} 11 \\ 4 \\ 22 \end{bmatrix}
$$

> **应用**：Linear Classifier 中，权重矩阵 $W$ 的每一行可视为一个"类别模板"，与输入向量 $x$ 做点积得到该类别的得分，衡量输入与模板的匹配程度。

#### 视角二：$A$ 的列的线性组合

$x$ 的每个分量作为系数，对 $A$ 的对应列进行加权求和：

$$
Ax = 4 \cdot \begin{bmatrix} 2 \\ 1 \\ 5 \end{bmatrix} + 1 \cdot \begin{bmatrix} 3 \\ 0 \\ 2 \end{bmatrix}
= \begin{bmatrix} 8 \\ 4 \\ 20 \end{bmatrix} + \begin{bmatrix} 3 \\ 0 \\ 2 \end{bmatrix}
= \begin{bmatrix} 11 \\ 4 \\ 22 \end{bmatrix}
$$

> **核心洞察**：矩阵乘以向量 = **对矩阵的列做加权线性组合**，结果始终落在 $A$ 的列空间（column space）中。

### 矩阵乘向量（行向量：$M = xA$）

设：

$$
A = \begin{bmatrix} 2 & 3 \\ 1 & 0 \\ 5 & 2 \end{bmatrix}, \quad
x = \begin{bmatrix} 4 & 1 & 2 \end{bmatrix}
$$

#### 视角一：$A$ 的每列与 $x$ 做点积

结果行向量的第 $j$ 个元素，等于 $x$ 与 $A$ 的第 $j$ 列的点积：

$$
xA = \begin{bmatrix} [4,1,2] \cdot [2,1,5] & [4,1,2] \cdot [3,0,2] \end{bmatrix}
= \begin{bmatrix} 19 & 16 \end{bmatrix}
$$

#### 视角二：$A$ 的行的线性组合

$x$ 的每个分量作为系数，对 $A$ 的对应行进行加权求和：

$$
xA = 4 \cdot \begin{bmatrix} 2 & 3 \end{bmatrix} + 1 \cdot \begin{bmatrix} 1 & 0 \end{bmatrix} + 2 \cdot \begin{bmatrix} 5 & 2 \end{bmatrix}
= \begin{bmatrix} 8 & 12 \end{bmatrix} + \begin{bmatrix} 1 & 0 \end{bmatrix} + \begin{bmatrix} 10 & 4 \end{bmatrix}
= \begin{bmatrix} 19 & 16 \end{bmatrix}
$$

> **核心洞察**：行向量乘以矩阵 = **对矩阵的行做加权线性组合**，结果落在 $A$ 的行空间（row space）中。

> **口诀**：
>
> - $Ax$（列向量在右）→ 看**列**：结果是 $A$ 各列的线性组合
> - $xA$（行向量在左）→ 看**行**：结果是 $A$ 各行的线性组合
> - 无论哪种形式，都可以拆解为**点积**（逐元素视角）或**线性组合**（整体视角）

### 矩阵乘矩阵

矩阵乘法 $C = AB$ 可以看作是上述两种视角的自然推广。设 $A$ 为 $m \times k$ 矩阵，$B$ 为 $k \times n$ 矩阵：

$$
C = AB, \quad C_{ij} = \sum_{l=1}^{k} A_{il} B_{lj}
$$

#### 视角一：逐元素（点积视角）

$C$ 的第 $(i,j)$ 个元素 = $A$ 的第 $i$ 行与 $B$ 的第 $j$ 列的点积：

$$
C_{ij} = \mathbf{a}_i^\top \cdot \mathbf{b}_j
$$

#### 视角二：列视角（$B$ 的每列作为输入向量）

$C$ 的第 $j$ 列 = $A$ 乘以 $B$ 的第 $j$ 列，即 $A$ 各列的线性组合：

$$
C_{:,j} = A \, \mathbf{b}_j
$$

#### 视角三：行视角（$A$ 的每行作为输入向量）

$C$ 的第 $i$ 行 = $A$ 的第 $i$ 行乘以 $B$，即 $B$ 各行的线性组合：

$$
C_{i,:} = \mathbf{a}_i^\top B
$$

#### 视角四：外积展开（列×行）

矩阵乘法还可以分解为 $k$ 个秩为 1 的矩阵之和（列向量乘以行向量）：

$$
AB = \sum_{l=1}^{k} \mathbf{a}_{:,l} \, \mathbf{b}_{l,:}
$$

> **核心洞察**：矩阵乘法的四种视角本质上是同一件事的不同切片方式。根据具体场景选择最直观的视角，能大幅降低理解难度。
