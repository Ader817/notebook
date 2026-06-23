---
counter: True
---

# Ch 7：断点回归（Regression Discontinuity, RD）

## 基本思想

**断点回归（Regression Discontinuity Design, RDD）**利用处理分配规则中的阈值来识别因果效应。

当处理 $T$ 完全由某个**连续"跑步变量"（Running Variable / Forcing Variable）** $X$ 是否超过阈值 $c$ 决定时：

$$
T_i = \mathbf{1}[X_i \geq c]
$$

由于接近阈值的个体在各方面（除 $X$ 本身外）高度相似，阈值两侧的结果差异可以归因于处理本身：

$$
\tau_{\text{RD}} = \lim_{x \downarrow c} \mathbb{E}[Y \mid X = x] - \lim_{x \uparrow c} \mathbb{E}[Y \mid X = x]
$$

!!! info "核心直觉"
    想象奖学金门槛：考了 89.9 分和考了 90.1 分的学生在各方面几乎没有区别，但一个拿到奖学金一个没有。

    这个"几乎偶然"的差异（恰好在阈值两侧）就是 RDD 识别因果效应的来源。

## 关键假设

### 连续性假设（Continuity Assumption）
在阈值 $c$ 附近，潜在结果的期望关于 $X$ 是连续的：

$$
\lim_{x \downarrow c} \mathbb{E}[Y(0) \mid X = x] = \lim_{x \uparrow c} \mathbb{E}[Y(0) \mid X = x]
$$

$$
\lim_{x \downarrow c} \mathbb{E}[Y(1) \mid X = x] = \lim_{x \uparrow c} \mathbb{E}[Y(1) \mid X = x]
$$

**直觉**：若不是因为处理分配，阈值左右的个体结果在断点处本应平滑过渡（无跳跃）。

### 无精确操控（No Precise Manipulation）
个体无法精确决定自己的 $X$ 值落在阈值哪一侧：

$$
f_X(x) \text{ 在 } x = c \text{ 处连续}
$$

其中 $f_X$ 是 $X$ 的概率密度函数。

!!! warning "操控检验（McCrary Test）"
    如果个体可以精确操控 $X$，得分密度在阈值处会出现异常聚集（density discontinuity）。

    **McCrary 检验**可以检测这种现象：若阈值处密度存在显著跳跃，则操控假设可能违反。

## 两种类型的 RDD

### 尖锐断点（Sharp RDD）
处理分配完全由阈值决定：

$$
T_i = \mathbf{1}[X_i \geq c]
$$

直接估计阈值两侧结果均值之差即可。

### 模糊断点（Fuzzy RDD）
处理概率在阈值处发生跳跃，但不是从 0 跳到 1：

$$
\lim_{x \downarrow c} P(T = 1 \mid X = x) \neq \lim_{x \uparrow c} P(T = 1 \mid X = x)
$$

此时阈值作为**工具变量**使用，估计量为：

$$
\tau_{\text{Fuzzy RD}} = \frac{\lim_{x \downarrow c} \mathbb{E}[Y \mid X = x] - \lim_{x \uparrow c} \mathbb{E}[Y \mid X = x]}{\lim_{x \downarrow c} P(T=1 \mid X=x) - \lim_{x \uparrow c} P(T=1 \mid X=x)}
$$

与 IV 的 Wald 估计量形式相同，估计 LATE。

## 估计方法

### 局部多项式回归（Local Polynomial Regression）
在阈值 $c$ 两侧各拟合一个低阶多项式，用两个多项式在 $c$ 处的预测值之差估计断点效应：

```python
import numpy as np
from sklearn.linear_model import LinearRegression

def rdd_estimate(Y, X, c, bandwidth=None, poly_order=1):
    if bandwidth is None:
        bandwidth = (X.max() - X.min()) / 4

    mask = np.abs(X - c) <= bandwidth
    X_local, Y_local = X[mask], Y[mask]

    # 以阈值为中心：x_centered = X - c
    x_c = X_local - c
    is_right = (x_c >= 0).astype(float)

    # 构造特征：多项式 × 阈值两侧
    features = np.column_stack([
        is_right,                        # 处理指示
        x_c,                             # 线性项（左侧）
        is_right * x_c,                  # 线性项（右侧，交互）
    ])

    model = LinearRegression().fit(features, Y_local)
    return model.coef_[0]   # is_right 系数 = 断点处跳跃

tau_rd = rdd_estimate(Y, X, c=0.5)
print(f"RDD estimate: {tau_rd:.4f}")
```

### 带宽选择（Bandwidth Selection）
- **带宽过窄**：样本量小，方差大
- **带宽过宽**：引入阈值远端的偏差

**MSE 最优带宽（IK 带宽、CCT 带宽）**在偏差和方差之间取得最佳平衡，是实践中的推荐做法。

## 有效性检验

RDD 分析通常需要报告以下检验，以增强结论的可信度：

| 检验 | 目的 |
|---|---|
| **密度检验（McCrary Test）** | 检验是否存在对 $X$ 的精确操控 |
| **协变量平衡检验** | 检验阈值两侧的其他协变量是否平衡 |
| **安慰剂断点检验** | 在非真实阈值处检验是否存在虚假跳跃 |
| **带宽敏感性分析** | 结果是否在不同带宽下稳健 |

## RDD 的例子

| 跑步变量 $X$ | 阈值 $c$ | 处理 $T$ | 结果 $Y$ |
|---|---|---|---|
| 学生成绩 | 奖学金分数线 | 是否获奖学金 | 学习成绩/升学率 |
| 选票占比 | 50% | 是否当选 | 政策/资金分配 |
| 年龄 | 法定饮酒年龄 | 是否合法饮酒 | 犯罪率/死亡率 |
| 考试分数 | 补习班资格线 | 是否获得补救教育 | 长期学业成就 |

## Terms

!!! info "术语小结"
    - **Regression Discontinuity Design（断点回归设计）**：利用处理分配阈值识别因果效应的方法
    - **Running Variable（跑步变量）**：决定个体是否接受处理的连续变量，也叫 Forcing Variable
    - **Cutoff / Threshold（阈值）**：跑步变量决定处理的临界点 $c$
    - **Sharp RDD（尖锐断点）**：处理完全由是否超过阈值决定
    - **Fuzzy RDD（模糊断点）**：阈值处理概率跳跃，但不是从 0 到 1
    - **Continuity Assumption（连续性假设）**：潜在结果在阈值处连续，是 RDD 的核心识别假设
    - **Manipulation（操控）**：个体精确操控跑步变量使自己落在有利的一侧，违反 RDD 假设
    - **McCrary Test**：检验跑步变量密度是否在阈值处发生跳跃的统计检验
    - **Bandwidth（带宽）**：局部估计时使用阈值附近多大范围内的数据
    - **Local Polynomial Regression**：在阈值附近拟合低阶多项式来估计极限值
