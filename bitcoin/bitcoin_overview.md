# bitcoin 源码概览
我们分析bitcoin的核心源码的时候，最好是先看下源码目录下的文档：

\~/bitcoin/doc/build-unix.md (bitcoin是源码所在的目录)

其中描述了怎么在unix(linux)下编译比特币源码，其中包含了一些必备的库。
其中几个必须的库：libssl,  libboost, libevent.

## 统计比特币源码行数：
在 ~/bitcoin/src 目录下执行以下命令，忽略目录 leveldb , qt , test, bench, univalue, zmq.

find . \( -path ./qt  -o -path ./leveldb  -o -path ./bench  -o -path ./test -o -path ./univalue -o -path ./zmq \) -prune -o -name \*.h -o -name \*.cpp | xargs wc -l

我现在的版本(2017/7/29)是:86196行，以后我们就反复翻看着8万多行的代码，当然我们也要随时pull最新的代码，跟进最新的更新，相信这样持续1年以上会有不小的收货。


## 当我们完全编译的时候，

在src目录下分别生成可执行程序:

bitcoin-cli : bitcoin-cli rpc client

bitcoind   :  bitcoind 核心

bitcoin-tx :  bitcoin-tx utilty

在src/qt目录下生成UI界面的：

bitcoin-qt :  bitcoin-qt


主要部分是： bitcoind 和 带UI的 bitcoin-qt 两大部分。

我们可以先把重点放在命令行界面的： bitcoind程序上。
