# 动态规划

## 动态规划的经典模型

### 区间模型 -- 过桥问题

需要通过 n-1 个人, 每次通过的人数不超过两人, 每次通过(正反)必须携带唯一的通证
通过第 i(i<n-1) 位需要 a[i] 时间, 通过前 i(i<n-1) 位通过需要 opt[i] 时间, 两人单次过桥的总时间是两人间较短的时间

状态转移方程:
$opt[i] = min{ opt[i-1] + a[1] + a[i], opt[i-2] + a[1] + a[i] + a[2]*2 }$

## 背包问题

### 0/1 背包

要求:
将前 i 件物品放进容量为 v 的背包中, 且获得最大价值, 其中第 i 件需要的空间为 c[i], 价值为 w[i]

状态转移方程:
$f[i][v] = max { f[i-1][v], f[i-1][v-c[i]]+w[i] }$

解释:
f[i-1][v] : 在 v 的空间中不放置第 i 件的最大总价值
f[i-1][v-c[i]]+w[i] : 在 v 的空间中放置第 i 件的最大总价值

优化: 利用滚动数组
$f[v] = max { f[v], f[v-c[i]]+w[i] }$

延伸:
1. 要求恰好填满背包空间 V 下的最大总价值
2. 只要求小于等于背包空间 V 下的最大总价值

### 完全背包

N 种商品(每种商品都有无限个) 和 容量为 V 的背包
放入第 i 件商品消耗的空间是 c[i] , 价值是 w[i]

状态转移方程:
$f[i][v] = max { f[i-1][v-k*c[i]]+k*w[i] | 0<=k<=v/c[i] }$
优化后:
$f[i][v] = max { f[i-1][v], f[i][v-c[i]]+w[i] }$

### 多重背包

N 种商品(第 i 种商品有 m[i] 个) 和 容量为 V 的背包
放入第 i 件商品消耗的空间是 c[i] , 价值是 w[i]

状态转移方程:
$f[i][v] = max { f[i-1][v-k*c[i]]+k*w[i] | 0<=k<=min{v/c[i], m[i]} }$

优化方案:
采用二进制拆分物品, 将 m[i] 个物品拆分成容量为 1, 2, 4, 2^k, m[i]-2^k
价值分别是 1*w[i], 2*w[i], 4*w[i], 2^k*w[i], (m[i]-2^k)*w[i]
再作为 01 背包求解

### 状态压缩模型

处理数据规模较小的问题, 将状态压缩成 k 进制的整数, k 常为 2

#### 哈密顿图

G=(V, E)
定义: 无向图, 由指定的起点前往指定的终点, 途中经过所有的节点且仅经过一次

哈密顿路径: 含有图中所有节点的路径
哈密顿回路: 闭合的哈密顿路径

必要条件: 若 G=(V, E) 是一个哈密顿图, 则对于 V 的每一个非空子集 S, 均有 W(G-S)<=|S| ( 图 G 擦去属于 S 的顶点后, 剩下的子图的连接分枝(?)个数 )
充分条件: 若 G=(V, E) 是一个无向简单图, 要求 { |V|=n n>=3 }. 如果对于任意的两个顶点 v 属于 V, d(u)+d(v) >= n, 则认为该图为哈密顿图

题目: 对于一条 N(N<11) 个点的哈密顿路径的 C[1] C[2] ... C[N] 的值由三部分组成:
  1. 每个顶点的的权值 V[i] 的和
  2. 对于路径上相邻的任意两个顶点 C[i] C[i+1], 累加权值乘积为 V[i]*V[i+1]
  3. 对于相邻的三个顶点 C[i] C[i+1] C[i+2], 如果 C[i] 和 C[i+2] 之间有边, 那么累加权值三乘积为 V[i]*V[i+1]*V[i+2]
  
