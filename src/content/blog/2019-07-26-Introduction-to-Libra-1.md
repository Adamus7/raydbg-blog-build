---
title: Libra 技术原理浅析（一）：基本设计
pubDate: 2019-07-26 11:36:13
tags:
- Libra
- Blockchain
---
Facebook 在 2019 年揭示了其在加密货币层面点野心。19 年 6 月，Facebook 突然宣布了自己的发币计划，Libra。其目的是想要打造一个属于“the internet of money”的时代。Facebook 希望借助 Libra 去服务全球万千用户，让所有人享受电子支付带来的便利。关于 Libra 的争议和各种政策层面分析层出不穷，但是从技术层面该如何理解 Libra，似乎并没有非常多的讨论。媒体和分析师更喜欢用夺人眼球的角度去看待 Libra，得出了很多“怪异”的预测和结论。抛开这些繁杂的概念和争议，本文希望从技术角度去理解 Libra 所构建的区块链世界。全部资料来源于 Libra 项目白皮书，另外目前 Libra 还在开发过程中，讨论仅限于白皮书中已经披露的技术细节。
<!--more-->

# Libra Blockchain 初窥
Libra 要构建一个“全新的”加密货币系统，其目标是要做到：
* 实现一个能够承载数十亿的账户的区块链系统，这个系统需要由高吞吐量，低延迟的交易能力以及一个高效的大容量存储系统；
* 极高的安全性，足以保障资金和金融数据的安全；
* 足够的灵活性以便支撑 Libra 生态的管理和金融创新。

在设计上，Libra 借鉴了当前市场上已有的区块链系统，选择了三个重要的技术方向：
* 设计了一套新的智能合约编程语言，Move；
* 选择 Byzantine Fault Tolerant 作为共识算法；
* 遵循已有的区块链数据存储模型。

## Libra Reserve
有新意的地方是 Libra Reserve 的发币机制。在传统的数字加密货币系统中，如比特币，数字货币往往是被“凭空”挖掘出来的。这种加密货币的“发行”行为是一种纯粹的“体内循环”的激励机制，其发币的动机是为了维持该加密货币系统的运转（激励矿工为用户的交易做验证、打包和执行等工作）。这些加密货币的背后并没有和实体资产有任何的锚定关系，也没有组织和机构宣布过对这些加密货币的刚性对付。而 Libra Reserve 则描绘了一个不一样“发币”机制：首先，Libra 的投资者和用户需要使用法币从 Libra 协会手中“购买”Libra 代币，当有 1 块钱的法币被 Libra 协会“收储”之后，才会有等价 1 块钱的 Libra 代币被“发行”出来（原谅我使用“收储”这个词，实在没有想到更好的动词去描述这个动作）。Libra 协会会去集中管理这些“收储”的法币，并使用这些法币去做高安全、低收益的投资，例如投资各种主权基金。这些投资的收益大部分会用来支持 Libra 生态的运转。看起来这似乎更像是一个“世界银行”，用户可以把各种法币“储蓄”到 Libra 上，然后在 Libra 的的支付网络上使用 Libra 代币去做交易，或者通过 Libra 授权经销商把代币兑换成法币。个人认为这种发币机制更多的是金融层面的概念，本文不再做更多讨论，其带来的影响和利弊还是留给专业人士去探讨。而从技术角度去看，这里实在没有什么好解释的，既没有比特币“挖矿”过程的艰辛，也不涉及复杂的执行过程。

# Libra 的设计
Libra 在设计上把自己的主要结构定义成一个“可信”数据库（a cryptographically authenticated database，不知道该如何翻译），然后通过 Libra 协议在这个数据库上维护了一个全局状态统一的总账本。Cryptographically Authenticated 的意思是说这个数据库中保存的数据都是经过密码学验证过的，可以保证数据的真实可靠。而在网络结构上，Libra 有两类节点：用户节点（Client）和验证节点（Validator）。如下图，client 可以提交或查询交易，Validator 则负责根据 Libra 协议去处理这些交易并维护账本的更新。
![1.png](/images/2019-07-26-Introduction-to-Libra-1/1.png)

