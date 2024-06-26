# 比特币中的数字签名

## ECDSA
比特币（BTC）在设计初期使用椭圆曲线数字签名算法（ECDSA）来保证交易的安全性。ECDSA在比特币中的应用提供了强大的安全性，保证交易不可篡改且可验证。它是确保比特币网络信任和安全的关键技术之一。

### 椭圆曲线密码学基础
椭圆曲线定义：在比特币中，通常使用的是secp256k1椭圆曲线。这条曲线在一个有限域上定义，具有一定的形状，能够提供高强度的加密保护。  
公钥和私钥：私钥是一个随机选定的数，而公钥是由私钥通过椭圆曲线的数学运算得出的点。

### 生成签名的过程
使用私钥创建签名：签名是使用交易发起者的私钥生成的。这个过程包括对交易的哈希值进行加密处理。
签名组成：ECDSA签名由两部分组成，通常表示为（r, s）。  
r：是椭圆曲线点的x坐标。  
s：是对交易信息、私钥和一个随机数的数学运算的结果。
  
### 验证签名的过程
使用公钥验证签名：接收方或网络节点使用发送方的公钥来验证签名。如果签名是有效的，这表明交易是由拥有相应私钥的人发起的，从而保证了交易的真实性和不可否认性。  
确保安全性：只有创建签名的私钥持有者才能生成有效的签名，而计算私钥本身几乎是不可能的，因为这需要在有限的时间内解决非常复杂的数学问题。

### 比特币中的应用
交易认证：比特币交易使用ECDSA签名来验证交易的合法性。每笔交易都包括至少一个输入（来源资金）和一个输出（目的地资金）。  
防止篡改：ECDSA签名保证了一旦交易被签名，任何对交易内容的修改都会使签名失效，从而防止了交易的篡改。

椭圆曲线密码学提供了比传统的RSA算法更高的安全性，这意味着即使使用更短的密钥，也能提供同等甚至更高的安全性。

### 数学公式
#### 密钥生成 (genKey)
1. 随机产生私钥 $d$，选点 $G$。
2. 计算公钥 $Q = d \* G $
   
注意事项： $G$点阶数 $n$，但是根据椭圆曲线随机选点 $G$却是计算出来难点。

#### 加密 (encrypt)
1. 选择随机数r的点 $k = r \* G $，随机数r。
2. 计算密文： $C_1 = M + r \* Q $,  $C_2 = r \* G $

#### 解密 (decrypt)
1.  $C_1 - d \* C_2 = M + r \* Q - r(d \* G) = M + r \* Q - r \* Q = M $

#### 签名 (sign)
1. 对消息m使用哈希算得摘要z，摘要 $z = hash(m)$
2. 生成随机数 $k ∈ [1,n)$，计算点 $k = r \* G $
3. 摘要 $z$， $R = k $，计算 $s = k^{-1}(z + rd) \mod n $，签名为  (r, s)  
4. 计算 $s = k^{-1}(z + rd) \mod n  $，签名 =  (r, s) 即是进行授权协议
5. 以上 $(r, s)$ 即为ECDSA签名

#### 验证 (verify)
使用公钥Q和消息m，对签名 $(r, s)$ 进行验证。

1. 验证 $r, s ∈ [1, n)$
2. 计算 $z = hash(m)$
3. 计算 $u_1 = z \* s^{-1} \mod n$ 和 $u_2 = r \* s^{-1} \mod n  $
4. 计算 $(x, y) = u_1 \* G + u_2 \* Q \mod n$
5. 判断 $r == x$，若相等则验证签名正确

#### 恢复 (recover)
已知消息m和签名 $(r, s)$，恢复计算出公钥 $Q$

1. 验证 $r, s ∈ [1, n)$
2. 计算 $R = (x, y)$，其中 $x = r, y + r, y + nr, y + 2nr, ...,$ 代入R到曲线方程计算得到R
3. 计算 $z = hash(m)$
4. 计算 $u_1 = z \* r^{-1} \mod n$ 和 $u_2 = s \* r^{-1} \mod n$
5. 计算公钥 $Q = (x', y') = u_1 \* G + u_2 \* R $


## Schnorr
比特币的Schnorr签名是一种相对较新的签名算法，它与传统的椭圆曲线数字签名算法（ECDSA）相比，提供了几个关键优势。在2020年的比特币协议升级中，Schnorr签名通过BIP340提案得到了引入。以下是Schnorr签名的详细介绍：

### Schnorr签名概述
发明者：Schnorr签名由Claus Schnorr提出。  
算法特性：Schnorr签名基于椭圆曲线密码学，与ECDSA使用相同的椭圆曲线。  
简洁性和效率：Schnorr签名比ECDSA更简洁、高效。它生成的签名更小，处理速度更快。

