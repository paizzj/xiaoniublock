# AppInitMain

>bool AppInitMain(boost::thread_group& threadGroup, CScheduler& scheduler)

我们先看函数的两个参数，threadGroup, 和 scheduler. 从字面意思理解前者是
线程组或者线程池(threadGroup), 后者是调度器(scheduler), 这个时候可以想象下线程组和
调度器协同工作的场景: 调度器去调度线程池中的线程协同完成工作,注意调度器也是一个线程。     
是不是可以理解为比特币就是由线程池和调度器组成的？

## 4a 应用初始化(application initialization)
通过初始化签名和脚步执行缓存，然后通过创建nScriptCheckThreads个线程去检查校验脚本。

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
   
当编译的时候，只有开启了钱包功能，才会执行校验钱包数据库。

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

## 7






[onion]:https://en.wikipedia.org/wiki/.onion
[zmq]:http://zeromq.org