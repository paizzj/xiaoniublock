整理下bitcoin源码文件对应的功能，分四部分:      
base, 基础的数据结构，序列化操作，密码算法等          
net, 主要是网络部分的操作，p2p        
tx, 交易部分       
db & ui:  存储和消息以及QT实现的UI部分       

base
* base58
* arith_uint256, uint256, univalue
* hash
* secp256k1
* crypto, key, keystore, pubkey, random
* bloom
* merkle
* indirectmap,  prevector, reverse_iterator, limitedmap
* tinyformat, allocators, sync, thread
* serialize, core_io
* cuckoocache


net
* addrdb, addrman, net, netbase,  net_processing , protocol, torcontrol

tx
* block, coins, chain
* primitives, rpc, consensus, script
* pow, minner, wallet

db & ui
* leveldb, zmq, qt

