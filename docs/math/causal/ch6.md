---
counter: True
---

# Ch 6：工具变量（Instrumental Variables, IV）

当**可忽略性假设不满足**（存在未观测混淆变量 $U$ 同时影响处理 $T$ 和结果 $Y$）时，直接用观测数据估计因果效应会有偏。

工具变量（IV）方法提供了一种在这种情况下仍能识别因果效应的策略。

## 什么是工具变量？

**工具变量（Instrument）** $Z$ 需满足三个条件：

### 1. 相关性（Relevance）
$Z$ 与处理 $T$ 相关：

$$
\text{Cov}(Z, T) \neq 0 \quad \Leftrightarrow \quad Z \not\perp T
$$

检验：可以用 $F$ 统计量（第一阶段回归的 $F > 10$ 通常认为是强工具变量）。

### 2. 外生性（Exogeneity / Independence）
$Z$ 与未观测混淆无关：

$$
Z \perp U \quad \Leftrightarrow \quad Z \perp (Y(0), Y(1))
$$

$Z$ 不能通过混淆因素影响结果。这个假设**不可检验**，需要依赖领域知识和设计。

### 3. 排除限制（Exclusion Restriction）
$Z$ 只通过 $T$ 影响 $Y$，没有直接路径：

$$
Z \to T \to Y \quad \text{（唯一路径）}
$$

在因果图中表示为：

```
Z ──→ T ──→ Y
              ↑
         U ───┘   (未观测混淆)
```

!!! warning "排除限制也不可检验"
    排除限制是研究设计的核心假设，无法从数据中验证。需要论证工具变量除了通过处理变量之外，没有其他影响结果的渠道。

## Wald 估计量

对于二元工具变量 $Z \in \{0, 1\}$ 和二元处理 $T \in \{0, 1\}$，IV 估计量（Wald 估计量）为：

$$
\hat{\tau}_{\text{WALD}} = \frac{\mathbb{E}[Y \mid Z=1] - \mathbb{E}[Y \mid Z=0]}{\mathbb{E}[T \mid Z=1] - \mathbb{E}[T \mid Z=0]} = \frac{\text{$Z$ 对 $Y$ 的简约式效应}}{\text{$Z$ 对 $T$ 的第一阶段效应}}
$$

分母是"**第一阶段（First Stage）**"，衡量工具变量对处理的影响力。

!!! info "两阶段最小二乘（2SLS）"
    在连续变量和多个协变量的情形下，IV 估计通常用**两阶段最小二乘（Two-Stage Least Squares, 2SLS）**实现：

    - **第一阶段**：用 $Z$（和协变量）预测处理 $T$，得到 $\hat{T}$
    - **第二阶段**：用 $\hat{T}$ 代替 $T$ 对 $Y$ 做回归

    ```python
    from sklearn.linear_model import LinearRegression

    # 第一阶段：T ~ Z
    stage1 = LinearRegression().fit(Z.reshape(-1, 1), T)
    T_hat = stage1.predict(Z.reshape(-1, 1))

    # 第二阶段：Y ~ T_hat
    stage2 = LinearRegression().fit(T_hat.reshape(-1, 1), Y)
    tau_iv = stage2.coef_[0]
    ```

## 局部平均处理效应（LATE）

Wald 估计量估计的并不是整体 ATE，而是**局部平均处理效应（Local Average Treatment Effect, LATE）**。

LATE 仅对**顺从者（Compliers）**成立——即那些"$Z$ 改变时，$T$ 也会跟着改变"的个体。

### 四类个体（潜在顺从性）

| 类型 | $T(Z=0)$ | $T(Z=1)$ | 说明 |
|---|---|---|---|
| **Complier（顺从者）** | 0 | 1 | $Z$ 为 1 时接受处理，$Z$ 为 0 时不接受 |
| **Never-taker（永不接受者）** | 0 | 0 | 无论 $Z$ 如何都不接受处理 |
| **Always-taker（总是接受者）** | 1 | 1 | 无论 $Z$ 如何都接受处理 |
| **Defier（违抗者）** | 1 | 0 | 反向响应（单调性假设排除此类） |

在额外的**单调性假设（Monotonicity）**下（不存在 Defiers），IV 估计 LATE：

$$
\tau_{\text{LATE}} = \mathbb{E}[Y(1) - Y(0) \mid \text{Complier}]
$$

!!! tip "LATE 的政策意义"
    LATE 描述的是"那些会被政策鼓励措施影响的人"的处理效应，这在政策评估中往往是最相关的群体。

## 工具变量的例子

| 工具变量 $Z$ | 处理 $T$ | 结果 $Y$ | 说明 |
|---|---|---|---|
| 越战征兵彩票（出生日期） | 是否服兵役 | 成年后收入 | 出生日期随机，满足外生性 |
| 距离最近大学的远近 | 是否上大学 | 成年后收入 | 距离影响上学概率但不直接影响收入 |
| 随机发放优惠券 | 是否参加项目 | 项目产出 | 随机鼓励设计（Encouragement Design） |
| 季度出生（教育法规） | 受教育年数 | 收入 | 生日在年底的人上学时已超义务教育年龄 |

## 弱工具变量问题

若第一阶段效应很弱（$\text{Cov}(Z, T)$ 接近 0），即使排除限制成立，IV 估计量也会：

- 方差极大（不稳定）
- 渐近偏差严重（有限样本下）

**实践检验**：第一阶段 $F$ 统计量 $< 10$ 通常被视为弱工具变量，需要谨慎。

## Terms

!!! info "术语小结"
    - **Instrumental Variable（工具变量）**：满足相关性、外生性、排除限制三个条件的辅助变量
    - **Relevance（相关性）**：工具变量与处理变量相关
    - **Exogeneity（外生性）**：工具变量与未观测混淆无关
    - **Exclusion Restriction（排除限制）**：工具变量只通过处理影响结果
    - **Wald Estimator（Wald 估计量）**：二元 IV 的基本估计量，简约式效应 / 第一阶段效应
    - **2SLS（Two-Stage Least Squares）**：两阶段最小二乘，连续 IV 的标准估计方法
    - **LATE（Local Average Treatment Effect）**：IV 估计的因果效应，仅对顺从者成立
    - **Complier（顺从者）**：工具变量改变时处理也随之改变的个体
    - **Monotonicity（单调性）**：假设不存在违抗者，是识别 LATE 的额外假设
    - **First Stage（第一阶段）**：$Z$ 对 $T$ 的影响，衡量工具变量的强度
    - **Weak Instrument（弱工具变量）**：第一阶段效应过小，导致 IV 估计不稳定
