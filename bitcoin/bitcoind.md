# bitcoind   
Bitcoin Daemon , 比特币核心守护进程

源码文件位置: [~/bitcoin/src/bitcoind.cpp][bitcoind]     
主要流程如下
<pre><code>
main() 
       --> SetupEnviroment()   
       --> noui_connect()   
       --> AppInit()   	   
       	           -->  InitLogging()   
		   -->  InitParameterInteraction()    
		   -->  AppInitBasicSetup()     
		   -->  AppInitParameterInteraction()    
		   -->  AppInitSanityChecks()    
		   -->  daemon()    
		   -->	AppInitLockDataDirectory()   
		   -->	AppInitMain() ---> 这个函数包含了最重要的初始化步骤           
		   -->  Interrupt()    
		   -->  Shutdown()   
</code></pre>

## 入口函数
<pre><code>
int main(int argc, char* argv[])
{
    SetupEnvironment();

    // Connect bitcoind signal handlers
    noui_connect();

    return (AppInit(argc, argv) ? EXIT_SUCCESS : EXIT_FAILURE);
}
</code></pre>

## SetupEnviroment()    
SetupEnvironment()见文件util.h/util.cpp     
功能: 主要设置语言环境。    
详细的语言设置参见：[c++标准库][c++]第16章   


## noui_connect()   
noui_connect()  见文件noui.h/nohui.cpp    
功能: 主要是设置信号处理函数。    
详细的信号处理函数用法 ：参见[boost程序库完全参考指南][boost]第11章(函数与回调)signals2   


## AppInit()
bool AppInit(int argc , char *argv) 见文件~/bitcoin/src/bitcoind.cpp
功能:     
* 参数，配置文件解析然后设置相应的值。
* 然后是依次初始化以下行数     
  InitLogging()   
  InitParameterInteraction()    
  AppInitBasicSetup()     
  AppInitParameterInteraction()    
  AppInitSanityChecks()    
  daemon()    
  AppInitLockDataDirectory()   
  AppInitMain() ---> 这个函数包含了最重要的初始化步骤           
  以上函数具体声明在文件~/bitcoin/src/init.h      

* 中断线程配置
   Interrupt()    
   Shutdown()   
 


## AppInitMain() ---> 这个函数包含了最重要的初始化步骤           
见文件 ~/bitcoin/src/init.cpp    
当我们仔细看init.cpp文件是，注意到代码中的注释已经把整个的初始化流程表达清楚了        
通过执行命令      
> grep "Step" init.cpp      
得到如下结果：                         
    // ********************************************************* Step 1: setup     
    // ********************************************************* Step 2: parameter interactions    
    // ********************************************************* Step 3: parameter-to-internal-flags   
    // ********************************************************* Step 4: sanity checks   
    // ********************************************************* Step 4a: application initialization   
    // ********************************************************* Step 5: verify wallet database integrity   
    // ********************************************************* Step 6: network initialization   
    // ********************************************************* Step 7: load block chain   
    // ********************************************************* Step 8: load wallet   
    // ********************************************************* Step 9: data directory maintenance   
    // ********************************************************* Step 10: import blocks   
    // ********************************************************* Step 11: start node    
    // ********************************************************* Step 12: finished   


其种从4a 开始就是函数AppInitMain() 主要的流程


### 4a: application initialization(应用初始化)       
### 5:  verify wallet database integrity(验证钱包数据库完整性)          
### 6： network initialization(p2p网络初始化)        
### 7:  load block chain (加载区块链)     
### 8:  load wallet (加载钱包)      
### 9:  data directory maintenance (维护数据目录)     
### 10: import blocks  (导入块)    
### 11: start node (启动p2p节点 )     
### 12: finished   

						    

















[bitcoind]:https://github.com/bitcoin/bitcoin/blob/master/src/bitcoind.cpp
[c++]: https://union-click.jd.com/jdc?d=39XS7i
[boost]:https://union-click.jd.com/jdc?d=Fv77Bs

