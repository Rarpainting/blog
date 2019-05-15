# 人工智能基础课

## 数学基础 -- 概率论

$P(H|D) = \dfrac{P(D|H) \cdot P(H)}{P(D)}$

贝叶斯定理:

- $P(H)$ / 先验概率: 预先设定的假设成立的概率
- $P(D|H)$ / 似然概率: 假设成立的前提下, 观测到的结果的概率
- $P(H|D)$ / 后验概率: 在观测到结果的前提下, 假设成立的概率

频率学派:

- 频率学派的概率: 可独立重复的随机试验中单个结果出现频率的 **极限**; 即试验的结果只包含有限个给本事件, 且每个基本事件发生的可能性 **相同**
- 频率学派认为先验分布是固定的, 模型参数依靠 **最大似然估计** 计算
- 假设, 是客观存在且不会改变的, 存在固定的先验分布

贝叶斯学派:

- 贝叶斯学派的概率: 随机事件的可信程度
- 贝叶斯学派认为先验分布是随机的, 模型参数依靠 **后验概率最大化** 计算
- 固定的先验分布是不存在的, 参数本身也是随机数

## 数理统计

- 数理统计, 通过可观察的样本反推断总体的性质
- 推断的工具是统计量, 统计量是样本的函数, 是个随机变量
- 参数估计通过随机抽取的样本来估计总体分布的未知参数, 包括点估计和区间估计
- 假设检验通过随机抽取的样本接受或拒绝关于总体的某个判断, 常用于估计机器学习模型的泛化错误率

## 信息论

### 信息熵

一个系统内在的混乱程度:

![熵](img/equation-message-1.svg)

$S(\Gamma)=-\sum_{x\in \Gamma} p(x)\ln p(x)$

其中:
- $\Gamma$ 是相空间
- $x$ 是相空间中的态

意义: 对单个信源的信息量和通信中传递信息的数量与效率等问题做出了解释, 并在世界的不确定性和信息的可测量性之间搭建起一座桥梁

### 信息量

如果事件 $A$ 发生的概率为 $P(A)$, 则这个事件的信息量定义为:

$H(A) = - \log_2 P(A)$

### 互信息/信息增益

$I(X; Y) = H(Y) - H(Y|X)$

$I(X; Y)$ : X 给 Y 的信息增益 -- 特征 X 对 训练集 Y 的区分度
$H(Y)$ : 未给定任何特征时, 对训练集进行分类的不确定性
$H(Y|X)$ : 使用特征 X 对训练集 Y 进行分类的不确定性

信息增益比:

$g(X, Y) = I(X; Y)/H(Y)$

### KL 散度

描述两个概率分布 P 和 Q 之间的差异的一种方法:

$D_{KL}(P||Q) = \sum_{i = 1}^n p(x_i) \log_2 \frac{p(x_i)}{q(x_i)}$

KL 散度是对额外信息量的衡量

#### 非负性

KL 散度 >=0 , 当且仅当两个分布相同时, 取等

#### 非对称性

$D_{KL}(P||Q) \ne D_{KL}(Q||P)$
即, 通过 $P(X)$ 去近似 $Q(X)$ 和 通过 $Q(X)$ 去近似 $P(X)$ 得到的偏差, 并不相同

### 最大熵

#### 最大熵思想

要猜一个概率分布时:
- 如果对该分布一无所知, 则以熵最大的均匀分布为目标
- 如果已知该分布的部分概率, 则以满足该概率的熵最大的分布为目标

#### 最大熵原理

**对于一个未知的概率分布, 最坏的情况就是 该概率 以 等可能性 取到每个可能性的取值**

- 此时随机变量的随机程度最高, 预测也是最困难的
- 最大熵原理是为了在推断未知分布时不引入多余的约束和假设, 预测的 **风险** 最小, 同时得到最不确定的结果

#### 最大熵模型

为了取得不确定性最大的条件分布, 定义了最大熵模型:

$H(p) = -\sum\limits_{x, y} \tilde p(x) p(y|x) \log_2 p(y|x)$

让该函数的取值最大化

## 形式逻辑

### 谓词逻辑

- 个体词: 可以独立存在的具体或抽象的描述对象
- 谓词: 描述个体词的属性与相互关系
- 量词: 描述个体词的数量关系, 包括全称量词 ∀($\forall$) 和存在量词 ∃($\exists$)

### 逻辑联结词

