---
title: Libra 技术原理浅析（二）：Move 语言
pubDate: 2019-08-09 13:00:45
tags:
- Libra
- Blockchain
---
在上一篇博客中讨论了 Libra 区块链的基本设计和交易执行的基本过程。Libra 区块链还有其他一些值得关注的技术点，包括：Move 语言，以及 Libra 共识算法等。在这篇文章中，我希望记录一下透过白皮书所看到的 Move 语言的设计细节，解释一下 Move 语言的独特之处。
<!--more-->

# 开始之前
为何区块链需要一套编程语言？我个人的理解，这是比特币的优雅工程实现给后来人带来的美妙灵感。比特币作为一个自治的数字货币系统，唯一操作就是转账。当一个 transcastion 被提交之后，矿工需要对 transcation 做一系列的验证操作，包括验证签名、验余额等，再做一些数据操作。逻辑上，这些操作看起来是在账本上记录了一个用户用掉一张“支票”的过程。美妙的地方在于比特币的代码实现过程：比特币设计了一个虚拟机，然后把 transcation script 相应地翻译成虚拟机的 bytecode，由此把验证和数据操作交给一个 VM 去自动执行。就是说，一个账本上的记账过程变得和执行程序一样优雅。
从这个角度出发，如果把这个虚拟机的能力做拓展（从非图灵完备到图灵完备，支持更多指令集），是不是就可以在区块链上执行更复杂的“程序”，实现更多的功能，且保留区块链的其他优秀特性？我猜想这可能是以太坊、智能合约以及很多后来者的灵感来源。也正是由此，区块链技术需要配套的编程语言来帮助开发者实现“转账”以及更多“状态”改变的逻辑。用可读性较高的程序语言去实现业务逻辑，再把这个程序语言翻译成区块链虚拟机支持的机器语言。

# 为何又造了一个轮子？
以太坊及一众跟随者已经给出了各种区块链编程语言，以 Solidity 为代表，这些编程语言和模型各有各的特点。在市场和开发者对 Solidity 等智能合约语言接受程度较高对情况下，为何 Libra 又要给这个世界带来一个新的轮子？（这是我去看白皮书对动力之一）。
## 现在的轮子有什么问题？
比特币的转账 script 虽然很精妙，但是只能做转账，没有任何 customize 的价值。以太坊的 EVM（Ethereum Virtual Machine）和 Solidity 的实现也非常漂亮，完美的证明了区块链不仅仅可以做支付，还能做更多有想象力的工作。但是，Solidity 也给开发者留下了一些很难克服的“坑”。我们知道以太坊是有自己的货币的，Ether。但是抛开这个，用户也可以通过智能合约去实现各种“代币”逻辑，类似于：
- 在智能合约中定义一个 kv map；
- k 是账户地址，v 是变量；
- 合约中实现的方法可以根据不同的逻辑去改变某个 k 对应的 v 的 value。

