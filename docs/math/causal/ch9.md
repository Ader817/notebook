# Ch 9：结构因果模型（Structural Causal Model, SCM）

## 什么是 SCM？

**结构因果模型（Structural Causal Model, SCM）**是 Judea Pearl 提出的形式化因果推断框架，是一个三元组 $(U, V, F)$：

- $V$：**内生变量（endogenous variables）**，即可观测的随机变量
- $U$：**外生变量（exogenous variables）**，即背景 / 噪声变量，可以不可观测
- $F$：**结构方程集合（structural equations）**，每个 $V_i$ 对应一个方程：

    $$
    V_i = f_i(\text{Pa}(V_i), U_i)
    $$

    其中 $\text{Pa}(V_i)$ 是 $V_i$ 在 DAG 中的父节点集合

每个结构方程描述了"$V_i$ 的取值由哪些因素决定"，是**因果机制（causal mechanism）**的形式化表达。

!!! info "结构方程 vs 回归方程"
    回归方程是对条件期望的统计描述，可以任意改写（例如把 $Y = aX + b$ 改成 $X = (Y-b)/a$）。

    结构方程描述的是单向因果机制：$Y = aX + U_Y$ 意味着 $X$ 决定 $Y$，而**不能**反过来写 $X$ 导致 $Y$。方向有因果含义，不能随意颠倒。

## SCM 与 DAG 的关系

从 SCM 可以自然地导出对应的 DAG：每个结构方程 $V_i = f_i(\text{Pa}(V_i), U_i)$ 对应图中 $\text{Pa}(V_i) \to V_i$ 的有向边。

DAG 是 SCM 的图形摘要，SCM 在 DAG 基础上增加了**函数形式**和**噪声分布**的完整规定。

## 三类推断：Pearl 因果层级

Pearl 提出了**因果层级（Ladder of Causation）**，将因果推断问题分为三个层次：

| 层级 | 活动 | 典型问题 | 符号 |
|---|---|---|---|
| **Layer 1：关联（Association）** | 观察（Seeing） | $X$ 与 $Y$ 同时出现吗？ | $P(Y \mid X)$ |
| **Layer 2：干预（Intervention）** | 行动（Doing） | 若我将 $X$ 设为 $x$，$Y$ 会怎样？ | $P(Y \mid do(X = x))$ |
| **Layer 3：反事实（Counterfactual）** | 想象（Imagining） | 若当初 $X$ 不同，$Y$ 会怎样？ | $P(Y_x \mid X = x', Y = y')$ |

**关键性质**：更高层次的问题无法仅用更低层次的信息回答。

- 仅有观测数据（Layer 1）无法回答干预问题（Layer 2）
- 仅有干预数据（Layer 2）无法回答反事实问题（Layer 3）——需要 SCM

## 干预：do 算子

**干预（Intervention）$do(X = x)$**：强制将 $X$ 设为某值，不管 $X$ 的自然取值机制。

在 SCM 中，$do(X = x)$ 等价于：

1. 将结构方程 $X = f_X(\text{Pa}(X), U_X)$ **替换**为 $X = x$（常数方程）
2. 其他变量的方程保持不变
3. 对修改后的 SCM 做推断

在图中表示为：删除所有指向 $X$ 的边，再令 $X = x$。

$$
P(V \mid do(X = x)) = \prod_{i: V_i \neq X} P(V_i \mid \text{Pa}(V_i)) \cdot \mathbf{1}[X = x]
$$


这叫**截断因式分解（Truncated Factorization）**。

## 反事实：个体层面推断

**反事实（Counterfactual）**询问的是：在已知个体的实际背景 $U = u$ 下，若其某个变量值不同，结果会如何？

在 SCM 中，计算反事实 $Y_x(u)$ 分三步（**abduction-action-prediction**）：

### Step 1：溯因（Abduction）
利用观测数据 $(X = x, Y = y, \ldots)$ 推断背景噪声 $U$ 的后验分布：

$$
P(U \mid \text{evidence})
$$


### Step 2：行动（Action）
修改 SCM，令 $do(X' = x')$（假设的反事实干预）。

### Step 3：预测（Prediction）
用修改后的 SCM 和推断出的 $U$ 计算 $Y$ 的反事实值：

$$
Y_{x'}(u) = f_Y(\text{Pa}(Y) \text{ under } do(X' = x'), u)
$$


!!! info "反事实的例子"
    - "如果我当时没有吸烟，现在会不会得肺癌？"（已知结果，询问不同干预）
    - "如果这个病人没有接受手术，他会不会康复？"（评估已发生的治疗决策）

## 识别层级的总结

| 研究设计 | 可回答的层级 |
|---|---|
| 纯观测数据 | Layer 1（关联） |
| 随机对照实验（RCT） | Layer 2（干预），可以识别 ATE |
| 观测数据 + 后门准则 | Layer 2（干预） |
| 完整 SCM（含 $U$ 分布） | Layer 3（反事实） |

## 线性 SCM 举例

考虑一个简单的线性 SCM：

$$
X = U_X, \quad Y = \alpha X + U_Y, \quad U_X \sim N(0,1), \quad U_Y \sim N(0,1)
$$


- **观测分布**：$P(Y \mid X = x) = N(\alpha x, 1)$
- **干预分布**：$P(Y \mid do(X = x)) = N(\alpha x, 1)$（此处两者恰好相同，因为没有混淆）
- **反事实**：若已知 $X = 1, Y = 2$，可推断 $U_Y = 2 - \alpha$，进而计算 $Y_{x=0} = U_Y = 2 - \alpha$

引入混淆因素 $Z \to X, Z \to Y$ 后，观测分布和干预分布不再相同，需要用后门准则调整。

## Terms

!!! info "术语小结"
    - **SCM（Structural Causal Model）**：Pearl 提出的因果推断形式化框架，包含内生变量、外生变量和结构方程
    - **Structural Equation（结构方程）**：描述变量因果机制的方程，方向不可任意颠倒
    - **Endogenous Variable（内生变量）**：模型中由其他变量决定的可观测变量
    - **Exogenous Variable（外生变量）**：外部背景 / 噪声变量，不被模型内其他变量决定
    - **Ladder of Causation（因果层级）**：关联 → 干预 → 反事实，三个层次的因果问题
    - **Intervention（干预）**：$do(X = x)$，强制设定变量值，删除指向该变量的所有边
    - **Counterfactual（反事实）**：在已知背景下，假设不同干预时结果的取值
    - **Abduction-Action-Prediction**：计算反事实的三步算法（溯因→行动→预测）
    - **Truncated Factorization（截断因式分解）**：干预后 SCM 的联合分布表达式
    - **Causal Mechanism（因果机制）**：结构方程所描述的变量之间的因果决定关系