### Schnorr签名的优势
线性性：Schnorr签名具有线性特性，使得多签名（multi-signature）操作更简单有效。这意味着可以将多个签名组合成一个单一签名，从而减少数据的大小和处理时间。  
隐私和安全性：Schnorr签名提升了隐私和安全性。它使得各种复杂的交易类型（如多签名交易）在区块链上看起来与普通交易无异，从而提高了隐私性。  
防止重放攻击：Schnorr签名包含额外的措施来防止重放攻击，这是ECDSA所缺乏的。

### 比特币中的应用
简化多签名：Schnorr签名简化了比特币网络中的多签名交易处理，减少了数据量和验证所需的时间。  
批量验证：Schnorr签名支持批量验证，允许网络同时验证多个签名，提高了整体效率。  
Taproot升级：结合Schnorr签名的Taproot升级为比特币引入了更高的效率和隐私性。

### 签名过程
私钥和公钥：与ECDSA一样，Schnorr签名使用一对私钥和公钥。  
签名生成：签名是使用私钥和交易的哈希值生成的。Schnorr签名的计算相对简单，它包括了一些线性数学运算。
   
### 全性和效率
安全性：Schnorr签名被认为至少与ECDSA同等安全，有些专家认为其安全性更高。  
效率和可扩展性：签名的大小和验证的效率对于比特币网络的可扩展性至关重要。Schnorr签名在这方面表现优异。

Schnorr签名在比特币中的引入被视为一个重大进步，它提高了交易的隐私性、效率和可扩展性。随着时间的推移，预计Schnorr签名将在比特币网络中发挥越来越重要的作用。

### 数学公式
#### 密钥生成 (genKey)
1. 在群的阶 $n$ 内随机选择一个私钥 $d$，通常从 $[1, n)$ 区间选择。
2. 计算公钥 $Q = d \cdot G$，其中 $G$ 是椭圆曲线上的基点。

#### 签名 (sign)
1. 从 $[1, n)$ 区间内随机选取一个数 $k$。
2. 计算 $r = (k \cdot G).x$（即 $k \cdot G$ 点的x坐标）。
3. 计算签名 $s = (k + d \cdot \text{hash}(m)) \mod n$，这里 $\text{hash}(m)$ 是消息 $m$ 的哈希值。
4. 签名结果是一个元组 $(r, s)$。

#### 验证 (verify)
1. 计算 $u1 = \text{hash}(m) \cdot s^{-1} \mod n$。
2. 计算 $u2 = r \cdot s^{-1} \mod n$。
3. 计算点 $R' = u1 \cdot G + u2 \cdot Q$。
4. 验证 $r$ 是否等于 $R'$ 的x坐标，如果相等，则签名有效。

#### 恢复 (recover)
假设我们有一个有效的Schnorr签名 $(r, s)$ 和消息 $m$，以下是计算公钥的步骤：
1. 计算消息的哈希值 $e = \text{hash}(m)$。
2. 计算签名 $s = k + e \cdot d$。
3. 进行 $k \cdot G$ 运算得到 $s \cdot G = k \cdot G + e \cdot d \cdot G$ 即 $s \cdot G = R + e \cdot Q$。
4. 最后，公钥 $Q = (s \cdot G - R) / e$ 计算出来。
5. 其中 $r$ 是 $R$ 的x坐标，可以计算得到 $R$。




## ECDSA VS Schnorr

| 特性       | ECDSA                                     | Schnorr                                      |
|------------|-------------------------------------------|----------------------------------------------|
| 安全性     | 基于椭圆曲线离散对数问题的困难性。随机数重用或泄露会暴露私钥。   | 基于椭圆曲线离散对数问题的困难性。对随机数的选择不敏感，即使随机数可预测，也相对安全。 |
| 效率       | 签名和验证相对复杂，计算量大。                 | 签名和验证过程更高效，特别是在批量操作时。              |
| 可组合性   | 无签名可组合性。                             | 支持签名聚合，可将多个签名合并为一个。                  |
| 签名大小   | 包含两个等长的整数`(r, s)`。                    | 同样包含`(r, s)`对，但因可组合性，最终可能存储更少的签名数据。 |
| 隐私与可证明性 | 无额外隐私保护或可证明性。                      | 签名可组合性可用于增强隐私和提供非交互式的多方协议。       |
| 实际应用   | 在众多标准和协议中广泛使用，如比特币和以太坊早期版本。    | 由于技术发展被越来越多的系统采纳，例如比特币的BIP 340提案。 |