因为智能合约中定义的 kv map 的值是全网统一的，因此，可以把 v 的 value 看成是账户 k 在区块链上的一种**数字资产**。具体的，比如一个基于智能合约的库存管理系统，其可以在合约中定义不同仓库对某个商品的库存量（这里我避免举各种“代币”、“积分”和“点券”的例子，避免限制读者对区块链的想象力，代币的例子可以参考蚂蚁区块链 BaaS 平台的 [样例](https://tech.antfin.com/docs/2/101920)）。那么我们可以根据实际的逻辑，对商品的库存量进行各种操作。比如，从仓库 a 把 n 个商品移到仓库 b，从合约代码的角度看，就是把 a 的对应的库存量 v 减去 n，把仓库 b 对应的库存量 v 加上 n。
很简单对吧，这会有什么问题呢？以太坊和比特币一样对于原生货币 Ether 的操作有原生的各种验证操作，比如说一笔 Ether 的转账，EVM 需要验证发起者是不是实际的拥有者、发起者是不是有足够的余额等。但是，对于基于合约定义的这些数字资产，安全性检查的操作只能由**开发者自己来维护**。假如合约的逻辑存在缺陷，那么会带来各种无法挽回的损失。比如，本来是从仓库 a 把 n 个商品移到仓库 b，合约代码逻辑类似于：
```
map[a] = map[a]-n;
map[b] = map[b]+n;
```
但是开发者写错了：
```
map[a] = map[a] - n;
map[b] = map[b] + 2*n;
```
那么这个操作带来的结果就错的离谱了，且这种错是无法“挽回”的。
所以，现有轮子的问题是：**只对原生数字资产的操作有强有力的验证保护过程，而对基于合约变量的数字资产操作缺乏保护。** 合约变量相比于原生币变量，像是一个**二等公民**。虽然以太坊社区有了 ERC-20 这样的代币协议标准，但是这种代币资产的安全性还是需要复杂的主动逻辑检查。在以太坊的历史上，也确实发生过因合约逻辑缺陷而带来严重的 DAO 攻击事件。实际上人们为了保证这种代币资产的安全，也在 Solidity 的基础上造了各种各样的轮子。
Libra 的愿景不仅是一个构建一个“全球电子货币”，还要能**支撑各种金融创新**。因此，对于数字资产的保护，也是其重点之一。设想某个赌场组织基于 Libra 合约发行了一套*赌场代币*系统，如果这个合约中存在逻辑缺陷，那么带来的影响将会是致命的。
因此，Libra 设计了全新的轮子，Move 语言，试图从程序语言的角度上去解决数字资产的保护问题，

# Move 语言的设计
上一节解释了 Move 语言想要解决的问题，在解决问题的同时，Move 语言同时还要实现其他智能合约编程语言的特性。因此，Move 语言给自己定了四个目标：一等资源类型的保护（Frist-Clase Resources），灵活，安全，以及可静态验证。
## Frist-Class Resources
上一篇博客中提到，在 Global State 中，Libra 维护了一个账户地址到账户模型的映射，账户模型中有 Module 和 Resource。Module 可以理解为程序代码，Resource 可以理解为 data。数据/数字资产都保存在 Resource 中。从上面的分析中我们看到，数字资产虽然可以用一个变量来表示，但作为资产，其本身又有很多**现实**特性，例如资产不能被随意**创造**、**修改**或**销毁**。因此，Move 语言在设计上引入了一个 Frist Class Resources 概念，简单说，一等资源类型首先是一种变量类型，然后规定这种类型的变量
* **不能被复制**，
* **不能被隐式地销毁**，
* **只能在不同程序地址上“移动”**。

Libra 通过 Frist-Class Resource 类型把“数字资产”给抽象出来，并且 Libra coin 本身就是一个普通的基于 First-Class 类型变量的实现，开发者可以同样基于 First Class 类型变量去定义各种数字资产，两者直接并无本质差别。Libra 把数字资产提到了相当高的一个高度，这和以太坊的差别比较大。
## 灵活性设计
Move 语言的灵活性设计和其他的智能合约语言区别不大。Libra 在交易脚本中包含了 Move 字节码，Move 字节码可以直接执行，也可以调用其他已发布的 Modules 代码。特殊的地方是 Libra 可以在一个交易中调用多个 procedures，能够灵活的实现在一笔交易中给多人转账（以太坊一笔交易只能调用一个 procedure）。
## 安全性设计
程序语言层面的安全性往往指内存安全，类型安全，Move 还要兼顾资源安全。Move 把可执行代码设计成一种介于字节码和程序代码之间的**类型字节码**（typed bytecode）。当*类型字节码*被发布上链的时候，会有一个 bytecode verifier 通过类型验证的方法去检查代码是否符合资源安全，类型安全和内存安全。通过检查的代码才可以被发布，再由 bytecode interpreter 去执行。
## 可验证性设计
这个地方，Libra 在链上只做轻量的关键安全性检查，但是会结合 Move 语言特性做链下的静态检查。（有点被绕晕了）Move 语言支持静态类型检查，Move 语言编译出来的类型字节码也支持安全性检查。Move 语言的高级语言还在实现中，目前白皮书只说明了几个关键点：
* 不支持 dynamic dispatch。
* Limitied mutability，对变量对修改的限制。Move 借鉴了 Rust 语言的资源“借用”机制，以此来保证一个变量在任意一个确定的时间上只能被一个引用所修改。
* 模块化。

# 从代码样例来看 Move 语言
上一节的内容概念太多，也不直观，这边还是结合白皮书中的样例去看一下 Move 语言的特点。
> 注：白皮书中的 Move 语言代码是一个 IR（intermediate representation）代码，作为介于字节码和高级语言之间的**中间层**代码，基本上可供人类阅读。
> Move 语言的高级源语言尚未实现。

## 转账
例子还是从**转账**这个最基本问题开始，结合 Libra 账户模型，一笔转账交易，就是把账户 a 的资源部分中定义的 coin，移动到账户 b 的名下。
![1.png](/images/2019-08-09-Introduciton-to-Libra-2/1.png)

上图里面有三个账户，`0x0`~`0x2`。`0x0`账户有一个`Currency`模块，有一个`0x0.Currency.Coin`类型的资源。`0x1`账户有另外一个模块 MyModule1，有一个`0x0.Currency.Coin`和一个`0x1.MyModule.T`的资源。
`Coin`是账户`0x0`下的`Currency`模块定义的：
```
module Currency {
  resource Coin { value: u64 }
  // ...
}
```
因为`Coin`是在`Currency`模块中定义的，因此`Coin`只能由`Currency`模块中的方法去创建或销毁。
我们再看两个`Currency`中暴露的两个方法：
**`withdraw_from_sender`**：从交易发起者的存款 Coin 中拿出 amount 个 Coin
```
// Withdraw `amount` LibraCoin.T from the transaction sender's account
public withdraw_from_sender(amount: u64): Coin {
  let transaction_sender_address: address = GetTxnSenderAddress();
  let coin_ref: &mut Coin = BorrowGlobal<Coin>(move(transaction_sender_address));
  let coin_value_ref: &mut u64 = &mut move(coin_ref).value;
  let coin_value: u64 = *move(coin_value_ref);
  RejectUnless(copy(coin_value) >= copy(amount));
  *move(coin_value_ref) = move(coin_value) - copy(amount);
  let new_coin: Coin = Pack<Coin>(move(amount));
  return move(new_coin);
}
```
**`deposit`**：把给定 Coin 送到接收账户下
```
// Deposits the `to_deposit` coin into the `payee`'s account
public deposit(payee: address, to_deposit: Coin) {
  let to_deposit_value: u64 = Unpack<Coin>(move(to_deposit));
  let coin_ref: &mut Coin = BorrowGlobal<Coin>(move(payee));
  let coin_value_ref: &mut u64 = &mut move(coin_ref).value;
  let coin_value: u64 = *move(coin_value_ref);
  *move(coin_value_ref) = move(coin_value) + move(to_deposit_value);
}
```

有了这两个方法，一笔交易的转账交易可以如下实现：
```
public main(payee: address, amount: u64) {
  let coin: 0x0.Currency.Coin = 0x0.Currency.withdraw_from_sender(copy(amount));
  0x0.Currency.deposit(copy(payee), move(coin));
}
```
这个过程就是先从交易发起者的手里拿出给定量的`Coin`，然后把这笔`Coin`转移到目标账户名下，比较直观。

>更完整的例子请参考 [libra](https://github.com/libra/) 仓库下的代码：
[peer_to_peer_transfer.mvir](https://github.com/libra/libra/blob/master/language/stdlib/transaction_scripts/peer_to_peer_transfer.mvir)
[libra_account.mvir](https://github.com/libra/libra/blob/master/language/stdlib/modules/libra_account.mvir)

## 仔细体会
> 看到上面代码我第一个反应是：WTF 什么鬼搞这么复杂，只能再仔细看看，去体会这么设计的缘由。

这里的语法语义很多借用自 Rust 和 C++11，如`move`，`copy`和`＆mut`：
* `move()`：获取变量使用权
* `copy()`：复制变量
* `＆mut`：获取变量的可变引用（于此相对的，`&`，只能获取只读引用）

另外有几个特殊的内置全局方法，`Pack`、`Unpack`和`BorrowGlobal`：
* `Pack<T>()`：创建一个类型为 T 的资源
* `Unpack<T>()`：将类型为 T 的输入资源销毁，并返回输入资源的数值
* `BorrowGlobal<T>()`：获取输入地址下的资源 T 实例的引用

所以`withdraw_from_sender`做了如下工作：
1. 获取付款账户下`Coin`资源的引用
2. 销毁转账金额的`Coin`
3. 创建一个新`Coin`，数值和转账金额相等，并把这个`Coin`作为返回值

`deposit`做了如下工作：
1. 销毁输入`Coin`，并获取输出`Coin`的数量值
2. 获取收款账户下`Coin`资源的引用
3. 修改收款账户下的`Coin`资源值，加上转账金额

本来加加减减的过程被搞的如此复杂是因为资源：
* **不能被复制**，
* **不能被隐式地销毁**，
* **只能在不同程序地址上“移动”**。

## 这么做能避免什么问题？
1. 如果对资源进行`copy`操作，会在字节码验证器找出来，因而无法凭空复制资源。
2. 一个资源在其生命周期内只能被`move`一次，如果试图执行下面的代码：
```
public main(payee: address, amount: u64) {
  let coin: 0x0.Currency.Coin = 0x0.Currency.withdraw_from_sender(copy(amount));
  0x0.Currency.deposit(copy(payee), move(coin));
}
```
也是会被检查出来。
3. 忘记处理`Coin`，比如下面的代码
```
public main(payee: address, amount: u64) {
  let coin: 0x0.Currency.Coin = 0x0.Currency.withdraw_from_sender(copy(amount));
}
```
`Coin`在过程中从未被使用过，如果开发者忘记处理，这笔资产就“丢失”了，这种也会被检查出来。

如此强的引用控制和变量生命周期控制，Move 试图从语言层面控制住资源的安全性。

# 小结
在我看来，Move 语言最大的亮点是把数字资产直接在语言层面进行表示，且在语言层面维护住了数字资产的稀缺性和安全性，而这正是**资产**所必备的现实特性。如此，Libra 希望开发者基于 Move 去实现各种代币、积分或者资产的时候，能够从语言层面获得帮助去优雅的维护好资产的安全性。
>*其他*：
>Move 语言高级语言版本还没放出来，具体能不能给开发者带来便利，目前还不好说。
>Libra 的开发者应该是 Rust 的支持者。

>抛开区块链，如果让我去设计一个基于数据库的资产管理应用，我也只会把资产定义成数据库中的一个普通值类型，和程序中的其他变量无二。对于资产的操作也无非是加加减减，至于对资产的操作会不会出错，完全基于应用的代码逻辑控制。如果，数据库也有类似的变量控制机制呢？会不会减少这类应用开发中发生数据错误的概率？

参考资料：
https://developers.libra.org/docs/move-paper
https://developers.libra.org/docs/assets/papers/libra-move-a-language-with-programmable-resources.pdf
https://github.com/libra/libra/tree/master/language/stdlib/modules
https://www.chainnews.com/articles/775489969485.htm