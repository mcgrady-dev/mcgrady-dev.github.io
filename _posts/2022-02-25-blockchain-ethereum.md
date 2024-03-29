---
layout: 
title: Android Blockchain Ethereum
date: 2022-02-25 17:01 +0800
tags: [android,blockchain,ethereum]
---

Android平台区块链应用技术调研

<!--more-->

## 1. 概述

Ethereum（以太坊）是一个开源的有智能合约功能的公共区块链平台，通过其专用加密货币以太币（Ether，简称 **ETH**）提供去中心化的以太虚拟机（Ethereum Virtual Machine，简称 **EVM**）来处理P2P（点对点）合约。

在以太坊世界中，有一个单一的、规范的计算机（称为以太坊虚拟机，或 **EVM**），其状态获得以太坊网络中的每个人认可。 每个参与以太坊网络的人（每个以太坊节点）都会保存一份该计算机的状态。 此外，任何参与者都可以广播请求这台计算机进行任意计算。 每当广播这样的请求网络时，网络上的其他参与者就会验证、确认并进行（"执行"）计算。 这个命令会导致 EVM 的状态变化，并且在整个网络中传播。

计算请求被称为交易请求；所有交易的记录以及 EVM 的当前状态都存储在区块链中，而区块链又由所有节点存储并达成一致。

加密机制确保了一旦交易被验证为有效并添加到区块链中后，之后就不会被篡改。 同样的机制还确保所有交易都以适当的“权限”签名和执行。



## 2. 以太坊的节点和客户端

“节点”是指一种称为客户端的软件。 客户端是一种以太坊的实现，它可以验证每个区块中的所有交易，从而确保网络安全和数据准确。所以每个节点都存储了整个区块链的数据，并重放所有的交易以验证结果的状态。



## 3. Smart contract（智能合约）