## Transactions and States
State（状态）指账本中某个数据的状态（值），在不同的时间节点上，数据可能有不同的状态。Trasaction（交易）指一个去改变某些数据状态的指令。比如：
> Alice 当前账户余额是 100，这是一个状态；
> Alice 要买一个包，向商户作出了一个付款的行为，这个付款指令是一个交易；
>付款之后，Alice 的账户余额变成了 50，这是一个新的状态。

![2.png](/images/2019-07-26-Introduction-to-Libra-1/2.png)
另外，Libra 网络还维护了一个 Ledger State，这里面存储了账户地址和账户数据之间的映射。
## Transaction 模型
一个 Transaction 由下列内容组成：
* Sender Address，交易发起者的地址（不是物理意义的地址，可以理解成发起者的“银行账号”）；
* Sender's publick key，交易发起者的公钥；
* Program，交易指令；
* Gas Price，Gas 价格。Gas 是衡量计算量一个度量，每执行一定的交易代码就会产生一定 Gas，客户要为这些 Gas 买单；
* Maximum gas amount，客户愿意支付的 Gas 上限；
* Sequence number，序列号；
* Expiration time，失效时间（如果一个交易在失效时间内没有被执行，则交易作废）；
* Signature，数字签名。

乍一看和很多传统区块链系统（以太坊）的 Transaction 模型类似，比较独特的地方有两处：Program 和 Sequence number。
* Program 部分包含三个部分：
   1. Move 语言字节码组成的交易脚本
   2. （可选项）交易脚本的输入内容，在转账交易中，脚本的输入是转账的金额和接收方地址。
   3. （可选项）Move 语言字节码模块，可以理解成一个智能合约的合约内容。
* Sequence number 是当前账户所发起的交易序列号，每个账户的交易序列号从 0 开始，每完成一笔交易则序列号自增 1。

## Account 模型
Libra 中的账户模型和以太坊类似，从逻辑上看，账户是一个拥有两类资源的一个集合：Move Mouldes（程序代码）和 Move Resources（数据）。Mouldes 存储的是 Move 语言字节码，即智能合约的代码，这些代码可以去访问或更新 Resouces 中的数据。Resouces 存储是是数据部分，账户拥有的 Libra Coin 也是存储在 Resource 中。
账户的地址 Address 是一个 256bit 的值。前面提到，Libra 网络通过一个`k-v map`的形式在 Ledger State 中维护了 Address 到 Account（Mouldes 和 Resoucrces）的映射。
要创建一个 Libra 账户，首先要构造一对公私钥`(vk, sk)`，该账户的地址等于公钥`vk`的`hash`值，`address=hash(vk)`。注意，这个时候账户并没有在 Ledger State 中出现，只有当一个已经存在的账户向该地址发起一笔交易的时候，Libra 网络才会在 Ledger State 中创建有关该账户的映射结构。一个用户可以有无数个账户，一个账户下面可以有无数的 Mouldes 和 Resource。

## Versioned Database
在讨论这个部分之前，先说一点题外话。虽然 Libra 把自己称之为 Libra Blockchian，然而 Libra 似乎并没有引入区块的链式（Block chain）结构，仅在共识协议的实现上引入了“区块”的概念作为共识算法的“优化”手段。这并不奇怪，实际上业界已经意识到 Blockchian 这个词并不能代表这一类系统的核心特性，很多学者和分析师更愿意用分布式账本技术（Distributed Ledger Technology，DLT ）来称呼这一类技术。这类系统的核心是在对等的分布式环境下维护一个全网状态统一的账本，至于是不是基于区块链机制实现的，并不重要。
回到 Versioned Database，Libra 定义了一个基于 version 的三元组存储在这个数据库中：对于每一个 Version i，数据库中存储了这样的一个三元组`<Ti, Oi, Si>`，分别指交易`Ti`，交易`Ti`执行时产生的输出`Oi`，交易`Ti`执行之后的账本状态`Si`（交易`Ti`是在状态`S_(i-1)`的基础上执行的）。简单讲，Libra 这个数据库在**逻辑上**是一个线性的结构，交易记录在数据库中只能单向增长，像一部定格电影，在每个时间点上都有一个对应的世界状态的快照。