用于为不同 命题 建立联系

- 否定(¬ / $\neg$)：复合命题 ¬P ($\neg P$) 表示否定命题 P 的真值的命题, 即 "非 P"
- 合取(∧ / $\wedge$)：复合命题 P∧Q ($P \wedge Q$) 表示命题 P 和 命题 Q 的合取, 即 "P 且 Q"
- 析取(∨ / $\vee$)：复合命题 P∨Q ($P \vee Q$) 表示命题 P 和 命题 Q 的析取, 即 "P 或 Q"
- 蕴涵(→ / $\to$)：复合命题 P→Q ($P \to Q$) 表示命题 P 是命题 Q 的 *条件* , 即 "如果 P 成立, 那么 Q 成立"
- 等价((↔ / $\leftrightarrow$)：复合命题 P↔Q ($P \leftrightarrow Q$) 表示命题 P 和命题 Q 相互蕴涵，即“如果 P，那么 Q 且 如果 Q, 那么 P"

### 产生式系统

产生式系统以产生式的规则描述符号串来替代运算

产生式系统包括规则库、事实库和推理机三个基本部分

- 规则库: 专家系统的核心和基础, 存储着以产生式表示的规则集合, 其中规则的完整性, 准确性和合理性都将对系统产生直接影响
- 事实库: 存储的是输入事实, 中间结果与最终结果, 当规则库中的某条产生式的前提可与事实库中的某些已知事实匹配时, 该产生式即被激活, 其结论也就可以作为已知事实存储在事实库中
- 推理机: 用于 控制/协调 规则库和事实库 的运行, 包括了推理方式和控制策略

推理方式:
- 正向推理: 自底向上 -- 通过已知事实, 在规则库中不断选择匹配的规则前件, 得到匹配规则的后件, 进而推演出目标结论
- 反向推理: 自顶向下 -- 从目标假设出发, 通过不断用规则库中规则的后件与已知事实匹配, 选择出匹配的规则前件, 进而回溯已知事实
- 双向推理: 推理从自顶向下和自底向上两个方向进行, 直到在某个中间点汇合, 这种方式具有更高的效率

**虽然在数学定理的证明上显示出强大的能力, 可解决日常生活中的问题时却远远谈不上智能, 其原因在于常识的缺失**

### 歌德尔不完备性定理

- 第一不完备性定理: 在任何包含初等数论的形式系统中, 都必定存在一个不可判定命题

- 如果将认知过程定义为对符号的逻辑运算，人工智能的基础就是形式逻辑
- 谓词逻辑是知识表示的主要方法
- 基于谓词逻辑系统可以实现具有自动推理能力的人工智能
- 不完备性定理向 "认知的本质是计算" 这一人工智能的基本理念提出挑战

## 线性回归

线性回归的目标模型:

$f({\bf x}) = {\bf w} ^ T {\bf x} = \sum\limits_{i = 0}^{n} w_i \cdot x_i$

**对于单变量线性回归而言, 在误差函数服从正态分布的情况下， 从 几何意义出发的最小二乘法 和 从概率出发的最大似然估计 是等价的**

### 最小二乘法:

${\mathbf{w}}^* = \mathop {\arg \min }\limits_{\mathbf{w}} \sum\limits_{k = 1} {{{({{\mathbf{w}}^T}{{\mathbf{x}}_k} - {y_k})}^2}}

= \mathop {\arg \min }\limits_{\mathbf{w}} \sum\limits_{k = 1} || y_k - \mathbf{w}^T \mathbf{x}_k ||^2$

$X_k$ 代表训练集中的一个样本

在单变量线性回归任务中, 最小二乘法的作用就是, **找到一条直线, 使所有样本到直线的欧式距离之和(L^2 范数)最小**

### 最大似然估计

### 多元线性回归

理想下, 最优参数:

${\mathbf{w}}^* = (\mathbf{X} ^ T  \mathbf{X}) ^ {-1}  \mathbf{X} ^ T  \mathbf{y}$

- $\mathbf{X}$: 所有样本 ${\bf x} = (x_0; x_1; x_2; \cdots, x_n)$ 的转置共同构成的矩阵
- 要求 矩阵 $(\mathbf{X} ^ T  \mathbf{X})$ 的逆矩阵存在(满秩)

### 正则化

在实际项目中, 往往会存在多个最优解, 可能会导致过拟合

常见的解决过拟合的方式即正则化 -- 通过添加额外的惩罚项

#### 岭回归

"参数衰减"

$$|| y_k - \mathbf{w}^T \mathbf{x}_k || ^ 2 + || \Gamma \mathbf{w}|| ^ 2$$

- $\Gamma$ 季霍诺夫矩阵

#### LASSO 回归

"最小绝对缩减和选择算子"

$|| y_k - \mathbf{w}^T \mathbf{x}_k || ^ 2 + \lambda ||\mathbf{w}||_1$

- 引入稀疏性, 简化复杂问题
- 削减非关键因素

## 朴素贝叶斯

原因: 认为 各个属性 之间正交

影响朴素贝叶斯的分类是 **所有属性之间的 依赖关系 在 不同类别上的 分布**, 而不仅仅是依赖关系本身

朴素贝叶斯具体目标是 后验概率最大化, 意味着把实例划分到最可能的类中, 使分类的错误概率最小, 即期望风险最小

### 半朴素贝叶斯

目的: 考虑了部分属性间的依赖关系, 既保留属性间较强的相关性, 又不需要完全计算复杂的联合概率分布

方式: 建立独依赖关系, 假设每个属性除了类别之外, 最多只依赖一个其他属性

## 逻辑回归

逻辑回归下的映射函数(可归一化 y 值) -- Sigmoid:

$y = g(z) = \dfrac{1}{1 + e ^ {-z}} = \dfrac{1}{1 + e ^ {- (\mathbf{w} ^ T \mathbf{x})}}$

特性:
- $g'(z) = g(z)(1-g(z))$

![Sigmoid](img/d009b3de9c82d158dfb4e7218a0a19d8bc3e426f.jpg)

### 与线性回归的关系

认为: 对数几率函数的结果 $y$ 视为样本 $x$ 作为正例的可能性, $1-y$ 则为反例的可能性, 两者的比值 $0 < \dfrac{y}{1 -y} < +\infty$ 视为 几率(体现样本作为正例的相对可能性), 则对几率函数取对数:

$\ln \dfrac{y}{1 - y} = \mathbf{w} ^ T \mathbf{x} + b$

即当使用逻辑回归解决分类问题时, **线性回归的结果正是以对数几率的形式出现**

如果逻辑回归模型由条件概率分布表示:

$p(y = 1 | \mathbf{x} ) = h(x) = \dfrac{e ^ {\mathbf{w} ^ T \mathbf{x} + b}}{1 + e ^ {\mathbf{w} ^ T \mathbf{x} + b}}$

$p(y = 0 | \mathbf{x} ) = \dfrac{1}{1 + e ^ {\mathbf{w} ^ T \mathbf{x} + b}}$

合并上面两式:

$p(y | \mathbf{x};\mathbf{} ) = h(x) * (1 - h(x))$

![单事件的概率](img/equation-logic.svg)

对于给定的实例，逻辑回归模型比较两个条件概率值的大小，并将实例划分到概率较大的分类之中

学习时，逻辑回归模型在给定的训练数据集上应用最大似然估计法确定模型的参数

假设采集到了一组数据:

$\{(\bm{x}_1,y_1),(\bm{x}_2,y_2),(\bm{x}_3,y_3)...(\bm{x}_N,y_N)\}$

它们的合事件的总概率:

![P_{总} 多事件的概率](img/equation-logic-mutli.svg)

![F(w) 多事件概率的对数表示](img/equation-logic-mutli-log.svg)

其中:

![](img/equation-logic-p.svg)

对数形式的多事件概率 $F(\bm{w})$ 是该模型(逻辑回归?)的损失函数

### 求梯度

求 $F(\bm{w})$ 的梯度 $\nabla F(\bm{w})$

首先, 求 p 关于 权值 w 的导数的过程:

![](img/equation-logic-nabla.svg)

即 $p' = p(1-p)\bm{x}$

对 $F(\bm{w})$ 对 $w$ 的偏导:

![](img/equation-logic-nabla2.svg)

![](img/equation-logic-nabla3.svg)

### 求最值

由于 sigmoid 是一个 **凸函数** , 因此可以使用 梯度下降法(GD) 或者 拟牛顿法(QNM) 求解

### 与线性回归的关系

从信息论角度:**对数似然函数的最大化 可以等效为 待求模型与最大熵模型之间 KL 散度的最小化**
从数学角度: 线性回归和逻辑回归之间的渊源 来源于 非线性的对数似然函数
从特征空间: 两者的区别在于 数据判定边界的变化

### 与朴素贝叶斯的关系

- 朴素贝叶斯, 生成模型, 以 后验概率 的形式估计输入和输出的联合概率分布, 再根据联合概率分布生成 符合条件的输出
- 逻辑回归, 判别模型, 以 似然概率 的形式先估计出输入输出的联合概率分布, 再根据条件概率分布来判断 给定的输入 应该现则哪种输出

#### 联系

在特定条件下, 逻辑回归与朴素贝叶斯等效:
使用朴素贝叶斯分类器处理二分类任务, 假设对每个 $\mathbf{x}_i$ , 属性条件概率 $p(\mathbf{x}_i | Y = y_k)$ 都满足正态分布, 且正态分布的标准差与输出标记 $Y$ 无关, 那么根据贝叶斯定理, 后验概率就可以写成

$$
p(Y = 0 | X) =
\dfrac{p(Y = 0) \cdot p(X | Y = 0)}{p(Y = 1) \cdot p(X | Y = 1) + p(Y = 0) \cdot p(X | Y = 0)}
= \dfrac{1}{1 + \exp (\ln \frac{p(Y = 1) \cdot p(X | Y = 1)}{p(Y = 0) \cdot p(X | Y = 0)})}
...
\dfrac{1}{1 + \exp(\ln \frac{1 - p_0}{p_0} + \sum\limits_i(\frac{\mu_{i1} - \mu_{i0}}{\sigma_i^2} X_i + \frac{\mu_{i0} ^ 2 - \mu_{i1} ^ 2}{2\sigma_i^2}))}
$$

**朴素贝叶斯的形式和逻辑回归中条件概率 $p(y = 0 | \mathbf{x} )$ 的形式时完全一致的, 这表明朴素贝叶斯方法和逻辑回归学习到的是同一个模型**

#### 区别

- 两者的区别在于当朴素贝叶斯分类的模型不成立时, 逻辑回归和朴素贝叶斯方法通常会学习到不同的结果
- 收敛速度不同: TODO:

### 多分类模型

#### 二分类组

#### Softmax 回归

## 决策树

- 特征选择
  - 决定使用哪些特征用于划分特征空间
  - 准则: 信息增益
- 决策树生成
- 决策树剪枝
  - 对抗过拟合

### 常用决策树

#### ID3

由 增熵(Entrophy) 原理, 使用 信息增益 来决定作为父节点的分类器; 一般而言, 熵越大, 分类效果越好

#### C4.5

添加 信息增益率 避免 过拟合(overfit)

#### CART

分类回归树(Classification and Regression Tree)

采用基尼系数取代熵模型

## 支持向量机

支持向量机是一种二分类算法, 通过在高维空间中构造超平面实现对样本的分类

下：

d = \frac{|\boldsymbol{\omega}^T\boldsymbol{x}+\gamma|}{||\boldsymbol{\omega}||} (2.6)

这里:
- $||\boldsymbol{\omega}||$ 是向量 $\boldsymbol{\omega}$ 的模, 表示在空间中向量的长度
- $\boldsymbol{x}=[x_1,x_2]^T$ 就是支持向量样本点的坐标
- $\boldsymbol{\omega}, \gamma$ 就是决策面方程的参数

### 线性可分支持向量机

目标: **在给定训练数据集的条件下, 根据间隔最大化学习最优的划分超平面的过程**

在已有超平面的前提下, 特征空间中的样本点 $\mathbf{x}_i$ 到超平面的距离(几何间隔):

$r = \dfrac{\mathbf{w}^T \cdot \mathbf{x} + b}{|| \mathbf{w} ||}$

为了使每个点到最优划分超平面的距离都不小于 -1, 则需要满足约束的 函数间隔:
- $\mathbf{w}^T \cdot \mathbf{x}_i + b \ge 1, y_i = +1$
- $\mathbf{w}^T \cdot \mathbf{x}_i + b \le -1, y_i = -1$

目标: $\dfrac{1}{2} || \mathbf{w} || ^ 2$ 的最小值

线性可分支持向量机 是 **使硬间隔最大化** 的算法

### 线性支持向量机

目标: **使原始(线性可分支持向量机)的硬间隔转变为软间隔最大化**

依据: 在线性不可分的训练集中, 认为导致不可分的只是少量异常点; "排除" 这部分异常点后, 余下的大部分样本点依然满足线性可分

于是, 在上文的函数间隔下添加 $\xi$ 参数 ($\xi \ge 0$):
- $\mathbf{w}^T \mathbf{x}_i + b \ge 1 - \xi_i, y_i = +1$
- $\mathbf{w}^T \mathbf{x}_i + b \le 1 - \xi_i, y_i = -1$

目标: $\dfrac{1}{2} || \mathbf{w} || ^ 2 + C\sum\limits_{i = 1}^N \xi_i$ 的最小化

其中:
- 松弛变量($\xi \ge 0$): 以一定的代价, 允许错误的分类
- 惩罚参数($C \g 0$): 表示对误分类($\xi$)的惩罚力度

### 核函数

由于无论是线性可分向量机还是线性支持向量机, 都只能处理线性问题, 对于非线性问题则无能为力;
因此需要将原始低维空间上的非线性问题转化为新的高维空间上的线性问题

假设:

原始空间是 **低维欧几里得空间 $\mathcal{X}$** , 新空间是 **高维希尔伯特空间 $\mathcal{H}$** , $\mathcal{X}$ 和 $\mathcal{H}$ 的映射可以通过函数

$\phi (x) : \mathcal{X} \rightarrow \mathcal{H}$

表示, 核函数可以表示成映射函数内积的形式:

$K(x, z) = \phi (x) \cdot \phi (z)$

**一般的核函数都是正定核函数**

支持向量机的学习是个凸二次规划问题, 可以用 **SMO 算法(Sequential Minimal Optimization, 序列最小最优化)** 快速求解

#### 常见核函数

- 线性核:
  - $K(\mathbf{X}, \mathbf{Y}) = \mathbf{X} ^ T \mathbf{Y}$
- 多项式核:
  - $K(\mathbf{X}, \mathbf{Y}) = (\mathbf{X} ^ T \mathbf{Y} + c) ^ d$
  - $c$ 为常数, $d \ge 1$ : 多项式次数
- 高斯核:
  - $K(\mathbf{X}, \mathbf{Y}) = \exp (-\dfrac{|| \mathbf{X} - \mathbf{Y} || ^ 2}{2\sigma ^ 2})$
  - $\sigma > 0$ : 高斯带宽
- 拉普拉斯核:
  - $K(\mathbf{X}, \mathbf{Y}) = \exp (-\dfrac{|| \mathbf{X} - \mathbf{Y} ||}{\sigma})$
  - $\sigma > 0$ : 高斯带宽
- Sigmoid 核:
  - $K(\mathbf{X}, \mathbf{Y}) = \tanh (\beta \mathbf{X} ^ T \mathbf{Y} + \theta)$
  - $\beta > 0, \theta < 0$

## 集成学习

核心问题在于 **在多样性和准确性间做出折衷, 进而产生并结合各局优势的个体学习器**

对于数据的使用策略不同, 集成学习分为两类:
- 序列化: 个体学习器间存在强依赖关系
- 并行化: 个体学习器之间不存在强依赖关系

### 序列化方法

提升(Boosting): 序列化方法中的数据使用机制

自适应提升方法(Adaptive Boosting/AdaBoost): 典型的序列化方法
- 先通过改变训练数据的权重分布, 训练出一系列具有粗糙规则的弱个体分类器
- 再基于这些弱分类进行反复学习和组合, 构造出具有精细规则的强分类器

即, AdaBoost 需要解决两个主要问题: **训练数据权重调整** 的策略和 **弱分类器结果** 的组合策略

AdaBoost 可以视为使用加法模型, 以指数函数作为损失函数, 使用前向分布算法的二分类学习方法

利于降低平均意义上的偏差

### 并行化方法

自助聚合(Bootstrap Agregation)/打包(Bagging)

随机森林方法():
- 每棵决策树在选择划分属性时, 首先从节点的属性集合中随机抽取出包含 k 个属性的一个子集, 再在这个子集中选择最优的划分属性生成决策树

利于降低方差

## 聚类分析

无监督学习

目标: 学习没有分类标记的训练样本(聚类), 以揭示数据的内在性质和规律

聚类与分类的区别:
- 分类: 先确定类别再划分数据
- 聚类: 先划分数据再确定类别

## 凸集/凸函数/凸优化

凸集具有的几何性质是凸集中的任意两点都是 **无障碍可见的**