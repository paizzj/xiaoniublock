## 学习比特币源码经历

# 背景
   我是个码农也是个典型的失败的投机者，13年接触了挖矿，去买了阿瓦隆矿机，挖了几十币600 RMB单价出的。
   当初也去下载了比特币的源码，看过，但是没看懂，然后就作罢。期间断断续续折腾了很多事情，依然没头绪。
   16年买了《把时间当做朋友》，人困惑的时候从书里最能找到解决的方法。17年5月重新入场。

# 目的
   这次入场，不想再走原来的老路，给自己定了小目标，一定先看懂代码是咋回事儿，再去玩blockchain。

# 突破口
   认证研读了INBlockchain 的开源区块链 ICO [投资原则][inblockchain]
   然后自己去研究了下文中提到的几个币（sia, namecoin, zcash, qtum, eos），随后下载了他们的源码开始蜗牛般的阅读。


# 整理框架
   * 比特币及其为基础衍生： bitcoin, namecoin, zcash, qtum 等
   * 以太坊为基础的衍生： eth,  eth上开发的DAPP， sia( 严格意义上sia代码层面没有借鉴eth，
   只是重新实现了智能合约的概念，但是其简单直接的业务场景非常好，private cloud. )
   * steem，bts, eos 为基础的
   * tezos （这里为什么提到tezos， 是因为其筹集规模，投资团队，研发团队，技术实现都很特别）


# 技术参考资料
   经过一段时间的学习，就2块理论需要掌握：p2p( 对等网络，不是p2p网贷), 密码协议。
   具体的实现细节需要掌握编程语言（c/c++， go） , 如果仅仅想了解比特币为基础的代码, eth 也有c++实现，所以c++是必须的 。 
   我整理了看比特币源码对自己帮助最大的一些资料和参考书。
   
   * P2P 参考书：[P2P系统及其应用][p2p]
   * C++参考书： [C++标准库][c++] 
   * boost 库：  [boost程序库完全指南][boost]
   * Ocaml参考： [Real World OCaml][ocaml]这个对想学习tezos的可以了解下，目测国内没几个人玩这个

# 体会
   关于blockchain的技术背景，其实没有一个是新的，有的甚至是10几年前的东西。
   
   对于一些术语的理解，其实已经有很多资料解释了，但是对于程序员来说还远远不够，
   
   包括李笑来解释的什么是blockchian, 这些解释都是面向大众的投资者做的正确的说明而已，
   程序员应该用这些解释说明去和源码的实现做下对应，这样才能加深理解，也就不会被一些很高大上的词给蒙了，也就可以帮助大家做投资决策了。

[p2p]: https://union-click.jd.com/jdc?d=oGNekr
[c++]: https://union-click.jd.com/jdc?d=39XS7i
[boost]:https://union-click.jd.com/jdc?d=Fv77Bs
[ocaml]:https://union-click.jd.com/jdc?d=QQq61i
[inblockchain]:https://github.com/xiaolai/INB-Principles/blob/master/Chinese.md