![blockchain-ethereum-637x357](https://s2.loli.net/2022/07/19/3NP7EZvG5XRABzJ.png)

智能合约只是一个运行在以太坊链上的一个程序。 它是位于以太坊区块链上一个特定地址的一系列代码（函数）和数据（状态）。

智能合约也是一个[以太坊帐户](https://ethereum.org/zh/developers/docs/accounts/)，我们称之为合约帐户。 这意味着他们有余额，他们可以通过网络进行交易。 但是，他们无法被人操控，他们是被部署在网络上作为程序运行着。 个人用户可以通过提交交易执行智能合约的某一个函数来与智能合约进行交互。 智能合约能像常规合约一样定义规则，并通过代码自动强制执行。 默认情况下，您无法删除智能合约，与它们的交互是不可逆的。

一个智能合约像是一套设立在区块链上的规则，所有人都可以准确地观察和运行这套规则。

设想一下，有一台自动贩卖机：如果向它提供足够的资金和正确的选择，您将得到您想要的货品。正如自动贩卖机一样，智能合约可以像您的以太坊账户一样存储持有资金。这允许代码之间协议和交易。

一旦去中心化应用部署到以太坊网络，您将无法更改他们。去中心化应用之所以可以被去中心化就是因为他们被合约中的逻辑所控制，而不是公司或者个人。

因此，通过智能合约，开发者可以任意构建和部署面向用户的复杂应用程序和服务，例如市场、金融工具、游戏等。



### 智能合约的安全

一旦一个智能合约部署到了以太坊的网络上，它将是永不可变的，且将永久存在。如果你写了一个 bug，你将不能下架这个有问题的版本，你只能在后续的版本中修复。



## 3. EVM（以太坊虚拟机）

以太坊虚拟机是一个全局虚拟计算机，以太坊网络每个参与者都会存储并同意其状态。 任何参与者都可以请求执行 EVM 上的任意代码；代码执行会改变 EVM 的状态。



## 4. DApp (去中心化应用)

![blockchain-dapp-light](https://s2.loli.net/2022/07/19/NrOAU9wKdI15X3D.png)

DApp 即 Decentralized 与 Applicationd 的组合，又称分布式应用，**去中心化应用的后端代码（智能合约）运行在一个去中心化的网络上，而非中心化的服务器。他们使用以太坊区块链网络作为数据存储端，并且用智能合约实现 app 的业务逻辑。**

一个DApp与传统的开发中的，客户端或前端是类似，区别仅在于它们与以太坊的区块链进行交互（也可以同时与其它服务交互）。这些客户端往往用 JS 编写，因为当前还暂时没有完成全部的向 NodeJS 的转换。

另外，大多数的 DApp 客户端使用JS的原因，是因为它可以在浏览器中运行，因为大家都有浏览器，这样每个人都可以运行了。由于有更多的 Go 语言的开发工具，使用 Go 语言来写客户端的也不少。在现在这个激烈的发展期，这意味着，除非你有自己的偏好，否则可能要从 Go 和 JS 来选择一种语言，来与以太坊区块链，以及基于以太坊开发的协议进行交互了。



## 5. ETHER（以太币）

以太币 (**ETH**) 是用于以太坊网络上许多事物的加密货币。 从根本上讲，它是唯一可接受的交易费用支付方式，需通过[合并](https://ethereum.org/zh/upgrades/merge) 以便在主网得到验证并提出区块。

ETH 这种加密货币存在的目的是允许算力市场的存在。 这种市场为参与者提供了一种经济激励，从而验证并执行交易请求，为网络提供计算资源。

任何广播交易请求的参与者也必须向网络提供一定数量的 ETH 作为奖励。 这笔奖金将颁发给最终验证交易、执行交易并将其提交到区块链，并在网络中广播的任何人。

支付的 ETH 数量对应于进行计算所需的时间。 这类奖励也可以防止恶意参与者通过请求执行无限循环或资源密集型脚本来故意堵塞网络，因为这些参与者将为自己的计算时间付费。



## 6. 以太坊钱包

以太坊钱包是一种帮助您与您的以太坊账户进行交互的工具。可以把它想像成一个背后没有银行的互联网银行应用。通过钱包您可以查看余额，发送交易或者链接到各种应用。



## 7. 以太坊网络

- **Mainnet**
  以太坊主网，通常是所有客户端的默认网络
- **Ropsten**
  以太坊使用工作量证明的主测试网络。
- **Kovan**
  parity 客户端组成的测试网络，使用授权证明来提升对垃圾邮件攻击的抗扰度，并且持续4秒的阻塞时间。
- **Rinkeby**
  geth 客户端组成的测试网络，使用集团共识，尽管计算量低，但是对恶意行为者更有弹性。



## 8. 以太坊开发框架

### web3j

![blockchain-web3j-network](https://s2.loli.net/2022/07/19/Z9kWQiASPhsHtJL.png)

web3j是一个轻量级、高度模块化、响应式、类型安全的Java和Android类库提供丰富API，用于处理以太坊智能合约及与以太坊网络上的客户端(节点)进行集成。

**web3j的特点**

- 基于HTTP和IPC的以太坊[JSON-RPC](http://cw.hubwiz.com/card/c/ethereum-json-rpc-api/)客户端API的完整实现
- 对于以太坊钱包的支持
- 自动生成Java智能合约封装包，以创建、部署、交易和调用来自本机Java代码的智能合约（支持[solidity](http://solidity.readthedocs.io/en/latest/using-the-compiler.html#using-the-commandline-compiler)和[Truffle](https://github.com/trufflesuite/truffle-contract-schema)定义格式）
- 用于过滤器工作的响应式函数API
- 以太坊名称服务（[ENS](https://ens.domains/)）支持
- 支持`Parity`的[personal模块](https://github.com/paritytech/parity/wiki/JSONRPC-personal-module)和`Geth`的[personal客户端API](https://github.com/ethereum/go-ethereum/wiki/Management-APIs#personal)
- 支持[Infura](https://infura.io/)，所以你不必自己运行一个以太坊客户端
- 综合集成测试并展示了以上几种场景
- 命令行工具
- Android兼容
- 通过[web3j-quorum](https://github.com/web3j/quorum)支持JP摩根的Quorum



### Solidity 语言

Solidity 是一门面向合约的、为实现智能合约而创建的高级编程语言。这门语言受到了 C++，Python 和 Javascript 语言的影响，设计的目的是能在以太坊虚拟机（EVM）上运行。

作为一种真正意义上运行在网络上的去中心合约，它又有很多的不同，下面列举一些：

- 以太坊底层是基于帐户，而非UTXO的，所以有一个特殊的`Address`的类型。用于定位用户，定位合约，定位合约的代码（合约本身也是一个帐户）。
- 由于语言内嵌框架是支持支付的，所以提供了一些关键字，如`payable`，可以在语言层面直接支持支付，而且超级简单。
- 存储是使用网络上的区块链，数据的每一个状态都可以永久存储，所以需要确定变量使用内存，还是区块链。
- 运行环境是在去中心化的网络上，会比较强调合约或函数执行的调用的方式。因为原来一个简单的函数调用变为了一个网络上的节点中的代码执行，分布式的感觉。
- 最后一个非常大的不同则是它的异常机制，一旦出现异常，所有的执行都将会被回撤，这主要是为了保证合约执行的原子性，以避免中间状态出现的数据不一致。



### Truffle 框架

一旦你开始写智能合约，你会重复做大量的操作，比如编译源码为字节码和 abi，部署到网络，测试然后部署合约等等。你也许希望更关注于你想要实现的东西。Truffle 等框架标准化和自动化了这些琐碎的工作。

Truffle 是以太坊开发智能合约过程中主流的框架，本身采用 JavaScript 编写，支持智能合约的编译、部署和测试。

#### Truffle 包含以下功能：

- 内置的智能合约编译，链接，部署和二进制文件的管理
- 快速开发下的自动合约测试
- 脚本化的，可扩展的部署与发布框架
- 部署到不管多少的公网或私网的网络环境管理功能
- 使用 EthPM&NPM 提供的包管理，使用 [ERC190](https://github.com/ethereum/EIPs/issues/190) 标准
- 与合约直接通信的直接交互控制台（写完合约就可以命令行里验证了）
- 可配的构建流程，支持紧密集成
- 在 Truffle 环境里支持执行外部的脚本

#### Truffle 客户端：

- **适用于开发的客户端**

  - [EtherumJS TestRPC](https://github.com/ethereumjs/testrpc)
    它是一个完整的在内存中的区块链仅仅存在于你开发的设备上。它在执行交易时是实时返回，而不等待默认的出块时间，这样你可以快速验证你新写的代码，当出现错误时，也能即时反馈给你。它同时还是一个支持自动化测试的功能强大的客户端。Truffle充分利用它的特性，能将测试运行时间提速近90%。

- **使用正式发布的客户端**
  这些是完整的客户端实现，包括挖矿，网络，块及交易的处理，Truffle可以在不需要额外配置的情况下发布到这些客户端：

  - [Geth (go-ethereum)](https://github.com/ethereum/go-ethereum)
  - [WebThree (cpp-ethereum)](https://github.com/ethereum/webthree-umbrella)
  - [More](https://www.ethereum.org/cli)

- **当发布到私有网络中**

  私人网络中使用了相同的技术，但却有不同的配置。所以你可以将上面提及的客户端来运行一个私有的网络，部署到这样的网络也是使用同样的方式。



### Infura 平台

![blockchain-infura](https://s2.loli.net/2022/07/19/tMVRcJxE71iQaqn.png)

Infura 是一种 IaaS（Infrastructure as a Service）产品，目的是为了降低访问以太坊数据的门槛。通俗一点讲，Infura 就是一个可以让你的 DApp 快速接入以太坊的平台，不需要本地运行以太坊节点。



## 10. 相关技术

### RPC (Remote Procedure Call) 远程过程调用

**RPC即远程过程调用**，也就是说两台服务器A，B，一个应用部署在A服务器上，想要调用B服务器上应用提供的方法，由于不在同一内存空间，不能直接调用，需要通过网络来表达调用的语义和传达调用的数据。

RPC技术简单说就是为了解决远程调用服务的一种技术，使得调用者像调用本地服务一样方便透明。



### IPFS (InterPlanetary File System) 星际文件系统

IPFS是星际文件系统，是一个旨在创建持久且分布式存储和共享文件的网络传输协议。它是一种内容可寻址的对等超媒体分发协议。在IPFS网络中的节点将构成一个分布式文件系统。它是一个开放源代码项目，自2014年开始由Protocol Labs在开源社区的帮助下发展。其最初由Juan Benet设计。

IPFS是一个互联网的底层协议，类似HTTP协议，上线时间是2015年的5月5号。它的目标是为了补充甚至是取代目前统治互联网的超文本传输协议（HTTP）。

IPFS是传输协议，不是区块链项目，没有使用任何区块链技术。但是具备区块链去中心化的精神。所以，IPFS没有Token、没有发币、不能挖矿；Filecoin才是Token，挖的是Filecoin。

IPFS目标是打造一个更加开放、快速、安全的互联网，利用分布式哈希表解决数据的传输和定位问题， 把点对点的单点传输改变成P2P（多点对多点）的传输，其中存储数据的结构是**哈希链**。



### 代币

代币常常用来激励用户与某个协议进行交互，或者证明对某个资产的所有权，投票权等等。