# 一笔交易的执行过程
当一笔交易请求提交到 Validator 上之后，会触发一系列的处理过程。本文以“**Alice 向 Bob 转账 10Libra**”为例来详解这个过程。
## 构造交易
首先，Alice 需要在本地构造这个交易，就像填一张“支票”一样，如下图。
![3.png](/images/2019-07-26-Introduction-to-Libra-1/3.png) 

## Validator 的工作
Libra 在 Validator 上划分出了多个逻辑组件，不同的组件负责不同的操作，这些组件包括：
* Admission control（AC）
* Virtual Machine（VM）
* Mempool
* Consensus
* Execution
* Storage

后面会结合下图去解释一个交易的执行过程
![4.png](/images/2019-07-26-Introduction-to-Libra-1/4.png)

### 接受交易
1. Validator 通过 AC 获取交易。
2. AC 通过 VM 执行交易检查，包括：使用交易中的公钥（地址）验证交易签名（基于密码学的数字签名原理：有且只有通过 Alice 的公钥解开 Alice 的签名可以获得和原文内容一致的文本；由此可以确认交易的发起者一定是 Alice 本人，且交易的内容真实可信），检查 Alice 余额是否足够，交易序列号是否正常等。
3. 当交易通过检查，AC 会把这个交易放到 Mempool 中。

### 在 Validator 之间共享交易信息
4. Mempool 中可能已经有很多笔交易了。
5. Mempool 会通过 shared-mempool 协议和其他 Validator 节点共享各自所有的已接受的交易信息。

### 打包提议
6. 假设当前 Validator 是共识过程中的 proposer/leader（不在此详细讨论 Libra 共识算法的具体过程），该节点会从 Mempool 中拿出一部分交易，打成一个区块（Block）。Consensus 模块负责同步这个块到其他 Validator 节点上。
7. Consensus 模块接下来负责协调各个 Validator 对该区块内当交易内容达成共识，包括交易记录的顺序。

### 执行区块中的交易
8. 当 Validator 们达成共识之后，这个区块（一个排序好的交易集合）会被送到 Execution 模块。
9. Execution 模块通过 VM 去按序执行区块中的交易。对于 Alice 的交易来说，执行过程在逻辑上需要把 Alice 的账户余额减少，把 Bob 的账户余额增加；物理上需要对 Resource 部分的数据进行修改。
10. 执行完成后，Execution 会把这些交易按序添加到一个临时的 Merkel 树结构中。
11. Leader 节点的共识模块再次协调所有 Validator 节点对执行结果进行确认并达成共识。

### Commit 区块
12. 当一个区块当执行结果被绝大多数当 Validator 认可之后，Execution 模块就会从刚刚的 cache 中读取之前的执行结果，然后把所有当交易提交到存储模块做持久化保存。
13. 至此，Alice 的转账交易完成，Alice 的账户余额减少了`(10 + gas) ` Libra，Bob 的账户增加`10`，Alice 的`sequence number`从`5`变成`6`。

# 小结
从基本的设计结构和流程上，Libra 似乎没有给世界带来什么特别大的惊喜。Libra Reserve 的发币机制是一个特色，但更多的是金融层面的创新。
在后续的文章中，我们会再看一下 Libra 的 Move 语言、性能和共识算法的等相关内容。

参考资料：
https://libra.org/en-US/white-paper/
https://developers.libra.org/docs/assets/papers/the-libra-blockchain.pdf
https://www.raydbg.com/2018/Understand-Blockchain-and-Bitcoin/
