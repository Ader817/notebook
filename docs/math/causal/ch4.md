# Ch 4：因果图（Causal Graphs / DAGs）

## 有向无环图

用**有向无环图（Directed Acyclic Graph, DAG）**表示变量间的因果结构：

- **节点（node）**：随机变量（可观测或不可观测）
- **有向边（directed edge）**：$A \to B$ 表示 $A$ 是 $B$ 的直接原因之一
- **无环（acyclic）**：图中不存在有向环路，即不允许 $A$ → $B$ → $A$ 这样的循环

DAG 隐含了**局部马尔可夫假设（Local Markov Assumption）**：每个节点在其**父节点（parents）**给定时，条件独立于所有非子孙节点。

由此可以写出联合分布的因式分解：

$$
P(X_1, X_2, \ldots, X_n) = \prod_{i=1}^n P(X_i \mid \text{Pa}(X_i))
$$


## 基本路径结构

DAG 中有三种基本路径模式，决定了信息是否可以在两个节点间"流动"：

| 结构 | 图示 | 特点 |
|---|---|---|
| **链（Chain）** | $X \to Z \to Y$ | $Z$ 中介（传递）$X$ 对 $Y$ 的影响 |
| **叉（Fork）** | $X \leftarrow Z \rightarrow Y$ | $Z$ 是共同原因（混淆因素） |
| **对撞（Collider）** | $X \to Z \leftarrow Y$ | $Z$ 是共同结果 |

### 链（Chain）

$$
X \to Z \to Y
$$

$X$ 通过 $Z$ 间接影响 $Y$。若**条件在 $Z$ 上**，$X$ 与 $Y$ 条件独立（路径被阻断）。

### 叉（Fork）

$$
X \leftarrow Z \rightarrow Y
$$

$Z$ 是共同原因（**混淆因素，Confounder**），导致 $X$ 和 $Y$ 相关，但并非因果关系。若**条件在 $Z$ 上**，$X$ 与 $Y$ 条件独立（路径被阻断）。

### 对撞（Collider）

$$
X \to Z \leftarrow Y
$$

$X$ 和 $Y$ 的共同结果 $Z$。在**不控制 $Z$** 时，$X$ 与 $Y$ 边际独立；一旦**条件在 $Z$ 上**，$X$ 与 $Y$ 之间会产生虚假关联（路径被打开）。

!!! warning "对撞偏差（Collider Bias）"
    在对撞结构 $X \to Z \leftarrow Y$ 中，如果我们在回归中控制了 $Z$，反而会引入 $X$ 和 $Y$ 之间的虚假关联，得出错误的因果结论。

    **例子**：研究才华（$X$）和美貌（$Y$）与成名（$Z$）的关系。成名艺人中才华与美貌可能负相关，但这并不意味着才华和美貌本身负相关——这是对撞偏差。

## d-分离（d-separation）

**d-分离**给出判断两个节点集合在给定第三集合时是否条件独立的图形准则。

**路径阻断规则**（给定集合 $\mathbf{Z}$）：

一条路径 $p$ 在给定 $\mathbf{Z}$ 后被阻断，当且仅当：

1. 路径中存在**链/叉**结构，且中间节点在 $\mathbf{Z}$ 中；或
2. 路径中存在**对撞**结构，且对撞节点及其所有后裔都**不在** $\mathbf{Z}$ 中

若集合 $X$ 和 $Y$ 之间所有路径都被阻断，则称 $X$ 与 $Y$ 在给定 $\mathbf{Z}$ 下 d-分离，记作：

$$
X \perp\!\!\!\perp Y \mid \mathbf{Z} \quad \text{（在图中）}
$$


由此推断统计独立性（马尔可夫相容性）。

## 后门准则（Backdoor Criterion）

想要估计 $T$ 对 $Y$ 的因果效应，需要阻断所有"**后门路径**"——即从 $T$ 出发、但沿着某条指向 $T$ 的边进入 $T$ 的路径。

集合 $\mathbf{X}$ 满足**后门准则**的条件：

1. $\mathbf{X}$ 中没有 $T$ 的后裔（不能控制中间变量或 $T$ 的结果）
2. $\mathbf{X}$ 阻断 $T$ 与 $Y$ 之间的所有后门路径

若 $\mathbf{X}$ 满足后门准则，则：

$$
P(Y \mid do(T = t)) = \sum_x P(Y \mid T = t, X = x) \cdot P(X = x)
$$


这也叫**后门调整公式（Backdoor Adjustment Formula）**。

!!! info "选择要控制的变量"
    - **要控制**：混淆因素（Confounders），即叉结构的共同原因
    - **不要控制**：对撞因素（Colliders）、中间变量（Mediators）、处理的后裔

    应当根据 DAG 决定控制哪些变量，而不是在回归中"尽量多地控制变量"。

## 前门准则（Front-door Criterion）

当存在不可观测混淆（隐变量 $U$），但 $T$ 对 $Y$ 的影响全部通过可观测中间变量 $M$ 传递时，可以用**前门准则**识别因果效应：

$$
P(Y \mid do(T)) = \sum_m P(M = m \mid T) \sum_{t'} P(Y \mid T = t', M = m) P(T = t')
$$


条件：$T \to M$（$T$ 阻断不到其他 $M$ 的路径）、$M \to Y$（所有 $M$ 到 $Y$ 的后门路径被 $T$ 阻断）。

## do 算子与干预分布

Pearl 引入 **do 算子**来区分观测与干预：

- $P(Y \mid T = t)$：在自然观测下，$T = t$ 的个体中 $Y$ 的分布
- $P(Y \mid do(T = t))$：若**强制将 $T$ 设为 $t$**（外部干预），$Y$ 的分布

在图中，$do(T = t)$ 等价于从 DAG 中**删除所有指向 $T$ 的边**，再令 $T = t$：

$$
\text{截断因式分解（Truncated Factorization）}: \quad P(V \mid do(T = t)) = \prod_{i: V_i \neq T} P(V_i \mid \text{Pa}(V_i)) \cdot \mathbf{1}[T = t]
$$


!!! info "do 算子 vs 条件概率"
    在叉结构 $T \leftarrow U \rightarrow Y$ 中：

    - $P(Y \mid T = 1)$ 包含 $U$ 引起的混淆，$\neq$ 因果效应
    - $P(Y \mid do(T = 1))$ 删除 $U \to T$ 的边，消除混淆，= 真正的因果效应

## Terms

!!! info "术语小结"
    - **DAG（Directed Acyclic Graph）**：有向无环图，表示变量间的直接因果关系
    - **Parent（父节点）**：直接指向当前节点的节点
    - **Chain（链）**：$X \to Z \to Y$，中间变量传递因果信息
    - **Fork（叉）**：$X \leftarrow Z \rightarrow Y$，共同原因引起虚假相关
    - **Collider（对撞）**：$X \to Z \leftarrow Y$，条件控制后引入虚假关联
    - **Confounder（混淆因素）**：同时影响处理和结果的变量
    - **Mediator（中间变量）**：位于处理到结果因果路径上的变量
    - **d-separation**：判断在给定集合下两节点集合是否条件独立的图形准则
    - **Backdoor Criterion（后门准则）**：选择控制变量集合以阻断所有后门路径的图形准则
    - **do-operator**：表示外部干预的算子，$P(Y \mid do(X = x))$
    - **Collider Bias（对撞偏差）**：不当控制对撞变量导致的虚假因果关联
