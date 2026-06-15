# Ch 2：潜在结果框架（Potential Outcomes Framework）

## 潜在结果
**潜在结果（Potential Outcome）**框架由 Rubin 形式化，也叫 **Rubin Causal Model**。

对于个体 $i$，定义：

- $T_i \in \{0, 1\}$：是否接受处理（treatment）
- $Y_i(1)$：若 $T_i = 1$ 时的潜在结果
- $Y_i(0)$：若 $T_i = 0$ 时的潜在结果

这两个值对每个个体**同时存在**（在思维实验中），但实际上只能观测到其中一个：

$$
Y_i^{\text{obs}} = T_i \cdot Y_i(1) + (1 - T_i) \cdot Y_i(0)
$$

未被观测到的那个叫做**反事实（counterfactual）**。

!!! warning "因果推断的根本问题"
    对于同一个个体 $i$，我们**不可能同时观测到** $Y_i(1)$ 和 $Y_i(0)$——个体在某一时刻只能处于一种状态。

    这被称为"因果推断的根本问题（Fundamental Problem of Causal Inference）"。

## 个体处理效应与平均处理效应

**个体处理效应（Individual Treatment Effect, ITE）：**

$$
\tau_i = Y_i(1) - Y_i(0)
$$

由于反事实不可观测，$\tau_i$ 无法直接算出。

转而估计**平均处理效应（Average Treatment Effect, ATE）：**

$$
\tau_{\text{ATE}} = \mathbb{E}[Y(1) - Y(0)] = \mathbb{E}[Y(1)] - \mathbb{E}[Y(0)]
$$

以及**处理组平均处理效应（ATT）**和**对照组平均处理效应（ATC）**：

$$
\tau_{\text{ATT}} = \mathbb{E}[Y(1) - Y(0) \mid T = 1]
$$

$$
\tau_{\text{ATC}} = \mathbb{E}[Y(1) - Y(0) \mid T = 0]
$$

各指标的关系（$\pi = P(T = 1)$）：

$$
\tau_{\text{ATE}} = \pi \cdot \tau_{\text{ATT}} + (1 - \pi) \cdot \tau_{\text{ATC}}
$$

## 关联差 vs 因果效应

直接比较处理组与对照组的观测结果：

$$
\mathbb{E}[Y \mid T = 1] - \mathbb{E}[Y \mid T = 0]
$$

这是**关联差（associational difference）**，通常不等于 ATE，因为两组个体可能本就不同。

**分解式：**

$$
\underbrace{\mathbb{E}[Y \mid T=1] - \mathbb{E}[Y \mid T=0]}_{\text{关联差}} = \underbrace{\tau_{\text{ATT}}}_{\text{ATT}} + \underbrace{\mathbb{E}[Y(0) \mid T=1] - \mathbb{E}[Y(0) \mid T=0]}_{\text{选择偏差（Selection Bias）}}
$$

**选择偏差（Selection Bias）**是关联差与因果效应之间差距的来源。

!!! info "何时关联差 = ATE？"
    若处理是**完全随机分配**的，则处理组与对照组在接受处理前完全可比：

    $$
    \mathbb{E}[Y(0) \mid T=1] = \mathbb{E}[Y(0) \mid T=0]
    $$

    选择偏差为零，关联差 = ATT = ATE。这正是随机对照实验（RCT）的核心优势。

## 一个直观的例子：教育与收入

- $T = 1$：上大学；$T = 0$：没上大学
- $Y$：成年后年收入
- $X$：家庭背景、智力水平等

直接比较"大学毕业生 vs 非大学毕业生"的收入：

$$
\mathbb{E}[Y \mid T=1] - \mathbb{E}[Y \mid T=0]
$$

这不等于教育的因果效应，因为愿意上大学的人本来就可能有更高的家庭资源和能力（选择偏差）。

因果推断的目标：估计"如果同一个人上了大学 vs 没上大学"——真正的反事实对比 $Y_i(1) - Y_i(0)$。

## Terms

!!! info "术语小结"
    - **Potential Outcome（潜在结果）**：在不同处理取值下，某个体会出现的结果 $Y_i(t)$
    - **Counterfactual（反事实）**：没有被观测到的那个潜在结果
    - **ITE（Individual Treatment Effect）**：个体处理效应 $\tau_i = Y_i(1) - Y_i(0)$
    - **ATE（Average Treatment Effect）**：全体个体的平均处理效应 $\mathbb{E}[Y(1) - Y(0)]$
    - **ATT（Average Treatment Effect on the Treated）**：处理组个体的平均处理效应
    - **ATC（Average Treatment Effect on the Control）**：对照组个体的平均处理效应
    - **Selection Bias（选择偏差）**：处理组与对照组本就存在的系统性差异，导致关联差 $\neq$ ATE
    - **Rubin Causal Model**：以潜在结果为基础的因果推断框架
