---
counter: True
---

# Ch 5：估计方法

在识别假设成立（SUTVA + 可忽略性 + 正值性）的前提下，本章介绍从观测数据估计 ATE 的四种主要方法。

## 结果回归（Outcome Regression）

直接对条件期望 $\mu(t, x) = \mathbb{E}[Y \mid T = t, X = x]$ 建模，再用后门调整公式求 ATE：

$$
\hat{\tau}_{\text{ATE}} = \frac{1}{n} \sum_{i=1}^n \left[\hat{\mu}(1, X_i) - \hat{\mu}(0, X_i)\right]
$$

其中 $\hat{\mu}$ 可以是线性回归、随机森林、神经网络等任意预测模型。

```python
from sklearn.linear_model import LinearRegression
import numpy as np

# 训练结果模型：特征 = [T, X]
mu_model = LinearRegression()
mu_model.fit(np.column_stack([T, X]), Y)

# 为每个个体预测两种反事实
n = len(X)
X1 = np.column_stack([np.ones(n), X])   # 假设所有人接受处理
X0 = np.column_stack([np.zeros(n), X])  # 假设所有人不接受处理

ate_hat = (mu_model.predict(X1) - mu_model.predict(X0)).mean()
print(f"ATE estimate: {ate_hat:.4f}")
```

!!! warning "结果回归的局限"
    - 若模型设定错误（例如用线性模型拟合非线性关系），ATE 估计会有偏
    - 对协变量维度较高时，模型外推（extrapolation）到反事实区域可能不可靠

## 逆概率加权（Inverse Probability Weighting, IPW）

用**倾向性分数（propensity score）** $e(x) = P(T = 1 \mid X = x)$ 对样本重新加权，使处理组与对照组在协变量上可比：

$$
\hat{\tau}_{\text{IPW}} = \frac{1}{n} \sum_{i=1}^n \left[\frac{T_i Y_i}{e(X_i)} - \frac{(1-T_i) Y_i}{1 - e(X_i)}\right]
$$

**直觉**：倾向得分小但仍接受处理的个体在处理组中代表性不足，因此赋予较大权重，使加权后的处理组像是随机分配的。

```python
from sklearn.linear_model import LogisticRegression

# 第一步：估计倾向性分数
ps_model = LogisticRegression()
ps_model.fit(X, T)
ps = ps_model.predict_proba(X)[:, 1]   # P(T=1 | X)

# 第二步：IPW 估计量
ate_ipw = np.mean(T * Y / ps - (1 - T) * Y / (1 - ps))
print(f"IPW ATE: {ate_ipw:.4f}")
```

!!! warning "IPW 的不稳定性"
    当倾向性分数接近 0 或 1 时，权重 $1/e$ 或 $1/(1-e)$ 会非常大，导致估计量**方差爆炸**。

    常用缓解方法：
    - **截断（Trimming）**：删除倾向得分过于极端的样本
    - **标准化 IPW（Hajek Estimator）**：用权重之和归一化
    
    $$
    \hat{\tau}_{\text{Hajek}} = \frac{\sum_i T_i Y_i / e(X_i)}{\sum_i T_i / e(X_i)} - \frac{\sum_i (1-T_i) Y_i / (1-e(X_i))}{\sum_i (1-T_i) / (1-e(X_i))}
    $$

## 双重稳健估计量（Doubly Robust Estimator, DR）

结合结果回归与 IPW，形成**双重稳健（doubly robust）**估计量（也叫 AIPW，Augmented IPW）：

$$
\hat{\tau}_{\text{DR}} = \frac{1}{n} \sum_i \left[\hat{\mu}(1, X_i) - \hat{\mu}(0, X_i) + \frac{T_i(Y_i - \hat{\mu}(1, X_i))}{e(X_i)} - \frac{(1-T_i)(Y_i - \hat{\mu}(0, X_i))}{1 - e(X_i)}\right]
$$

**"双重稳健"的含义**：只要结果模型 $\hat{\mu}$ **或**倾向性分数模型 $\hat{e}$ 中有一个是正确设定的，估计量便一致（consistent）。

```python
def doubly_robust(Y, T, X, mu_model, ps_model):
    ps = ps_model.predict_proba(X)[:, 1]
    n = len(X)
    mu1 = mu_model.predict(np.column_stack([np.ones(n), X]))
    mu0 = mu_model.predict(np.column_stack([np.zeros(n), X]))
    dr = (mu1 - mu0
          + T * (Y - mu1) / ps
          - (1 - T) * (Y - mu0) / (1 - ps))
    return dr.mean()
```

!!! tip "为什么双重稳健更好？"
    - 当两个模型都正确时，DR 估计量还具有**半参数效率（semiparametric efficiency）**，方差达到理论下界
    - 结合 Cross-fitting（交叉拟合）和机器学习模型，可得到 **DML（Double Machine Learning）** 框架

## 匹配（Matching）

为处理组的每个个体找一个协变量相似的对照组个体，用对照组的观测结果作为处理组的反事实估计：

$$
\hat{\tau}_{\text{ATT}} = \frac{1}{n_1} \sum_{i: T_i = 1} \left(Y_i - Y_{\text{match}(i)}\right)
$$

### 匹配方式

| 方式 | 说明 | 优缺点 |
|---|---|---|
| **精确匹配** | 要求所有协变量完全相同 | 高维时几乎不可用 |
| **倾向性分数匹配** | 在 $e(X)$ 上最近邻匹配 | 降维，但依赖倾向得分模型的准确性 |
| **近邻匹配（k-NN）** | 在协变量空间找 $k$ 个最近的对照样本 | 需要定义距离度量，边界效应显著 |
| **卡路里匹配（Caliper）** | 只匹配距离小于阈值的样本 | 可能舍弃部分样本 |

!!! info "匹配的本质"
    匹配是一种**非参数方法**，不需要设定结果模型的具体函数形式。但它估计的通常是 ATT，而非 ATE。

## 方法比较

| 方法 | 依赖假设 | 估计目标 | 优点 | 缺点 |
|---|---|---|---|---|
| 结果回归 | 结果模型正确 | ATE | 直观，可扩展 | 模型误设时有偏 |
| IPW | 倾向性分数模型正确 | ATE | 不需结果模型 | 方差大，极端权重 |
| 双重稳健 | 两个模型之一正确 | ATE | 更强鲁棒性 | 实现略复杂 |
| 匹配 | 可忽略性 | ATT | 直观，非参数 | 高维困难，只估 ATT |

## Terms

!!! info "术语小结"
    - **Outcome Regression（结果回归）**：对 $\mathbb{E}[Y \mid T, X]$ 建模来估计因果效应
    - **Propensity Score（倾向性分数）**：$e(x) = P(T=1 \mid X=x)$，给定协变量下接受处理的概率
    - **IPW（Inverse Probability Weighting）**：用倾向性分数倒数加权的估计量
    - **Doubly Robust（双重稳健）**：结合结果回归与 IPW，只需其中一个模型正确即可一致
    - **AIPW（Augmented IPW）**：双重稳健估计量的另一个名称
    - **DML（Double Machine Learning）**：结合交叉拟合和机器学习的双重稳健框架
    - **Matching（匹配）**：为处理组找协变量相似的对照组个体来估计反事实
    - **ATT（Average Treatment Effect on the Treated）**：处理组的平均处理效应，匹配常估计此量
