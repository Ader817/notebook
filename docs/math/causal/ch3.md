# Ch 3：识别假设

为了能从**观测数据**里识别（identify）因果效应，需要若干假设。本章介绍三个核心假设：SUTVA、可忽略性和正值性。

## SUTVA

**稳定单元处理值假设（Stable Unit Treatment Value Assumption, SUTVA）**包含两个子条件：

### 1. 无干扰（No Interference）
个体 $i$ 的潜在结果不受其他个体处理状态的影响：

$$
Y_i(T_1, T_2, \ldots, T_n) = Y_i(T_i)
$$

即 $Y_i$ 只取决于自身是否接受处理，与其他人的处理状态无关。

!!! warning "SUTVA 的违反"
    - **疫苗接种**：群体免疫效应使未接种者也受益（处理效应溢出）
    - **市场干预**：政府给部分农民补贴，会影响商品价格，间接影响其他农民

    当 SUTVA 违反时，需要用网络因果推断（Network Causal Inference）等扩展框架处理。

### 2. 一致性（Consistency）
处理的定义必须是明确唯一的：

$$
T_i = t \implies Y_i^{\text{obs}} = Y_i(t)
$$

即观测到的结果确实对应于所定义的那个处理版本，不存在"不同版本的处理"。

!!! info "一致性违反的例子"
    研究"减肥"（$T = 1$）对健康的影响：但减肥的方式（节食 vs 运动 vs 手术）各不相同，潜在结果也不同。这时处理的定义不够明确。

## 可忽略性 / 无混淆

**可忽略性假设（Ignorability / Unconfoundedness）**：

$$
(Y(1), Y(0)) \perp T \mid X
$$

给定协变量 $X$ 之后，处理分配 $T$ 与潜在结果条件独立。

**直觉**：在 $X$ 相同的个体中，是否接受处理几乎是"随机"的，没有我们未观测到的混淆因素在起作用。

!!! info "随机实验的黄金标准"
    完全随机化实验（RCT）中，$T$ 与 $(Y(0), Y(1))$ **无条件**独立：

    $$
    (Y(1), Y(0)) \perp T
    $$

    关联差 = ATE，不存在选择偏差。这就是随机对照实验被称为"黄金标准"的原因。

在观测研究中，可忽略性通常只能**有条件地**（给定协变量 $X$）成立，我们称之为**条件可忽略性（Conditional Ignorability）**。

### 可忽略性的含义

给定 $X$，可以在"伪随机"子组内估计因果效应：

$$
\mathbb{E}[Y(1) - Y(0) \mid X = x] = \mathbb{E}[Y \mid T=1, X=x] - \mathbb{E}[Y \mid T=0, X=x]
$$

再对 $X$ 求期望，即得到 ATE。

## 正值性

**正值性（Positivity）**：对所有 $X$ 的取值，处理概率严格在 $(0, 1)$：

$$
0 < P(T = 1 \mid X = x) < 1, \quad \forall x \text{ s.t. } P(X = x) > 0
$$

**直觉**：如果某个类型的个体只可能接受处理（或只可能不接受处理），则我们根本观测不到这类人的反事实，无从估计其因果效应。

!!! warning "正值性违反"
    - **结构性违反**：某类个体根据规则只能接受某种处理（例如年龄限制），相关处理概率为 0 或 1
    - **随机性违反**：样本量不足时，某个 $X$ 取值的子组中出现处理或对照均缺失的情况

    正值性违反会导致 IPW 估计量的权重无穷大（或无法计算），需要在分析前检验倾向得分的分布。

## 可识别性

在 **SUTVA + 可忽略性 + 正值性** 三个假设均成立时，ATE 可以从观测数据**识别（identified）**：

$$
\tau_{\text{ATE}} = \mathbb{E}_X\left[\mathbb{E}[Y \mid T=1, X] - \mathbb{E}[Y \mid T=0, X]\right]
$$

这步推导的关键：

$$
\mathbb{E}[Y(1)] \overset{\text{SUTVA}}{=} \mathbb{E}[Y \mid T=1] \quad \text{(仅在 RCT 中)}
$$

在观测数据下要先条件在 $X$ 上：

$$
\mathbb{E}[Y(1)] = \mathbb{E}_X\mathbb{E}[Y(1) \mid X] \overset{\text{Unconf.}}{=} \mathbb{E}_X\mathbb{E}[Y(1) \mid T=1, X] \overset{\text{Consist.}}{=} \mathbb{E}_X\mathbb{E}[Y \mid T=1, X]
$$

!!! tip "可识别性 ≠ 可估计性"
    可识别性是一个**理论**结果：在假设成立时，因果量在原则上可以从观测分布中计算出来。

    但在实践中，还需要足够多的数据、合适的模型和估计方法，才能得到精确的估计。

## Terms

!!! info "术语小结"
    - **SUTVA**：稳定单元处理值假设，包含无干扰和一致性两个条件
    - **No Interference（无干扰）**：个体的潜在结果不受他人处理状态影响
    - **Consistency（一致性）**：观测结果与潜在结果在对应处理值下一致
    - **Ignorability / Unconfoundedness（可忽略性）**：给定观测协变量后，处理与潜在结果条件独立
    - **Positivity（正值性）**：每种协变量取值下，接受与不接受处理的概率均大于 0
    - **Identification（识别）**：在给定假设下，因果量能够用观测数据的函数来表达
