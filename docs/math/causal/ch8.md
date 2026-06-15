# Ch 8：差分中差分（Difference-in-Differences, DiD）

## 基本思想

**差分中差分（DiD）**是一种利用**面板数据（Panel Data）**估计因果效应的方法，适用于：

- 某一事件/政策在某个时间点只影响**部分个体（处理组）**
- 同时可以观测**处理前和处理后**的结果

DiD 估计量的核心思想：用**对照组的时间变化**作为处理组在没有处理时的反事实趋势。

$$
\tau_{\text{DiD}} = \underbrace{(\bar{Y}_{\text{treated, post}} - \bar{Y}_{\text{treated, pre}})}_{\text{处理组前后变化}} - \underbrace{(\bar{Y}_{\text{control, post}} - \bar{Y}_{\text{control, pre}})}_{\text{对照组前后变化}}
$$

## 平行趋势假设（Parallel Trends Assumption）

DiD 的核心识别假设：若不存在处理，处理组和对照组的**结果变化趋势完全相同**：

$$
\mathbb{E}[Y_{i,\text{post}}(0) - Y_{i,\text{pre}}(0) \mid T_i = 1] = \mathbb{E}[Y_{i,\text{post}}(0) - Y_{i,\text{pre}}(0) \mid T_i = 0]
$$

即处理组的反事实趋势等于对照组的实际趋势。

!!! warning "平行趋势无法直接检验"
    平行趋势关于**反事实**（处理组在没有处理时会如何变化），不可直接观测。

    常用的间接验证方法：

    - **事前平行趋势检验**：在处理之前有多个时间期时，检验两组趋势是否平行（*pre-trend test*）
    - **安慰剂处理组检验**：用一个未受处理的子组做"伪处理组"，检验是否出现虚假的 DiD 效应

## 回归框架

DiD 通常用以下双向固定效应（Two-Way Fixed Effects, TWFE）回归来估计：

$$
Y_{it} = \alpha_i + \lambda_t + \tau \cdot D_{it} + \varepsilon_{it}
$$

其中：

- $\alpha_i$：个体固定效应（控制不随时间变化的个体差异）
- $\lambda_t$：时间固定效应（控制两组共同面对的时间趋势）
- $D_{it} = \mathbf{1}[\text{个体 } i \text{ 在时间 } t \text{ 已接受处理}]$：处理指示变量
- $\tau$：DiD 估计量，处理的因果效应

```python
import pandas as pd
import statsmodels.formula.api as smf

# 面板数据：列包括 id, period, Y, treated, post, D (= treated * post)
df['D'] = df['treated'] * df['post']

# TWFE 回归（使用 entity 和时间固定效应）
model = smf.ols('Y ~ D + C(id) + C(period)', data=df).fit()
tau_did = model.params['D']
print(f"DiD estimate: {tau_did:.4f}")
```

## 直觉分解

当只有两组（处理/对照）、两期（处理前/后）时，DiD 估计量可以用 4 个均值计算：

$$
\hat{\tau}_{\text{DiD}} = (\bar{Y}_{11} - \bar{Y}_{10}) - (\bar{Y}_{01} - \bar{Y}_{00})
$$

其中下标第一位表示是否处理组（1=处理，0=对照），第二位表示时期（1=处后，0=处前）。

| | 处理前 | 处理后 | 差值（后-前） |
|---|---|---|---|
| 处理组 | $\bar{Y}_{10}$ | $\bar{Y}_{11}$ | $\bar{Y}_{11} - \bar{Y}_{10}$ |
| 对照组 | $\bar{Y}_{00}$ | $\bar{Y}_{01}$ | $\bar{Y}_{01} - \bar{Y}_{00}$ |
| **DiD** | | | $(\bar{Y}_{11} - \bar{Y}_{10}) - (\bar{Y}_{01} - \bar{Y}_{00})$ |

DiD 消去了：

1. 不随时间变化的处理组 vs 对照组的**系统性差异**（第一个差分）
2. 两组共同面对的**时间趋势**（第二个差分）

##  事件研究设计（Event Study）

在有多个处理前时期的面板数据中，可以用**事件研究（Event Study）**检验平行趋势并估计动态处理效应：

$$
Y_{it} = \alpha_i + \lambda_t + \sum_{k \neq -1} \beta_k \cdot \mathbf{1}[t - T_i^* = k] + \varepsilon_{it}
$$

其中 $T_i^*$ 是个体 $i$ 的处理时间，$k$ 是相对处理时间。

- $k < 0$（处理前）：$\beta_k$ 应接近 0（平行趋势的图形证据）
- $k \geq 0$（处理后）：$\beta_k$ 给出处理效应随时间的动态变化

## 交错处理（Staggered DiD）

当不同个体在不同时间接受处理（staggered adoption）时，传统 TWFE 估计量可能存在**负权重问题**，得出错误的效应方向。

近年来涌现出一批针对交错处理的方法：Callaway & Sant'Anna (2021)、Sun & Abraham (2021) 等，建议用**聚合处理效应（Aggregated ATT）**替代传统 TWFE。

!!! warning "TWFE 的局限性"
    当处理效应随时间变化（dynamic effects）或处理时间存在异质性（staggered）时，TWFE 的 $\tau$ 系数是所有组别、时期处理效应的加权平均，某些权重可能为负，导致估计量失去可信度。

## DiD 的例子

| 研究 | 处理 $T$ | 处理时间 | 结果 $Y$ |
|---|---|---|---|
| Card & Krueger (1994) | 新泽西州提高最低工资 | 1992年 | 快餐店就业人数 |
| 网约车政策评估 | 滴滴进入某城市 | 各城市不同年份 | 出租车价格/事故率 |
| 封锁政策效果 | COVID-19 封城 | 各地不同时间 | 感染率/经济活动 |

## Terms

!!! info "术语小结"
    - **Difference-in-Differences（差分中差分，DiD）**：利用处理前后和组间双重差分估计因果效应
    - **Panel Data（面板数据）**：同一批个体在多个时间点的观测数据
    - **Parallel Trends Assumption（平行趋势假设）**：若无处理，处理组和对照组趋势相同
    - **Individual Fixed Effects（个体固定效应）**：控制不随时间变化的个体特征
    - **Time Fixed Effects（时间固定效应）**：控制随时间变化但对所有个体相同的因素
    - **TWFE（Two-Way Fixed Effects）**：同时包含个体和时间固定效应的回归模型
    - **Pre-trend Test（事前趋势检验）**：检验处理前两组趋势是否平行的辅助检验
    - **Event Study（事件研究）**：估计处理效应随时间动态变化的方法
    - **Staggered DiD（交错差分）**：不同个体在不同时间接受处理的 DiD 设计
