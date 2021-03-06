# 区块链

## 分布式系统

作为分布式系统的区块链, 首先需要考虑的是一致性的保障

而理想的分布式系统, 需要满足:

- 可终止性(Termination): 一致的结果能够在*有限时间*内完成
- 共识性(Consensus): 不同节点最终完成决策的结果应该相同 (提示: 顺序执行)
- 合法性(Validity): 决策结果必须是其他进程提出的提案

分布式系统无法实现的**强一致性**包括:
- 顺序一致性
- 线性一致性

为此, 分布式系统提出了**最终一致性**的概念, 以及相应的一致性解决方案

### 共识问题

#### 非拜占庭错误 -- 故障不响应

常用算法 -- Paxos/Raft

#### 拜占庭错误 -- 恶意响应

常用算法 -- PBFT/PoW

### FLP 不可能性原理

即使在网络通信可靠的前提下, 一个可扩展的分布式系统的共识问题的下限是**无解**的

#### CAP 原理

一致性(Consistency) 可用性(Availablity) 分区耐受性(Partition)

- ACID 原则: 原理性(Atomicity) 一致性(Consisitency) 隔离性(Isilation) 持久性(Durability)

- BASE 原则: 基本可用(Basic Availiability) 软状态(Soft state) 最终一致性(Eventually Consistency)

#### Paxos 与 Raft

##### Paxos -- 二阶段提交

解决分布式系统中存在 故障(fault), 即信息丢失或重复, 但不存在恶意(corrupt)节点场景, 即无错误信息, 下的共识达成(Consensus)问题

##### Raft -- Paxos 简化

### 密码学技术

一个优秀的 Hash 算法 能够实现:
1. 正向快速
2. 逆向困难
3. 输入敏感
4. 冲突避免

#### HTTPS 过程 -- 混合加密机制

安全连接建立:
- (明文) 客户端发送信息(随机数 R1, 支持的加密算法类型, 协议版本, 压缩算法等)到服务器
- (明文) (私钥签名) 服务端返回信息(随机数 R2, 选定的加密算法类型, 协议版本, 服务器证书(公钥))
- (公钥认证) 浏览器通过权威 CA 的根证书检查公钥证书
- (公钥加密) 使用公钥加密随机数 R3 发送到服务器
- (私钥解密) 服务器解密 R3, 同时双方通过 3 个随机数(R1 R2 R3) 生成对称的会话密钥(AES 密钥), 后续通信通过对称加密进行保护

#### 数字签名

##### HMAC (Keyed-Hashing for Message Authentication) -- 基于 HASH 的消息认证码

- 对 **散列算法** 添加 **混淆** 的概念

HMAC(K H Message)
K - 提前共享的对称密钥
H - 提前商定的 Hash 算法

##### 盲签名

RSA 盲签名 -- 在无法获得原始内容的前提下对信息进行签名

##### 多重签名

n 个持有者中, 收集到至少 m 个签名, 即认为合法
n - 提供的公钥数
m - 需要匹配的最少签名数

##### 群签名

##### 环签名

#### 数字证书 -- PKI(Public Key Infrastructure) 体系

管理 分发证书

PKI 至少包含以下组件:
- CA : 负责证书的颁发和作废 接受来自 RA 的请求
- RA : 对用户身份进行认证 检验数据合法性 负责登记
- 证书数据库 : 存放证书

#### 同态加密

```
存在
e = E(m)
其中
E: 加密操作
m: 明文
e: 密文

同时存在
F(e) = E(f(m))
其中
F: 针对 E 的构造器
f: 针对明文 m 的操作

则称 E 为针对 f 的同态加密算法
```

相关概念:
代数同态: 同时满足加法同态和乘法同态, 也称全同态
算数同态: 同时满足 加法/乘法/减法/除法 四种同态性

##### 函数加密

保护处理函数本身

目前不存在对多个通用函数的任意多 KEY 的方案, 仅能做到对某个特定函数的一个 KEY

### 比特币项目

#### 闪电网络

##### RSMC(Recoverable Sequence Contract)

可撤销的顺序成熟度合同

保障了两个人之间的交易可以直接在链下完成

##### HTLC(Hashed Timelock Contract)

哈希的带时钟的合约

保障了任意两人间的转账都可以通过一条 通道 完成

### 以太坊项目
