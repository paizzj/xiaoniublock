# AppInitMain

>bool AppInitMain(boost::thread_group& threadGroup, CScheduler& scheduler)

我们先看函数的两个参数，threadGroup, 和 scheduler. 从字面意思理解前者是
线程组或者线程池(threadGroup), 后者是调度器(scheduler), 这个时候可以想象下线程组和
调度器协同工作的场景: 调度器去调度线程池中的线程协同完成工作,注意调度器也是一个线程.     
是不是可以理解为比特币就是由线程池和调度器组成的？

## 4a 应用初始化(application initialization)
通过初始化签名和脚步执行缓存，然后通过创建nScriptCheckThreads个线程去检查校验脚本.

接着创建一个轻量级的任务调度器，这个调度器是一个线程:           

	    // Start the lightweight task scheduler thread                  
    	    CScheduler::Function serviceLoop = boost::bind(&CScheduler::serviceQueue, &scheduler);                             
    	    threadGroup.create_thread(boost::bind(&TraceThread<CScheduler::Function>, "scheduler", serviceLoop));</code></pre>		                         
                   
注册这个线程到后台信号调度器中:
	    GetMainSignals().RegisterBackgroundSignalScheduler(scheduler);                 
                       
然后启动HTTP server:         
>AppInitServers(threadGroup);            

## 5 校验钱包数据库的完整性(verify wallet database integrity)
     #ifdef ENABLE_WALLET                  
        if (!CWallet::Verify())                  
          return false;                   
     #endif                  
   
当编译的时候，只有开启了钱包功能，才会执行校验钱包数据库.

## 6 网络初始化 (network initialization)      
	    assert(!g_connman);
	    g_connman = std::unique_ptr<CConnman>(new CConnman(GetRand(std::numeric_limits<uint64_t>::max()), GetRand(std::numeric_limits<uint64_t>::max())));
	    CConnman& connman = *g_connman;

	    peerLogic.reset(new PeerLogicValidation(&connman));
	    RegisterValidationInterface(peerLogic.get());
	    RegisterNodeSignals(GetNodeSignals());      
以上主要初始化两个对象： CConnman 和 PeerLogicValidation. 前者是网络连接的管理类， 后者是对等逻辑验证类.                   

接着处理参数：                     
* -uacomment, -onlynet, -dns, [.onion][onion], -proxy,                                
* -listen", -discover, -blocksonly, -externalip, -maxloadtarget                      

然后如果开启了ZMQ，初始化ZMQ. 这是一个消息管理器，[zeromq][zmq]
<pre><code>#if ENABLE_ZMQ 
    pzmqNotificationInterface = CZMQNotificationInterface::Create();

    if (pzmqNotificationInterface) {
            RegisterValidationInterface(pzmqNotificationInterface);
    }
#endif</code></pre>		

## 7 加载区块链 (load block chain)
我们在看这部分的时候，需要了解区块的组成以及和交易相关的细节,                     
这部分我们留在后面详细介绍，现在我们先把代码逻辑梳理下.
* 通过 -reindex 参数决定是否需要重建块索引
* LoadBlockIndex(chainparams),根据参数加载块索引
* LoadGenesisBlock(chainparams)，初始化块数据
* pcoinsdbview->Upgrade(),如果有必要更新数据库格式
* ReplayBlocks(chainparams, pcoinsdbview)，重放块数据
* CVerifyDB().VerifyDB() 验证块数据                  

## 8 加载钱包(load wallet)
如果开启钱包功能，加载钱包数据:       
* CWallet::InitLoadWallet();


## 9 数据目录管理(data directory maintenance)
这部分有点难懂: 
* fPruneMode 是什么意思?
* DEPLOYMENT_SEGWIT 关于分叉? 

这2部分后续继续再看看.

## 10 导入块数据( import block)
* 这部和第7部: 加载块有什么区别呢？
* 这部分的代码逻辑是通过创建一个线程来导入块.

## 11 开启节点( start node ) 
* Discover(threadGroup); 发现节点
* 设置p2p节点的网络连接参数后，由连接管理器开启节点: connman.Start(scheduler, connOptions)

## 12 完成 (finished)









[onion]:https://en.wikipedia.org/wiki/.onion
[zmq]:http://zeromq.org